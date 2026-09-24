---
layout: post
title: "MySQL 数据迁移（三）mydumper/myloader 多线程迁移实战与评测"
date: 2026-09-24 15:34:58 +0800
categories: ["MySQL", "数据迁移", "mysqldump", "mydumper", "myloader", "数据库", "DTS"]
description: "mysqldump 单线程导入 2000 万行要几十分钟，mydumper/myloader 用多线程把导出导入都并行化，是中大数据量逻辑迁移的主力工具。本篇在同样的 2000 万行数据集、同样的压测条件下实战 mydumper 多线程导出和 myloader 多线程导入，对比线程数、调参前后的性能差异，并给出它相对 mysqldump 的完整性能对照表。"
keywords: ["MySQL", "数据迁移", "mysqldump", "mydumper", "myloader", "数据库", "DTS"]
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/166594700
> - 发布时间：2026-09-24 15:34:58
> - 标签：#mysql, #数据库, #mysqldump, #mydumper, #myloader, #DTS

---

## MySQL 数据迁移（三）mydumper/myloader 多线程迁移实战与评测

### 摘要

mysqldump 单线程导入 2000 万行要几十分钟，mydumper/myloader 用多线程把导出导入都并行化，是中大数据量逻辑迁移的主力工具。本篇在同样的 2000 万行数据集、同样的压测条件下实战 mydumper 多线程导出和 myloader 多线程导入，对比线程数、调参前后的性能差异，并给出它相对 mysqldump 的完整性能对照表。

### 1. 建议先看

