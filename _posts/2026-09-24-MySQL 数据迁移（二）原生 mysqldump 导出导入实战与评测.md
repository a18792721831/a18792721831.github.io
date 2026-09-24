---
layout: post
title: "MySQL 数据迁移（二）原生 mysqldump 导出导入实战与评测"
date: 2026-09-24 15:04:12 +0800
categories: ["MySQL", "数据迁移", "mysqldump", "mydumper", "myloader", "数据库", "DTS"]
description: "mysqldump 是 MySQL 自带的逻辑迁移工具，人人都会用，但真正理解它在压测下对源库的冲击、导入慢的根因、以及适用数据量级边界的人不多。本篇在 2000 万行真实数据集上完整实战 mysqldump 导出 → 压缩传输 → mysql source 导入全流程，边迁移边压测，用数据回答：它到底能扛多大数据量。"
keywords: ["MySQL", "数据迁移", "mysqldump", "mydumper", "myloader", "数据库", "DTS"]
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/166593481
> - 发布时间：2026-09-24 15:04:12
> - 标签：#mysql, #数据库, #mysqldump, #mydumper, #myloader, #DTS

---

## MySQL 数据迁移（二）原生 mysqldump 导出导入实战与评测

### 摘要

mysqldump 是 MySQL 自带的逻辑迁移工具，人人都会用，但真正理解它在压测下对源库的冲击、导入慢的根因、以及适用数据量级边界的人不多。本篇在 2000 万行真实数据集上完整实战 mysqldump 导出 → 压缩传输 → mysql source 导入全流程，边迁移边压测，用数据回答：它到底能扛多大数据量。

### 1. 建议先看

本篇是《MySQL 数据迁移》系列第二篇，评测环境（单节点 / 主从 / 读写分离三套架构）、数据集（sysbench 2000 万行）和评测指标定义都在第一篇：

- [MySQL 数据迁移（一）为什么迁、怎么迁、怎么评——三大方案总览](https://blog.csdn.net/a18792721831/article/details/166593341)

### 2. mysqldump 的工作原理

#### 2.1 它本质上是一个"大号 SELECT 客户端"

mysqldump 是 MySQL 官方自带的客户端工具，逻辑导出的本质：连接源库 → 按表执行 `SELECT *` → 把结果拼成 `INSERT` 语句写进文本文件。它不碰数据文件、不碰存储引擎内部结构，所以导出产物是纯 SQL 文本，跨版本、跨平台兼容性最好。

#### 2.2 一致性是怎么保证的：--single-transaction

导出 20 张表需要时间，导出期间业务还在写，凭什么导出来的数据是同一时刻的？靠的是 `--single-transaction`：

```sql
-- mysqldump --single-transaction 内部等价于：
START TRANSACTION WITH CONSISTENT SNAPSHOT;  -- 建立一致性读视图
SELECT * FROM sbtest1;  -- 表1的数据，永远看到事务开始时刻的版本
SELECT * FROM sbtest2;  -- 表2同上
...
```

InnoDB 的 MVCC 机制保证：快照建立后，后续所有 SELECT 都读到同一时刻的数据版本，期间业务的新写入被隔离在快照之外。**注意两个坑**：

1. 只对 InnoDB 表有效。如果有 MyISAM 表，`--single-transaction` 保护不了它，需要 `--lock-all-tables`（锁全库，业务停摆）
2. 长事务的代价：快照期间源库的写操作会产生 undo 日志版本链，导出越慢、业务写入越猛，undo 膨胀越厉害（第四节实测）

#### 2.3 extended insert：一条 INSERT 装一火车的值

mysqldump 默认开启 `--extended-insert`（8.0 中已不可关闭），导出产物长这样：

```sql
INSERT INTO `sbtest1` VALUES (1,499504,'...', '...'),(2,502015,'...','...'),(3,...),...;
```

一条 `INSERT` 挂几百上千个 value 元组，而不是每行一条。这个设计决定了导出文件的"批次"大小由 `net_buffer_length`（默认 1MB）控制，也直接影响导入速度——逐行 INSERT 和批量 INSERT 的导入性能差几倍。

#### 2.4 位点记录：--source-data 与 GTID

全量导出只是迁移第一步，导出后源库还有增量写入，需要从某个位点开始追 binlog。`--source-data=2` 会把导出时刻的 binlog 位点写进产物（8.0.22+ 用 `--source-data`，老版本叫 `--master-data`，`=2` 表示注释掉不执行）：

```sql
--
-- source to log replication position to file
-- POSITION  = binlog file and position of the snapshot
CHANGE REPLICATION SOURCE TO SOURCE_LOG_FILE='mysql-bin.000007', SOURCE_LOG_POS=157;
```

这个位点就是后续增量追赶的起点。**注意**：从主库导出才能拿到这个位点；从只读副本导出时副本自身通常不开 binlog，`--source-data` 会直接失败——这是"从从库/只读节点导出"方案的一个注意点，第三篇会展开。

**file+pos 位点的粒度其实有点粗，GTID 才是最精确的**。

`SOURCE_LOG_FILE + SOURCE_LOG_POS` 只是 binlog 字节流里的一个偏移量：它不表达"这个位点之前包含哪些事务"，而且很脆弱——主从切换后新主上的 file/pos 完全对不上旧主的、binlog 轮转清理后按位点接复制容易错位、手工指定位点时差一个事务就会丢数据或重复。GTID 则把位点精确到**事务粒度**：导出时刻的 `gtid_executed` 是一个"已执行事务集合"（`uuid:1-N`），增量追赶时 `SOURCE_AUTO_POSITION=1` 让源库根据这个集合自动协商从哪里发，集合里已有的事务幂等跳过——不重不漏，天然容错。

mysqldump 在 GTID 环境下的产物头部会带 `SET @@GLOBAL.gtid_purged='uuid:1-N'`（即 5.5 节方案 B 的机制），它记录的就是快照时刻的事务集合；也可以在导出完成时立即执行 `SHOW MASTER STATUS` 取 `Executed_Gtid_Set` 单独记录。

生产结论：**能开 GTID 就用 GTID 管位点，file+pos 是没有 GTID 的老版本环境下的兜底**（第一篇 3.2 节讲过 binlog + GTID 是生产级迁移的前提）。

### 3. 实战：完整迁移流程

以下命令全部在真实环境执行过，环境说明见第一篇（脱敏处理）。

#### 3.1 导出

```bash
# 标准导出命令（InnoDB + GTID 环境）
docker exec mysql-source sh -c "mysqldump -uroot -p'***' \
  --single-transaction \
  --source-data=2 \
  --set-gtid-purged=OFF \
  --triggers --routines --events \
  --databases sbtest" > dump_single.sql
```

参数逐个说清楚：

| 参数 | 作用 | 不加的后果 |
|------|------|-----------|
| `--single-transaction` | 一致性快照导出，不锁表 | 用默认的 `--lock-tables`，导出期间锁表业务停摆 |
| `--source-data=2` | 记录 binlog 位点到产物（GTID 环境下同时有事务集合可记，见 2.4） | 后续增量追赶找不到起点 |
| `--set-gtid-purged=OFF` | 产物不含 SET @@GLOBAL.gtid_purged | 目标库已有 GTID 历史时防报错/防污染；若目标库是全新空实例，生产上推荐**保留默认**让源库 GTID 接管（方案选择详见 5.5） |
| `--triggers --routines --events` | 导出触发器/存储过程/事件 | 触发器和存储过程丢失，业务出问题 |
| `--quick` | 流式逐行读，不缓存整表到内存 | 默认已开，但显式写出防被配置覆盖 |

**实测数据**（2000 万行 / 20 张表，数据文件约 4.2GB，压测流量伴随）：

| 指标 | 数值 |
|------|------|
| 导出耗时 | 60~66 秒 |
| 产物大小 | 3810 MB（.sql 文本） |
| 源库压测 TPS（导出期间） | 286（基线 261，详见 4.2 节分析） |

单看导出不算慢——**mysqldump 的真正瓶颈在导入侧**（见 4.1 导入数据）。

#### 3.2 导出产物长什么样

```bash
$ head -30 dump_single.sql
-- MySQL dump 10.13  Distrib 8.0.46, for Linux (x86_64)
...
/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
...（一堆环境恢复用 SET 语句）
CREATE DATABASE /*!32312 IF NOT EXISTS*/ `sbtest`;
USE `sbtest`;
--
-- Table structure for table `sbtest1`
--
DROP TABLE IF EXISTS `sbtest1`;      -- 注意：默认先 DROP！
CREATE TABLE `sbtest1` (
  `id` int NOT NULL AUTO_INCREMENT,
  ...
) ENGINE=InnoDB AUTO_INCREMENT=1000001 DEFAULT CHARSET=utf8mb4;
--
-- Dumping data for table `sbtest1`
--
INSERT INTO `sbtest1` VALUES (1,...),(2,...),...
```

两个值得注意的细节：

1. **产物默认包含 `DROP TABLE IF EXISTS`**：往已有数据的库导入会先删表！同构迁移（源目标都有同名表）时要非常小心，必要时加 `--skip-add-drop-table`
2. **`--databases` 会带出 `CREATE DATABASE`**：不加 `--databases` 只导表，导入时需要目标库先存在

#### 3.3 压缩与传输

裸 SQL 文本压缩率很高。实测三种压缩工具对比（同一份 3810MB 的产物）：

| 工具 | 算法 | 耗时 | 产物大小 | 压缩比 |
|------|------|------|---------|--------|
| gzip | DEFLATE（单线程） | 329s | 1841MB | 0.46 |
| pigz | DEFLATE（多线程） | **43s** | 1843MB | 0.46 |
| qpress | QuickLZ | **23s** | 2610MB | 0.65 |

三个工具三种取舍：**qpress 最快但压缩率最差**（QuickLZ 是为速度设计的算法）；**pigz 速度是 gzip 的 7.6 倍，压缩率几乎无损**（多核并行切分压缩）；gzip 单线程在现代多核机器上没有任何理由再用。SQL 文本重复度高，gzip 系能压到 46%，传输带宽直接省一半多。

这里澄清一个高频混淆：**`xbstream` 和 `qpress` 经常成对出现，但它们是 XtraBackup 物理备份体系的工具**——`xbstream` 是 XtraBackup 的流式归档格式（把数据文件打包成流），`qpress` 是配套的压缩器。mysqldump 的 SQL 文本产物用 gzip/pigz/qpress 压缩都可以，`xbstream` 在这个场景完全用不上。物理备份的 `xbstream + qpress` 组合实战在第四篇。

传输环节，按 100Mbps（12.5MB/s）跨机房带宽换算传输时间：

| 产物 | 大小 | 100Mbps 传输 | 500Mbps 传输 |
|------|------|-------------|--------------|
| 裸 SQL | 3810MB | 约 5.1 分钟 | 约 61 秒 |
| gzip/pigz | 1841MB | 约 2.5 分钟 | 约 30 秒 |
| qpress | 2610MB | 约 3.5 分钟 | 约 42 秒 |

跨机房场景先压缩再传输是基本素养，pigz 是首选。

#### 3.4 导入：mysql source / 重定向

```bash
# 方式一：重定向（推荐脚本使用）
mysql -h*** -P13311 -uroot -p*** sbtest < dump_single.sql

# 方式二：mysql 客户端内 source 命令（交互排查用）
mysql> source /backup/dump_single.sql;
```

`source` 和 `<` 重定向本质相同：mysql 客户端逐行读文件、逐条执行 SQL。**单线程、单连接**——这是 mysqldump 方案导入慢的根本原因，无论源端机器多少核，导入速度都卡在这一个连接上。

导入期间目标库在干什么：每条 INSERT 要经过 SQL 解析 → 权限检查 → binlog 写入（`sync_binlog=1` 时每次提交刷盘）→ InnoDB redo 写入（`innodb_flush_log_at_trx_commit=1` 时每次提交刷盘）→ 数据页修改。一批 INSERT 组成一个事务，每个事务两次 fsync，机械盘上一次 fsync 约 10ms，一秒最多扛几十个事务——导入速度天花板由此决定。

#### 3.5 导入调优：三板斧

生产上用 mysqldump 导大表，默认参数导入慢到怀疑人生。三板斧下来能快一个数量级：

```bash
# 第一板斧：关 binlog（目标库导入期间不需要记录 binlog）
mysql --init-command="SET SESSION sql_log_bin=0" -h*** sbtest < dump_single.sql

# 第二板斧：临时调弱刷盘（双1 -> 双0，导完必须改回来）
mysql -h*** -e "SET GLOBAL innodb_flush_log_at_trx_commit=2; SET GLOBAL sync_binlog=0;"
# ... 导入 ...
mysql -h*** -e "SET GLOBAL innodb_flush_log_at_trx_commit=1; SET GLOBAL sync_binlog=1;"

# 第三板斧：导入期间关闭唯一性检查和外键检查（写入产物头）
-- dump 文件开头会自动包含：
-- SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0;
-- SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0;
```

第三板斧 mysqldump 产物默认自带，前两板斧需要手动。**调优实测数据**（同机同数据集）：

| 导入方式 | 配置 | 耗时 | 相对默认 |
|---------|------|------|---------|
| 默认 | 双 1 + 写 binlog | 529s | 基准 |
| 调参三板斧 | flush=2 + sync_binlog=0 + sql_log_bin=0 | **333s** | **快 37%** |

关掉 binlog 双写和两次 fsync，导入时间砍掉三分之一还多——每个事务省掉的两次刷盘，2000 万行攒下来就是几分钟。

三板斧只是入门，生产上完整的导入参数清单（兼容性防错 3 项 + 性能加速 12 项，含 `innodb_doublewrite=OFF`、`innodb_change_buffer_max_size=50`、`unique_checks=0` 等）见 5.2 节。

### 4. 实验评测：压测下的真实表现

实验设计（详见第一篇）：sysbench 2000 万行数据集，迁移全程对源库保持 8 线程 oltp_read_write 压测，对比基线 TPS/P95。

#### 4.1 三套架构结果总表

| 架构组 | 导出源 | 导出耗时 | 导入耗时(默认) | 导入后校验 |
|--------|--------|---------|---------------|-----------|
| 单节点 | single 本机 | 60s | 529s | 20/20 表行数一致 |
| 主从 | master 主库 | 66s | 458s | 20/20 表行数一致 |
| 读写分离 | ro 只读节点 | 60s | 459s | 20/20 表行数一致 |

先看大数字：**导出 60 秒，导入 529 秒——导入是导出的 8 倍耗时**。全流程（导出 60s + 压缩 43s + 导入 529s）约 10.5 分钟迁完 2000 万行，其中导入占了 84%。这就是 mysqldump 方案"导出不难受、导入等半年"的量化呈现。

三组导入耗时差异（458~529s）主要来自实验期间宿主机负载波动（同机跑着 6 个 MySQL 实例），量级上可视为一致。

#### 4.2 迁移期间源库性能退化分析

压测伴随导出的完整数据：

| 架构组 | 基线 TPS / P95 | 导出期间 TPS / P95 | 变化 |
|--------|---------------|-------------------|------|
| 单节点 | 256.64 / 25.74ms | 288.36 / 25.28ms | TPS +12% |
| 主从（从主库导） | 259.92 / 30.26ms | 269.94 / 30.26ms | TPS +4% |
| 读写分离（从ro导） | 260.69 / 30.81ms | 286.25 / 25.28ms | TPS +10%，P95 -18% |

**反直觉：导出期间 TPS 反而更高？** 先说结论：这是本实验数据集规模下的特例，不具备普遍性，但原理值得说透。

本实验数据文件 4.2GB ≈ `innodb_buffer_pool_size`（4G）。压测的热点行随机分布全表，基线压测时部分读请求要穿透到磁盘。而 mysqldump 顺序全表扫描把**所有数据页都灌进了 buffer pool**——相当于免费做了一次全量预热，压测的随机读从"部分命中"变成"全部命中"，抵消甚至反超了导出本身的 IO 开销。

**当数据量远大于 buffer pool 时，结论会反转**：全表扫描会持续挤掉热页（buffer pool 污染），业务命中率下降，TPS 退化、P95 上升——这才是生产库（数据量通常是内存的几倍到几十倍）导出时的常态。本实验的"反常数据"恰好从边界条件验证了原理：**导出对源库的冲击大小，取决于数据量与 buffer pool 的比例**。

真实生产借鉴：数据量小于内存时放心导；数据量大时，要么选业务低峰，要么从从库/只读节点导（见 4.4）。

#### 4.3 主从架构的复制延迟

主从组（从主库导出）期间，从库 `Seconds_Behind_Source` 采样（每 5 秒）全程为 0。

原因：mysqldump 是普通 SELECT，走 MVCC 快照读，不产生 binlog、不锁写；从库的 IO/SQL 线程只回放压测的正常写流量，导出对复制链路是"透明"的。**低写入压力下，从主库逻辑导出不会引起复制延迟**——但注意这是 8 线程压测（写入温和）的结果；高写入压力 + 大事务场景，从库单线程回放本身就会延迟，与导出无关。

#### 4.4 导出节点怎么选：主库、slave 还是 ro

读写分离组：压测打 rw（写节点），导出走 ro（只读节点），与主从组（压测和导出都打主库）对比：

| 对比项 | 主从组（主库导出） | 读写分离组（ro导出） | 结论 |
|--------|-------------------|---------------------|------|
| 写节点压测 P95 | 30.26ms | **25.28ms** | 从 ro 导出，主库延迟低 17% |
| 写节点压测 TPS | 269.94 | **286.25** | 从 ro 导出，主库 TPS 高 6% |
| 导出节点自身复制延迟 | 从库 0s | **ro 最高 12s** | 代价来了 |

从 ro 导出确实给主库减了压（P95 从 30.26ms 降到 25.28ms）。**但天下没有免费的午餐**：ro 节点一边要回放主库 binlog（SQL 线程），一边要服务 mysqldump 的全表扫描，两者抢 IO/CPU，ro 的复制延迟最高冲到 12 秒（243 次采样为 0s，约 40 次在 1~12s 之间）。

这里有个生产上更值得关注的架构场景：**主从架构下，从 slave 导出**。

在 m-s 架构里，如果 slave 是**纯备库**（不接任何业务读流量），它就是为这种时刻准备的：主库全力服务业务写入，slave 只做复制回放——此时把导出任务打到 slave 上，是**最优解**：

- 主库零干扰：导出的全表扫描、网络出口都不碰主库（实测同构数据：主库 P95 -17%、TPS +6%）
- slave 自身延迟上升完全无害：它不服务读，延迟 0s 还是 12s 业务都无感知，导出完延迟自然归零
- slave 上的数据经由复制获得，与主库逻辑一致，导出产物与从主库导出等价（位点注意事项见 5.8）

注意本实验读写分离组的 ro 在实验中恰好就处于"只回放复制 + 被导出、不服务读"的状态——压测流量全部打在 rw 上——所以上表 ro 的实测数据，可以直接作为"从纯备 slave 导出"的性能参照。

**完整版决策树**（导出节点选择）：

| 架构形态 | 推荐导出节点 | 理由 |
|---------|------------|------|
| 单节点 | 本机（低峰窗口） | 没得选，控制导出对业务的冲击 |
| 主从，slave 纯备库 | **slave** | 最优解：主库零干扰，slave 延迟上升无害 |
| 主从，slave 承接读流量 | 主库（低峰）或临时加备库 | slave 延迟抖动会让读到旧数据 |
| 读写分离，ro 服务读 | 视延迟容忍度定 | ro 导出保护了主库（P95 -17%），但 ro 延迟冲到 12s，业务读到旧数据是否可接受要单独评估 |
| 读写分离，ro 有富余/专用导出副本 | ro/专用副本 | 兼得主库保护和数据一致 |

### 5. 会遇到的问题与解决方案

#### 5.1 导入速度慢的根因

实测数据（4.2GB / 2000 万行，单节点组）：

| 环节 | 耗时 | 占比 |
|------|------|------|
| 导出（源库） | 60s | 10% |
| 压缩（pigz） | 43s | 7% |
| **导入（目标库默认配置）** | **529s** | **84%** |
| 导入（调参三板斧后） | 333s | -37% |

根因三层：

1. **单线程**：一个连接串行执行所有 INSERT，无法利用多核
2. **binlog 双写**：目标库默认把导入的每个事务写进 binlog，纯浪费（导入数据不需要复制出去）
3. **双 1 刷盘**：每个事务两次 fsync

调参三板斧（3.5 节）能砍 37%，但单线程的天花板还在——这是工具的结构性限制，只能靠第三篇的多线程方案突破。

#### 5.2 生产级导入参数清单：先防错，再提速

3.5 节的三板斧是最核心的三件，生产上完整的导入参数配置分两类：**兼容性参数（防中途报错）**和**性能参数（提速）**。下面这份清单来自生产实践，导入前逐项核对。

**兼容性三参数（防导入中途报错）**

| 参数 | 临时值 | 防什么坑 |
|------|--------|---------|
| `innodb_large_prefix` | ON | 老版本目标库上超长索引前缀（>767 字节）直接建不出来；8.0 已默认开启且废弃，5.7 及以下目标库要注意 |
| `txsql_json_full_precision` | ON | （腾讯云 TXSQL 内核参数）JSON 列全精度存储：源库 JSON 字段含高精度小数时，目标库不开启会精度丢失 |
| `innodb_strict_mode` | OFF | 关闭 InnoDB 严格模式：行大小超限等在严格模式下直接报错中断，宽松模式降级为警告，先导进去再治理 |

**性能加速：全局参数（导入完必须恢复）**

| 参数 | 临时值 | 提速原理 |
|------|--------|---------|
| `innodb_flush_log_at_trx_commit` | 0 | redo 每秒刷一次（比三板斧里的 2 更激进：断电最多丢 1 秒数据，全新目标库无所谓） |
| `innodb_doublewrite` | OFF | 关闭双写缓冲：防"半页写"损坏的机制导入期间不需要（坏了重导即可），省一遍顺序写盘 |
| `innodb_change_buffer_max_size` | 50 | change buffer 拉到 50%：大批量 INSERT 对二级索引的变更先缓冲后合并，带二级索引的大表提速明显 |
| `innodb_stats_auto_recalc` | OFF | 关闭统计信息自动重算：导入期间别让频繁的隐式 ANALYZE 添乱 |
| `max_allowed_packet` | 1G | 与导出端批次匹配，防 "Got a packet bigger than" 报错（详见 5.4） |
| `net_buffer_length` | 1M | 与导出端批次对齐 |
| `tmp_table_size` / `max_heap_table_size` | 256M | 减少导入过程中的临时表落盘 |
| `slow_query_log` | OFF | 大批量 INSERT 会被记成慢查询，慢日志既膨胀又有写入开销，导入期间直接关掉 |

**性能加速：会话级参数（只对导入连接生效，用完即走）**

| 参数 | 值 | 提速原理 |
|------|-----|---------|
| `unique_checks` | 0 | 跳过唯一性检查：InnoDB 信任导入数据无重复，省掉每行的唯一索引校验 |
| `foreign_key_checks` | 0 | 跳过外键检查：有依赖关系的表乱序导入不报错 |
| `sql_log_bin` | 0 | 导入连接不写 binlog（mysqldump 产物头部自动带，`--init-command` 显式保证） |

三点提醒：

1. **全局参数导入完成后逐项恢复原值**（双 1、双写、change buffer、统计信息、慢日志），这是生产操作的高压线——忘了恢复等于把数据库裸奔上线
2. `unique_checks=0` 的前提是数据来自一致性快照、信任源库无脏数据；导入完成后建议对关键唯一索引做一次有效性抽查
3. 这份清单和三板斧不冲突：三板斧是其中的核心三件（`sql_log_bin` + 刷盘 + 检查关闭），清单是完整版

#### 5.3 长事务与 undo 膨胀

`--single-transaction` 导出 4GB 数据需要数分钟，快照期间业务的 DELETE/UPDATE 产生的旧版本行不能被 purge 回收，undo 表空间持续膨胀。如果源库写入压力大，导出前要确认 undo 空间余量。极端情况下（长导出 + 高写入），undo 能膨胀到和数据量同量级。

#### 5.4 max_allowed_packet 陷阱

导出端和导入端的 `max_allowed_packet` 不一致时：源端按自己的 `net_buffer_length` 生成了大 INSERT 批次，目标端 `max_allowed_packet` 更小，导入直接报错：

```
ERROR 1153 (08S01) at line N: Got a packet bigger than 'max_allowed_packet' bytes
```

解决：导入前确认目标端 `max_allowed_packet >= net_buffer_length`（导出端），或导出时用 `--net-buffer-length=64K` 调小批次。

#### 5.5 GTID 环境的 --set-gtid-purged 陷阱

源库开启 GTID 时，mysqldump 默认会在产物头部加 `SET @@GLOBAL.gtid_purged='uuid:1-N'`（`--set-gtid-purged=ON`），这个默认行为是坑还是宝，取决于目标库的状态。两种方案：

**方案 A：目标库已有数据/GTID 记录 → `--set-gtid-purged=OFF`**

目标库已有自己的 GTID 执行历史时，导入带 `SET @@GLOBAL.gtid_purged` 的产物会直接报错 `@@SESSION.GTID_NEXT cannot be changed`，或污染目标库的 GTID 集合导致后续搭复制出诡异问题。这种场景导出时显式加 `--set-gtid-purged=OFF`，增量追赶用自己的位点记录单独管理。

**方案 B：目标库是干净空白的新实例 → 保留默认（推荐生产使用）**

如果目标库是一个全新空白的实例（`gtid_executed` 为空），**建议保留 `--set-gtid-purged=ON` 的默认行为**，彻底忽略目标库、完全以源库的 GTID 为主：

```bash
# 导出（不加 --set-gtid-purged=OFF，保留 GTID 信息）
mysqldump --single-transaction --source-data=2 --databases sbtest ... > dump.sql

# 导入后，目标库的 gtid_executed 与源库导出时刻完全一致
mysql> SELECT @@GLOBAL.gtid_executed;   -- uuid:1-N，与源库相同
```

生产环境里方案 B 通常更好，因为**导入后的目标库和源库在 GTID 维度上完全相同**：

1. 全量导入的这批数据，在目标库上"看起来就像源库自己执行过的一样"——GTID 历史无缝继承
2. 后续增量追赶（割接阶段）直接用 GTID 复制接上源库：`CHANGE REPLICATION SOURCE TO ... SOURCE_AUTO_POSITION=1`，源库发来的事务里凡是目标库 GTID 集合中已有的，自动幂等跳过，不会重复执行
3. 迁移完成后如果要把目标库升格为源库的从库（做双活校验或回切兜底），GTID 体系天然成立，不需要任何位点换算

**方案 B 的铁律**：目标库必须是 `RESET MASTER` 级别的干净实例（生产迁移的目标端本来就是新机器/新实例，天然满足）。目标库有一丝自己的 GTID 历史，方案 B 就退化成报错现场。

一句话总结：**目标库空白，让源库 GTID 全量接管（方案 B，生产首选）；目标库有历史，关掉 GTID 注入各管各的（方案 A）**。本系列实验用的是方案 A（实验目标库被多组实验反复导入，GTID 历史不干净）。

#### 5.6 字符集与 collation 兼容

字符集的坑分两层：

**第一层：collation 不兼容**。源库 utf8mb4_0900_ai_ci（8.0 默认）导到 5.7 目标库会报错（5.7 没有这个 collation）。跨大版本向下迁移要显式指定 `--default-character-set`，或者接受 collation 降级。

**第二层：源和目标字符集不同**（比如源是 utf8mb3、目标是 utf8mb4）。

utf8mb3 是 utf8mb4 的子集，字节层面天然兼容，看起来"升格"很安全——但坑藏在**导出/导入连接的字符集转换层**：连接字符集和数据字符集不一致时，server 会按连接字符集对字符串做一次转码再吐给客户端，产物文件里的字节已经不是源库里的原始字节了；这个转换一旦遇到不可映射字符，轻则乱码、重则报错中断。

解法：**导出时让字符串列走 `_binary` 语义，按原始字节导出，绕过连接层转码**：

```bash
# --default-character-set=binary：导出连接用二进制字符集，
# 字符串数据按源库里的原始字节写入产物，不做任何转码
mysqldump --default-character-set=binary --single-transaction ... > dump.sql
```

产物里的 INSERT 语句是字节的"原样搬运"，导入到 utf8mb4 目标库时按目标列字符集直接落盘——utf8mb3 的字节在 utf8mb4 下全部合法且含义不变，数据零损耗。对含 BLOB 的表再加 `--hex-blob`，二进制列以十六进制字面量（`0x...`）导出，彻底杜绝编码歧义。

直白记法：**字符集不同先别慌，`binary` 导出保平安**——字节搬运比字符转换可靠，只要"源字符集是目标字符集的子集"（utf8mb3→utf8mb4、latin1→utf8mb4 前提是确认数据确实是纯 latin1），这条都成立。反过来（utf8mb4→utf8mb3）是降格，4 字节 emoji 无处安放，先清洗数据再迁。

#### 5.7 导出对源库的冲击

实测压测数据（完整分析见 4.2 节）：

- 本实验数据量（4.2GB）≈ buffer pool（4G）时，导出甚至因"全量预热"让压测 TPS 不降反升（+4%~12%）
- **数据量远超 buffer pool 时结论反转**：全表扫描持续挤掉热页（buffer pool 污染），业务 TPS 退化、P95 上升——这是生产库导出的常态
- 退化三大来源：buffer pool 污染、导出连接占用网络带宽、`--single-transaction` 长事务撑大 purge 滞后（undo 膨胀，见 5.3）

#### 5.8 从库/只读节点导出的位点问题

从只读副本导出（给主库减压）时：副本通常没开 binlog，`--source-data` 的 file+pos 位点拿不到——**这正好是 GTID 位点的主场（2.4 节）**：GTID 环境下导出后立即在副本上执行 `SHOW REPLICA STATUS` 取 `Executed_Gtid_Set`（副本 SQL 线程已回放的事务集合，即快照对应的精确位点），增量追赶直接 `SOURCE_AUTO_POSITION=1`，比手工换算 file/pos 可靠得多。唯一要注意：这个集合比源库当前位点旧，增量追赶时确认源库 binlog 保留时长（或 GTID 保留）覆盖这个差值。

### 6. 适用数据量级结论

生产经验值：**mysqldump 的适用范围在 300GB 以内**——生产机器规格高、网络快，配合低峰窗口和三板斧调参，300GB 以内的迁移完全能撑住（按本实验单机折算全流程约 12.5 小时，生产高配机器可显著缩短）。

需要客观看待的边界：

| 关注点 | 说明 |
|--------|------|
| 耗时随数据量线性增长 | 单线程导入没有并行加速，数据量翻倍时间翻倍，300GB 是"窗口能等"的经验上限 |
| 迁移窗口要求 | 有明确窗口要求（比如必须 4 小时内完成）的，300GB 以内也要先实测折算 |
| 超出 300GB | 不勉强，直接看第三篇 mydumper/myloader |

实验数据参考（2000 万行 / 4.2GB：导出 60s + 压缩 43s + 导入 529s ≈ 10.5 分钟全流程，调参后 6.5 分钟）。

直白结论：**300GB 以内、迁移窗口不敏感（可以慢慢导）的场景，mysqldump 依然是生产上最省心的选择；数据量再大或者有明确窗口要求的，看第三篇 mydumper/myloader**。

### 7. 总结

1. mysqldump 导出环节其实不慢（60 秒导完 2000 万行），**瓶颈全在导入**（529 秒，占全流程 84%）：单线程 + 双 1 刷盘 + binlog 双写是三座大山，三板斧调参砍掉 37%
2. 导出对源库的冲击取决于数据量与 buffer pool 的比例：数据量小于内存时甚至有"预热红利"（TPS 不降反升）；数据量远超内存时 buffer pool 污染会让业务明显退化——生产库属于后者，导出要选低峰或走从库
3. 导出节点选择是个架构问题（4.4 决策树）：主从架构的 slave 若是纯备库，从 slave 导出是最优解（主库零干扰、延迟上升无害）；副本服务读流量时才需要权衡——从 ro 导出保护主库（P95 低 17%）但副本复制延迟冲到 12 秒
4. 导入前参数分两步配：兼容性三参数（`innodb_large_prefix`、`txsql_json_full_precision`、`innodb_strict_mode`）防中途报错，性能加速 12 项（关双写、拉 change buffer、跳检查等）提速——导入完成后全局参数必须逐项恢复；GTID、字符集、max_allowed_packet、DROP TABLE 语义，四个坑一个都不能踩
5. 量级结论（生产经验）：300GB 以内放心用，超出后果断换 mydumper/myloader

下一篇用多线程的 mydumper/myloader 重新跑同样的实验，看看多线程能带来多大提升：

下一篇：[MySQL 数据迁移（三）mydumper/myloader 多线程迁移](https://blog.csdn.net/a18792721831/article/details/166594700)

---

版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