- [MySQL 数据迁移（一）为什么迁、怎么迁、怎么评——三大方案总览](https://blog.csdn.net/a18792721831/article/details/166593341)
- [MySQL 数据迁移（二）原生 mysqldump 导出导入实战与评测](https://blog.csdn.net/a18792721831/article/details/166593481)

本篇沿用完全相同的评测环境（单节点 / 主从 / 读写分离三套架构）和数据集（sysbench 20 表 × 100 万行 ≈ 4.2GB），保证与前两篇数据可比。

### 2. mydumper/myloader 是什么

#### 2.1 为什么快：并行化改造了 mysqldump 的两个单点

第二篇结论：mysqldump 慢在导出和导入都是**单线程**。mydumper/myloader（开源社区工具，现由 Percona 维护）对症下药：

- **导出侧（mydumper）**：主线程建立一致性快照后，把表（以及大表的分块）分发给 N 个 worker 线程并行 `SELECT`，导出吞吐随线程数扩展
- **导入侧（myloader）**：产物天然按表/按块拆分成独立文件，N 个线程各自连目标库并行执行，导入吞吐随线程数扩展

```
mysqldump:   源库 ===单线程===> dump.sql ===单线程===> 目标库

mydumper:    源库 ==8线程==> 20个表文件+大表切块 ==8线程==> 目标库
                     mydumper导出        myloader导入
```

#### 2.2 安装

CentOS/RHEL 系直接下载 release 二进制，或源码编译（依赖 glib2、mysql client 库、zlib、pcre）：

```bash
# CentOS 8 系（实测环境）
yum install -y mysqldumper  # 或从 github.com/mydumper/mydumper 下载 release

$ mydumper --version
mydumper v1.0.0-1, built against MySQL 8.0.36 with SSL support
```

### 3. 实战：导出与导入

#### 3.1 mydumper 导出

```bash
mydumper \
  --host=*** --port=13306 \
  --user=sbtest --password=*** \
  --database=sbtest \
  --outputdir=/backup/mydumper_single \
  --threads=8 \
  --rows=250000 \
  --compress \
  --triggers --routines --events \
  --verbose=2
```

关键参数：

| 参数 | 作用 | 说明 |
|------|------|------|
| `--threads=8` | 导出并发线程数 | 线程数 = 同时压源库的连接数，不是越大越好 |
| `--rows=250000` | 大表按行数切块 | 100 万行的表切成 4 块并行导出，多表场景本身天然并行 |
| `--compress` | 每个数据文件 gzip 压缩 | 导出时在线压缩，省磁盘省传输 |
| `--outputdir` | 产物目录 | mydumper 自动创建，产物是目录不是单文件 |
| `--verbose=2` | 详细日志 | 排查导出卡住必备 |
| `--triggers --routines --events` | 同 mysqldump | 少一样丢一样 |
| `--less-locking` | 减少锁时间 | 有 MyISAM 表时用 |
| `--kill-long-queries` | 杀掉阻塞导出的长查询 | **危险参数**，生产慎用 |

一致性原理：主线程执行 `START TRANSACTION WITH CONSISTENT SNAPSHOT` 拿到统一位点后，worker 线程通过 `SAVEPOINT` 机制共享同一快照视图（`--threads` 个连接看到同一份数据），保证并行导出的数据属于同一时刻。

#### 3.2 导出产物解读

```bash
$ ls /backup/mydumper_single | head -10
metadata                    # 位点与时间信息
schema-create-sbtest.sql    # 建库语句
schema-sbtest.sql           # 库级对象（存储过程/触发器等）
schema-sbtest.sbtest1.sql   # sbtest1 建表语句
sbtest.sbtest1.00000.sql.gz # sbtest1 第1块数据（25万行）
sbtest.sbtest1.00001.sql.gz # sbtest1 第2块数据
sbtest.sbtest1.00002.sql.gz
sbtest.sbtest1.00003.sql.gz
sbtest.sbtest2.00000.sql.gz
...

$ cat metadata
Started dump: host=*** database=sbtest
SHOW MASTER STATUS:
   Log: mysql-bin.000007
   Pos: 1234567
   GTID:3E11FA47-27CA-11EE-...:1-5000

SHOW SLAVE STATUS:    # 有复制关系时同时记录，方便从库导出
```

metadata 同时记录 file+pos 位点和 **GTID 事务集合**——增量追赶时优先用 GTID（第二篇 2.4 讲过：file+pos 只是字节偏移、粒度粗，GTID 精确到事务且幂等回放），mydumper 的 `--enable-binlog` 之外还有配套的位点管理就是围绕它做的。

对比 mysqldump 单文件，目录结构的三个好处：

1. **并行导入的物质基础**：文件之间无依赖，才能多线程同时执行
2. **断点友好**：单表文件损坏只重导一张表；mysqldump 单文件第 8000 行报错，后面全卡住
3. **单表级校验/修复**：可以只对某张表做 checksum 后重传

#### 3.3 myloader 导入

```bash
myloader \
  --host=*** --port=13311 \
  --user=root --password=*** \
  --directory=/backup/mydumper_single \
  --threads=8 \
  --verbose=1
```

关键参数：

| 参数 | 作用 | 说明 |
|------|------|------|
| `--threads=8` | 导入并发线程数 | 8 线程 = 目标库 8 个连接并行执行 |
| `--enable-binlog` | 导入写 binlog | **默认不写**（自动 SET sql_log_bin=0），与 mysqldump 行为相反！ |
| `--innodb-optimize-keys` | 延迟建二级索引 | 先建表只带主键 → 灌数据 → 再建二级索引，大表提速明显 |
| `--overwrite-tables` | 目标有表时 DROP 重建 | 等价 mysqldump 产物里的 DROP TABLE |
| `--resume` | 断点续传 | 从中断的文件继续 |

**注意**：myloader 默认禁用 binlog（`sql_log_bin=0`），意味着导入的数据不会出现在目标库的 binlog 里。如果目标库下面还挂着下游从库，导入完成后从库会缺这批数据——这是生产事故高发点，需要下游同步时必须加 `--enable-binlog`。

#### 3.4 与 mysqldump 的行为差异对照

| 行为 | mysqldump | mydumper/myloader |
|------|-----------|-------------------|
| 产物形态 | 单个 .sql 文件 | 目录（每表/每块一个文件） |
| 导出并发 | 1 | --threads |
| 导入并发 | 1 | --threads |
| 导入写 binlog | 默认写 | **默认不写** |
| 位点记录 | --source-data 写入文件头 | metadata 文件（file+pos + GTID 事务集合） |
| 一致性快照 | 单连接 RR 快照 | 主线程快照 + SAVEPOINT 共享 |
| 大表切块 | 不支持 | --rows / --chunk-filesize |
| 断点续传 | 不支持 | --resume |
| 中断后果 | 单文件从头再来 | 按文件粒度恢复 |

### 4. 实验评测

同样的评测方法：迁移全程对源库 8 线程压测，对比基线。**先说一个实验方法论**：本篇与第二篇的压测基线绝对值存在环境漂移（两轮实验时宿主机背景负载不同），所以跨篇的 TPS 绝对值对比不可靠，**组内"基线 vs 迁移期间"的相对变化**才是有效结论；而导出/导入**时长**不受基线影响，可以直接对比。

#### 4.1 三套架构结果总表

| 架构组 | 导出源 | 导出耗时 | 导入耗时(8线程) | 导入后校验 |
|--------|--------|---------|---------------|-----------|
| 单节点 | single 本机 | 28s | 172s | 20/20 表行数一致 |
| 主从 | master 主库 | 30s | 167s | 20/20 表行数一致 |
| 主从（补充组） | **slave 纯备库** | 30s | 187s | 20/20 表行数一致 |
| 读写分离 | ro 只读节点 | 28s | 171s | 20/20 表行数一致 |

主从组导出期间从库复制延迟最高 2 秒（8 线程并发读叠加压测写入，SQL 线程回放轻微受阻），压测一停立即归零。补充组（压测打 master、从 slave 导出）是第二篇 4.4 决策树"纯备 slave 是最优导出节点"的直接实测验证，完整数据见 4.4。

导出产物：123 个文件（20 表 × 5 块 + schema/metadata），gzip 压缩后 1904MB（裸 SQL 3810MB 的 50%）。

与第二篇 mysqldump 的直接对比（同数据集 2000 万行 / 3.8GB）：

| 指标 | mysqldump | mydumper/myloader | 提升 |
|------|-----------|-------------------|------|
| 导出耗时 | 60s | **28s** | 2.1 倍 |
| 导入耗时(默认配置) | 529s | **172s** | 3.1 倍 |
| 产物大小 | 3810MB(裸) | 1904MB(gzip) | 导出即压缩 |
| 全流程 | ~632s | ~200s | 3.2 倍 |

**多线程把全流程提速 3 倍以上**，其中导入 529s → 172s 是最大贡献。

#### 4.2 线程数对比：4 / 8 / 16

安静环境下（无压测流量），对同一份产物做不同线程数的导入：

| myloader 线程数 | 导入耗时 | 变化 |
|----------------|---------|------|
| 4 | **163s** | 最快 |
| 8 | 168s | +3% |
| 16 | 183s | **+12%，反而更慢** |

**16 线程比 4 线程慢 12%**——这是本篇最重要的反直觉数据。多线程的收益到头了：

1. **并行度天花板**：20 张表切成 80 个数据块，4 线程已经能持续吃到活干；线程再多，分块排队等锁
2. **InnoDB 内核争用**：redo log 写入、buffer pool 修改、脏页刷盘都是实例内共享资源，写入并发超过一定阈值后锁等待上升
3. **宿主机资源饱和**：CPU/磁盘 IO 就那么多，16 个连接哄抢不如 4 个排队

经验公式修正：**线程数 ≈ min(可用核数 / 2, 表数)**，20 表场景 4~8 线程是最优区间。盲目加线程只会把源库/目标库打得更疼，迁移速度反而下降。

#### 4.3 调参导入对比

myloader 默认配置已经自动 `SET sql_log_bin=0`（不写 binlog），那 mysqldump 那套"三板斧"还有多少空间？

| 导入方式 | 配置 | 耗时 |
|---------|------|------|
| myloader 默认 | sql_log_bin=0 + 双 1 | 172s |
| myloader + 双 0 调参 | sql_log_bin=0 + flush=2 + sync_binlog=0 | 174s |

**几乎零增益**（174s vs 172s，误差范围内）。对照第二篇 mysqldump 调参 37% 的收益：那 37% 的大头本来就是"关 binlog"，而 myloader 默认就把它做了；剩下的双 1 → 双 0 对 8 线程批量写入的边际收益小到可以忽略。**myloader 的默认配置已经站在调优后的起点上**——这是工具设计层面的差距，mysqldump 用户要手工做对的事它默认做对了。

#### 4.4 对源库的冲击：mydumper vs mysqldump

读写分离架构下的组内对比（压测打 rw，导出走 ro）：

| 工具 | 基线 TPS/P95 | 导出期间 TPS/P95 | 组内变化 |
|------|-------------|-----------------|---------|
| mysqldump（第二篇） | 260.69/30.81ms | 286.25/25.28ms | TPS +10%，P95 -18% |
| mydumper 8线程（本篇） | 370.93/26.20ms | 323.01/32.53ms | **TPS -13%，P95 +24%** |

**同样是"从 ro 导出"，单线程的 mysqldump 让主库更舒服，8 线程的 mydumper 反而拖累了主库**。原因：ro 节点的 8 线程全表扫描吃掉的宿主机 CPU/IO，与同机部署的 rw 争抢资源；ro 自身还要回放 rw 的 binlog，压力三重叠加。

**补充实测：m-s 架构从 slave 纯备库导出**（压测打 master，mydumper 8 线程从 slave 导出）：

| 指标 | 数值 | 说明 |
|------|------|------|
| master 压测 TPS | 411.89 → 356.91（**-13%**） | 与"从 ro 导出"组（-13%）几乎一致 |
| master 压测 P95 | 26.68ms → 31.94ms（+20%） | 同上 |
| slave 自身复制延迟 | **最高 10s** | 导出完延迟归零 |
| 导出 / 导入 | 30s / 187s | 与其他组一致 |

这组数据带来一个重要发现：**从 slave 导出和从 ro 导出，主库的退化幅度相同（都是 -13%）**——说明 mydumper 多线程导出对主库的影响主要不是"导出打在谁身上"，而是**宿主机共享资源（CPU/IO）被抢占后向同机所有实例传导**。它验证了第二篇 4.4 的等价论证，也给出了纯备 slave 场景的精确预期：主库退化约 13%、slave 复制延迟最高 10 秒（纯备库不服务读，这个延迟无害）、导出导入时长不受影响。

结合 4.2 的线程数数据，结论收敛：**mydumper 的线程数要按源库的"空闲资源"来配，而不是"越大越快"**。生产建议：

- 源库有业务流量时：线程数 2~4，错峰执行
- 源库专门腾出来做迁移（已停业务）：线程数可以给到核数
- 主从架构 slave 为纯备库时：从 slave 导出（第二篇 4.4 决策树的最优解），线程可以放得更开——slave 上除了复制回放没有别的活；但注意本实验主从同宿主机部署、主库 TPS 仍降了 13%，主从独立部署的生产环境没有这层资源竞争
- 无论哪种，先在测试环境验证一轮

#### 4.5 与 mysqldump 方案总对照

| 维度 | mysqldump | mydumper/myloader | 实测数据 |
|------|-----------|-------------------|---------|
| 导出速度 | 慢 | 快 | 60s → 28s |
| 导入速度 | 很慢 | 快 | 529s → 172s |
| 导出对源库冲击 | 小 | 大 | TPS -13% vs +10% |
| 线程可调 | 否 | 是 | 16 线程反而比 4 慢 |
| 产物形态 | 单文件 | 目录+分块 | 断点/单表修复友好 |
| binlog 记录 | 默认写 | 默认不写 | 下游从库注意 |
| 适用量级 | <300GB | 300GB~15TB | 见第 6 节 |

### 5. 会遇到的问题与解决方案

#### 5.1 线程数不是越大越好

线程数翻倍，源库连接数、网络带宽、磁盘 IO 同步翻倍。实测数据（4.2 节）：

- 4 线程：163s（最快）
- 8 线程：168s
- 16 线程：183s（最慢，比 4 线程慢 12%）

经验值：**线程数 = min(源库可用核数的一半, 表数)**。20 张表的场景，4~8 线程已经吃满并行度；开到 16 只会让源库压测退化更狠（TPS -13%），导入时间却更 长。

#### 5.2 --rows 切块过碎的碎片问题

`--rows=1000` 会把 100 万行的表切成 1000 个小文件，每个文件才几百 KB：文件系统 inode 压力、myloader 线程调度开销、小块数据无法填满网络包。经验：每块 25 万～100 万行（或用 `--chunk-filesize=256` 按大小切块）。

#### 5.3 metadata 里的位点只对源库有效

从 ro/从库节点导出时，metadata 记录的是**该节点自己的复制状态**（SHOW SLAVE STATUS 部分）。file+pos 模式下要手工换算回源库位点，容易踩"位点差一个事务"的坑——**GTID 模式则天然免换算**：副本 `SHOW REPLICA STATUS` 的 `Executed_Gtid_Set` 就是它已回放的事务集合，即导出快照的精确位点，增量追赶直接 `SOURCE_AUTO_POSITION=1`，源库按集合自动协商、幂等跳过已有事务（第二篇 2.4 的结论在这里同样成立）。

#### 5.4 字符集/版本兼容

mydumper 产物同样是 SQL 文本，跨版本 collation 问题和 mysqldump 完全一致（见第二篇 5.6），导入参数清单（第二篇 5.2）同样适用。源和目标字符集不同（如 utf8mb3 → utf8mb4）时，同样用 `--default-character-set=binary` 让字符串按原始字节导出，绕过连接层转码。

#### 5.5 权限要求更高

mydumper 需要 `SELECT` + `RELOAD`（执行 FLUSH TABLES）+ `REPLICATION CLIENT`（取位点），普通业务账号往往没配齐，报错 `Access denied` 时先查这三项。

### 6. 适用数据量级结论

生产经验值：**mydumper/myloader 的适用范围在 15TB 以内**——这是生产上逻辑迁移的主力区间，多线程摊薄了 SQL 解析的行级成本，生产高配机器（更多核、更快盘）能把线程数和吞吐进一步拉高。

需要客观看待的边界：

| 关注点 | 说明 |
|--------|------|
| 行级成本刚性存在 | 每行都要过解析器，数据量翻倍导入时间近似翻倍，15TB 是"逻辑导入时间还能接受"的经验上限 |
| 线程数有甜蜜点 | 见 4.2 节，盲目加线程适得其反 |
| 超出 15TB | 逻辑迁移的耗时开始失控，看第四篇物理迁移 |

实验数据参考（2000 万行 / 4.2GB：导出 28s + 导入 172s ≈ 200s 全流程）。

直白结论：**300GB ~ 15TB 是 mydumper/myloader 的主场**。再往上，逻辑迁移"逐行过解析器"的税逃不掉，物理迁移才是出路。

### 7. 总结

1. 多线程实打实快：导出 2.1 倍、导入 3.1 倍，全流程 632s → 200s
2. 但线程数有甜蜜点：4~8 线程最优，16 线程反而慢 12%——InnoDB 内核争用 + 宿主机资源饱和，"给多少用多少"是错的
3. 8 线程导出对源库的冲击比单线程大得多（同场景 TPS 从 +10% 变成 -13%）：源库有业务时线程要保守，错峰 + 限流是基本功
4. myloader 默认不写 binlog，目标库下游挂着从库时记得 `--enable-binlog`，否则下游静默丢数据
5. 产物目录结构天然支持断点续传和单表修复，这是比速度更重要的工程价值

逻辑迁移到这里就到顶了。下一篇直接拷数据文件，绕过 SQL 层——

下一篇：[MySQL 数据迁移（四）数据文件物理迁移](https://blog.csdn.net/a18792721831/article/details/166594750)

---

版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
