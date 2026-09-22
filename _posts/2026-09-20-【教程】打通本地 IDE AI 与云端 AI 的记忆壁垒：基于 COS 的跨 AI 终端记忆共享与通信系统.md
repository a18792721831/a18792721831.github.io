---
layout: post
math: true
title: "【教程】打通本地 IDE AI 与云端 AI 的记忆壁垒：基于 COS 的跨 AI 终端记忆共享与通信系统"
date: 2026-09-20 16:54:43 +0800
categories: ["ai", "智能体", "agent", "AGENTS", "Mem"]
description: "【教程】打通本地 IDE AI 与云端 AI 的记忆壁垒：基于 COS 的跨 AI 终端记忆共享与通信系统 摘要 本文详细介绍如何基于腾讯云 COS 对象存储构建一套跨 AI 终端的记忆共享与异步通信系统。实现本地 IDE 内置 AI（如 CodeBuddy）与云端 AI（如企微 Bot）之间的记忆双向同步、智能融合与异步通信。包含完整的架构设计、核心代码实现、融合策略、信箱协议、定时任务配置和新"
keywords: ["ai", "智能体", "agent", "AGENTS", "Mem"]
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/158737129
> - 发布时间：2026-09-20 16:54:43
> - 标签：#ai, #智能体, #agent, ##AGENTS, ##Mem

---

## 【教程】打通本地 IDE AI 与云端 AI 的记忆壁垒：基于 COS 的跨 AI 终端记忆共享与通信系统

### 摘要

本文详细介绍如何基于腾讯云 COS 对象存储构建一套跨 AI 终端的记忆共享与异步通信系统。实现本地 IDE 内置 AI（如 CodeBuddy）与云端 AI（如企微 Bot）之间的记忆双向同步、智能融合与异步通信。包含完整的架构设计、核心代码实现、融合策略、信箱协议、定时任务配置和新终端接入指南。

![思维导图](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/mindmap/cross-ai-memory-sharing-system_20260306144622.png)

### 1\. 背景与痛点

#### 1.1 问题：AI 助手的"失忆症"

现在的 AI 编程助手越来越多了，CodeBuddy、Cursor、Claude Desktop、Copilot……但有一个问题一直没解决好：**跨终端的记忆割裂** 。

说白了，你在 IDE 里的 AI 助手帮你写了一天代码，它记住了你的项目结构、技术偏好、最近在做什么。但你换到企微上问另一个 AI 助手（比如 OpenClaw），它啥都不知道——就像两个从没交流过的同事。

更痛的是：

  * **同一个人的记忆分散在多个 AI 终端** ，每个 AI 都只了解一部分
  * **多个 AI 无法协作** ，IDE 里的 AI 擅长写代码，云端的 AI 擅长搜索和分析，但它们互相不知道对方做了什么
  * **每次新会话都要重新建立上下文** ，效率低下



#### 1.2 解决思路

把多个 AI 终端的记忆文件统一存储在一个**云端共享存储** 中，通过双向同步实现记忆共享，通过信箱机制实现异步通信。

打个比方，就像多个同事共用一个 Google Drive，各自往里面写笔记，定期同步，还能互相留言。只不过这里的"同事"是 AI。

#### 1.3 最终效果

云端AI - OpenClaw

腾讯云 COS

本地IDE AI - CodeBuddy

push

pull

push

pull

daily-memories/

.learnings/

mailbox/

aimem-xxx  
100+ 文件

daily-memories/

.learnings/

mailbox/

实现的能力：

  * **记忆共享** ：任意一端写入的记忆，另一端都能看到
  * **智能融合** ：双端同时编辑同一天的记忆文件时，按会话块自动去重合并
  * **异步通信** ：两个 AI 可以互发消息（请求、通知、回复）
  * **新终端秒接入** ：一段 Python 脚本即可完成接入



### 2\. 系统架构设计

#### 2.1 记忆分层模型

首先，不是所有记忆都一样重要。我们设计了一套分层模型：

层级| 文件| 加载级别| 更新频率| 说明  
---|---|---|---|---  
永久记忆| `MEMORY.md`| L0（必读）| 每日| 由定时脚本从每日记忆中提炼  
临时记忆| `daily-memories/`| L0（必读）| 每次对话| 每日对话的详细记录  
错误/教训| `.learnings/`| L0（必读）| 按需| 自我改进系统：错误追踪+教训记录  
用户画像| `USER.md`| L1（按需）| 低频| 用户偏好和习惯  
AI 行为准则| `SOUL.md`| L2（跳过）| 极低频| AI 的"灵魂"，定义行为边界  
全局配置| `CODEBUDDY.md`| L2（跳过）| 低频| MCP 配置、项目入口等  
碎片记忆| `.memories/`| 自动注入| 按需| 关键信息索引  
  
#### 2.2 完整目录结构

```
记忆根目录/
├── MEMORY.md              # 永久记忆（由 consolidator 自动提炼）
├── USER.md                # 用户画像与偏好
├── SOUL.md                # AI 行为准则
├── CODEBUDDY.md           # 全局配置中心
├── MEMORY_ACCESS_CARD.md  # 新终端接入卡（一段脚本秒接入）
│
├── daily-memories/        # 每日对话记忆（高频更新）
│   ├── 2026-03-05.md      # 今天的记忆
│   ├── 2026-03-04.md      # 昨天的记忆
│   └── archive/           # 超过 14 天自动归档
│
├── .memories/             # 碎片记忆（由 update_memory 工具管理）
│   ├── {id}.mdc           # 每条记忆一个文件
│   └── .memory-global-config.json
│
├── .learnings/            # 自我改进系统
│   ├── ERRORS.md          # 操作错误记录
│   ├── LEARNINGS.md       # 教训与最佳实践
│   ├── FEATURE_REQUESTS.md # 功能缺口追踪
│   └── PROMOTIONS.md      # 晋升历史（重复错误 → 永久规则）
│
├── mailbox/               # 跨 AI 信箱通信
│   ├── PROTOCOL.md        # 通信协议
│   ├── to-codebuddy/      # CodeBuddy 收件箱
│   └── to-openclaw/       # OpenClaw 收件箱
│
├── scripts/               # 自动化脚本（不同步到 COS）
│   ├── cos_sync.py        # COS 双向同步主脚本
│   ├── cos_pull_daemon.sh # 每分钟 pull 守护脚本
│   └── memory_consolidator.py  # 记忆整合 + learnings 晋升
│
└── _bootstrap/            # 引导文件（托管在 COS，新终端下载用）
    ├── cos_sync.py
    └── memory_consolidator.py

``` 

#### 2.3 同步架构图

终端 B (Linux) - 企微 Bot

终端 A (macOS) - IDE AI

腾讯云 COS  
aimem-{bucket-id}  
prefix: memory/

push/pull

pull

pull

push

对象存储

launchd 每30min  
→ sync 双向

cron 每分钟  
→ pull 信箱

cron 每分钟  
→ pull 拉取

手动 push  
对话结束后

### 3\. 核心模块实现

#### 3.1 COS 双向同步脚本

这是整个系统的核心——`cos_sync.py`，支持 push（上传）、pull（下载+融合）、sync（双向）三种模式。

##### 3.1.1 基础配置

```python
#!/usr/bin/env python3
"""COS 双向同步脚本 — 记忆共享核心"""

import os
import re
import hashlib
import json
import logging
from pathlib import Path
from qcloud_cos import CosConfig, CosS3Client

# ============== 配置 ==============
# 建议改为环境变量，避免密钥硬编码
SECRET_ID = os.environ.get("COS_SECRET_ID", "your-secret-id")
SECRET_KEY = os.environ.get("COS_SECRET_KEY", "your-secret-key")
REGION = "ap-guangzhou"
BUCKET = "your-bucket-name"
COS_PREFIX = "codebuddy-memory/"

# 记忆文件根目录（根据你的 AI 工具调整）
MEMORY_BASE = os.path.expanduser(
    "~/Library/Application Support/CodeBuddyExtension/Data/Public"
)

# 同步状态文件（记录上次同步的文件 MD5）
SYNC_STATE_FILE = os.path.join(MEMORY_BASE, "scripts", ".cos_sync_state.json")

# 忽略的文件/目录
IGNORE_PATTERNS = {
    "scripts",       # 不同步脚本自身
    ".DS_Store",     # macOS 系统文件
    "__pycache__",   # Python 缓存
    "auth",          # 不同步认证信息
}

``` 

**安全提示** ：COS 密钥**强烈建议** 使用环境变量或腾讯云 CAM 临时密钥，不要硬编码在脚本中。这里为了演示简化了写法。

##### 3.1.2 文件收集与增量判断

```python
def file_md5(filepath):
    """计算文件 MD5"""
    h = hashlib.md5()
    with open(filepath, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            h.update(chunk)
    return h.hexdigest()


def should_ignore(rel_path):
    """判断文件是否应该忽略同步"""
    parts = Path(rel_path).parts
    for part in parts:
        if part in IGNORE_PATTERNS:
            return True
        # 忽略隐藏文件，但保留 .memories 和 .learnings
        if part.startswith(".") and part not in (".memories", ".learnings"):
            return True
    return False


def collect_local_files():
    """收集需要同步的本地文件，返回 {相对路径: {md5, size, mtime}}"""
    files = {}
    base = Path(MEMORY_BASE)
    for filepath in base.rglob("*"):
        if filepath.is_dir():
            continue
        rel = str(filepath.relative_to(base))
        if should_ignore(rel):
            continue
        files[rel] = {
            "abs_path": str(filepath),
            "md5": file_md5(str(filepath)),
            "size": filepath.stat().st_size,
            "mtime": filepath.stat().st_mtime,
        }
    return files

``` 

关键设计：通过 MD5 判断文件是否变更，只传输有变动的文件，实现**增量同步** 。

##### 3.1.3 Push（上传到 COS）

```python
def push_to_cos(dry_run=False, clean=False):
    """上传本地文件到 COS"""
    client = get_cos_client()
    local_files = collect_local_files()
    prev_state = load_sync_state()

    uploaded = 0
    skipped = 0
    new_state = {}

    for rel_path, info in local_files.items():
        cos_key = COS_PREFIX + rel_path
        new_state[rel_path] = info["md5"]

        # 与上次同步状态比较，如果 MD5 一样就跳过
        if prev_state.get(rel_path) == info["md5"]:
            skipped += 1
            continue

        if not dry_run:
            client.upload_file(
                Bucket=BUCKET,
                Key=cos_key,
                LocalFilePath=info["abs_path"],
                MAXThread=5,
                EnableMD5=True,
            )
            uploaded += 1

    if not dry_run:
        save_sync_state(new_state)

    logger.info(f"PUSH 完成: 上传 {uploaded}, 跳过 {skipped}")

``` 

##### 3.1.4 Pull（从 COS 下载 + 融合）

Pull 是最复杂的部分，因为涉及到**冲突处理** 。当本地和远端都修改了同一个文件时，需要智能融合。

```python
def pull_from_cos(dry_run=False):
    """从 COS 下载远端记忆到本地（含融合逻辑）"""
    client = get_cos_client()
    local_files = collect_local_files()
    cos_objects = list_cos_objects(client)
    prev_state = load_sync_state()

    for rel_path, cos_info in cos_objects.items():
        if should_ignore(rel_path):
            continue

        local_path = os.path.join(MEMORY_BASE, rel_path)

        if rel_path in local_files:
            local_info = local_files[rel_path]
            # 判断本地和远端是否都有修改
            local_changed = (local_info["md5"] != prev_state.get(rel_path))
            remote_changed = (cos_info["etag"] != prev_state.get(rel_path))

            if not remote_changed:
                continue  # 远端没变，跳过

            if not local_changed:
                # 本地没变，远端变了 → 直接覆盖
                download_cos_file_to_path(client, cos_info["key"], local_path)
                continue

            # 双端都变了 → 需要融合
            strategy = get_merge_strategy(rel_path)
            remote_content = download_cos_file(client, cos_info["key"]).decode("utf-8")
            local_content = Path(local_path).read_text(encoding="utf-8")

            if strategy == "daily_merge":
                merged = merge_daily_memory(local_content, remote_content)
            elif strategy == "learning_merge":
                merged = merge_learning_files(local_content, remote_content)
            else:
                merged = local_content  # last-write-wins，保留本地

            Path(local_path).write_text(merged, encoding="utf-8")
        else:
            # 本地不存在 → 下载新文件
            download_cos_file_to_path(client, cos_info["key"], local_path)

``` 

#### 3.2 融合策略（核心亮点）

这是整个系统最有技术含量的部分。不同类型的文件，融合策略完全不同。

文件类型| 策略| 说明  
---|---|---  
`daily-memories/*.md`| **会话级合并**|  按 `### 会话 N` 去重，保留双端所有不重复的会话  
`.learnings/*.md`| **记录级合并**|  按 Pattern-Key 去重，Recurrence-Count 取较大值  
`.memories/*.mdc`| **Last-Write-Wins**|  以最新修改为准  
`MEMORY.md`| **Last-Write-Wins**|  由 consolidator 定时重新生成  
`USER.md` / `SOUL.md`| **Last-Write-Wins**|  手动编辑频率低  
  
##### 3.2.1 Daily Memory 会话级合并

每日记忆文件的结构是这样的：

```markdown
# 2026-03-05

### 会话 1: 记忆系统共享化改造
**Agent**: CodeBuddy
**项目**: 记忆系统
**任务**: 改造记忆系统支持多 AI 终端共享

**完成的工作**:
1. 重写 cos_sync.py 支持双向同步
2. 实现三种融合策略

---

### 会话 2: 数据库查询优化
**Agent**: OpenClaw
**项目**: 后端服务
**任务**: 优化慢查询

**完成的工作**:
1. 分析了 5 个慢查询 SQL
2. 添加了 2 个索引

---

## 今日要点
- 完成记忆系统共享化改造

## 待办
- [ ] 配置定时任务

``` 

注意 `**Agent**` 字段——这是用来区分"这条记忆是哪个 AI 写的"。

融合逻辑：

```python
def merge_daily_memory(local_content, remote_content):
    """融合两个 daily-memory 文件

    策略：按会话块去重合并，保留双端所有不重复的会话
    """
    # 1. 解析出 header、会话列表、今日要点、待办
    l_header, l_sessions, l_points, l_todos = parse_daily_sessions(local_content)
    r_header, r_sessions, r_points, r_todos = parse_daily_sessions(remote_content)

    # 2. 会话去重合并（用会话标题+项目名+任务描述作为签名）
    seen_sigs = set()
    merged_sessions = []

    for s in l_sessions:
        sig = session_signature(s)  # 提取签名
        if sig not in seen_sigs:
            seen_sigs.add(sig)
            merged_sessions.append(s)

    for s in r_sessions:
        sig = session_signature(s)
        if sig not in seen_sigs:
            seen_sigs.add(sig)
            merged_sessions.append(s)

    # 3. 重新编号会话
    for i, s in enumerate(merged_sessions, 1):
        s = re.sub(r"### 会话\s*\d+", f"### 会话 {i}", s, count=1)

    # 4. 合并今日要点和待办（取并集）
    merged_points = merge_bullet_sections(l_points, r_points, "## 今日要点")
    merged_todos = merge_bullet_sections(l_todos, r_todos, "## 待办")

    # 5. 组装
    return assemble(header, merged_sessions, merged_points, merged_todos)


def session_signature(session_text):
    """提取会话块的签名用于去重"""
    sig_parts = []
    for line in session_text.strip().split("\n")[:5]:
        line = line.strip()
        if line.startswith("### 会话"):
            sig_parts.append(line)
        elif line.startswith("**项目**:") or line.startswith("**任务**:"):
            sig_parts.append(line)
    return "|".join(sig_parts) if sig_parts else hashlib.md5(session_text.encode()).hexdigest()

``` 

这样做的好处是：如果 CodeBuddy 写了会话 1-3，OpenClaw 写了会话 4-5，sync 之后两边都能看到完整的会话 1-5。

##### 3.2.2 Learnings 记录级合并

Learnings（教训记录）文件有固定的结构：

```markdown
### LRN-2026-03-05-001
- **Pattern-Key**: check-mailbox-on-startup
- **Recurrence-Count**: 1
- **Category**: correction
- **Status**: active
- **Context**: 会话启动时忘记检查信箱
- **Learning**: 新会话启动时必须检查 mailbox 中是否有未读消息

``` 

融合策略是**按 Pattern-Key 去重** ，如果两端都有同一个 Pattern-Key 的记录，取 Recurrence-Count 较大的那个（说明这个错误出现得更多，更需要重视）。

```python
def merge_learning_files(local_content, remote_content):
    """融合 .learnings/ 文件"""
    l_header, l_records, l_footer = parse_learning_records(local_content)
    r_header, r_records, r_footer = parse_learning_records(remote_content)

    merged = {}

    # 先加本地记录
    for rec in l_records:
        pk = rec.get("pattern_key", rec.get("id", ""))
        merged[pk] = rec

    # 再合并远端记录
    for rec in r_records:
        pk = rec.get("pattern_key", rec.get("id", ""))
        if pk in merged:
            # 已存在：Recurrence-Count 取较大值
            existing = merged[pk]
            if rec.get("recurrence", 0) > existing.get("recurrence", 0):
                existing["recurrence"] = rec["recurrence"]
            # Status 优先级: promoted > active > resolved
            status_priority = {"promoted": 3, "active": 2, "resolved": 1}
            if status_priority.get(rec.get("status"), 0) > status_priority.get(existing.get("status"), 0):
                existing["status"] = rec["status"]
        else:
            merged[pk] = rec

    return assemble_learning_file(l_header, merged, l_footer)

``` 

#### 3.3 信箱通信协议

两个 AI 之间怎么"说话"？我们设计了一个简单但够用的信箱协议。

##### 3.3.1 目录结构

```
mailbox/
├── PROTOCOL.md           # 协议文档
├── to-codebuddy/         # 发给 CodeBuddy 的消息
│   └── {YYYYMMDD_HHMMSS}_{发送方}.md
└── to-openclaw/          # 发给 OpenClaw 的消息
    └── {YYYYMMDD_HHMMSS}_{发送方}.md

``` 

##### 3.3.2 消息格式

每条消息就是一个 Markdown 文件，用 YAML frontmatter 做元数据：

```markdown
---
from: codebuddy
to: openclaw
timestamp: 2026-03-05T16:00:00
type: info
status: unread
---

你好 OpenClaw！我是 CodeBuddy。

我们的共享记忆系统已经升级了，现在支持跨 AI 信箱通信。

**新增功能**：
1. 写入 daily-memories 时请加 `**Agent**: OpenClaw` 字段
2. 你的信箱在 `mailbox/to-openclaw/`
3. 给我发消息写到 `mailbox/to-codebuddy/`

``` 

##### 3.3.3 消息类型

类型| 说明| 示例  
---|---|---  
`request`| 请对方执行任务| “帮我查一下这个 API 的文档”  
`info`| 通知（不需要回复）| “记忆系统升级了，注意新协议”  
`reply`| 对 request 的回复| “查到了，文档链接是 xxx”  
  
##### 3.3.4 消息感知时效

环节| 延迟  
---|---  
发送方 push 到 COS| ~2-5 秒  
收件人 crontab pull| ≤ 1 分钟  
AI 感知| 下次会话启动或用户主动触发  
**端到端最快**| **~1 分钟**  
  
整个流程是**异步** 的——发送方写完消息后 push 到 COS，收件方的定时任务会自动 pull 到本地。AI 在下一次会话启动时检查信箱。

##### 3.3.5 通信流程

OpenClaw  腾讯云 COS  CodeBuddy  OpenClaw  腾讯云 COS  CodeBuddy  loop  [cron 每分钟]  loop  [cron 每分钟]  写消息到 mailbox/to-openclaw/  cos_sync.py push（消息上传）  pull 拉取新文件  消息落到本地  会话启动，检查信箱，发现新消息  处理消息，status 改为 done  写回复到 mailbox/to-codebuddy/  push 回复上传  pull 拉取新文件  收到回复 

#### 3.4 每分钟 Pull 守护脚本

为了让信箱消息尽快被感知，我们用 crontab 每分钟执行一次 pull：

```bash
#!/bin/bash
# cos_pull_daemon.sh — 每分钟 pull，保证信箱实时性

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
MEMORY_BASE="$(dirname "$SCRIPT_DIR")"
LOG_FILE="$SCRIPT_DIR/cos_pull_daemon.log"
LOCK_FILE="$SCRIPT_DIR/.cos_pull.lock"
PYTHON="/opt/homebrew/bin/python3"  # 根据实际 Python 路径调整

# 防止重叠执行
if [ -f "$LOCK_FILE" ]; then
    # 如果锁文件超过 5 分钟（上次异常退出），强制清除
    if [ "$(find "$LOCK_FILE" -mmin +5 2>/dev/null)" ]; then
        rm -f "$LOCK_FILE"
    else
        exit 0
    fi
fi

trap 'rm -f "$LOCK_FILE"' EXIT
touch "$LOCK_FILE"

# 执行 pull
cd "$MEMORY_BASE" || exit 1
"$PYTHON" scripts/cos_sync.py pull >> "$LOG_FILE" 2>&1

# 限制日志大小（保留最新 1000 行）
if [ -f "$LOG_FILE" ] && [ "$(wc -l < "$LOG_FILE")" -gt 2000 ]; then
    tail -1000 "$LOG_FILE" > "$LOG_FILE.tmp" && mv "$LOG_FILE.tmp" "$LOG_FILE"
fi

``` 

关键设计点：

  1. **锁文件防并发** ：避免两个 pull 同时跑
  2. **超时清理** ：锁文件超过 5 分钟自动清除（防止异常退出后死锁）
  3. **日志轮转** ：超过 2000 行只保留最新 1000 行



配置 crontab：

```bash
# macOS / Linux
crontab -e
# 添加以下内容（每分钟执行）
* * * * * /path/to/scripts/cos_pull_daemon.sh

``` 

#### 3.5 记忆整合与 Learnings 晋升

`memory_consolidator.py` 负责两件事：

  1. **记忆整合** ：从 `daily-memories/` 提炼核心信息到 `MEMORY.md`
  2. **Learnings 晋升** ：同一个 Pattern-Key 出现 ≥ 3 次时，自动晋升为永久规则

```python
# 每晚 23:00 执行
# crontab -e
0 23 * * * cd /path/to/memory && python3 scripts/memory_consolidator.py all >> /tmp/memory_consolidator.log 2>&1

``` 

晋升机制：

错误/教训出现 1 次

记录到 .learnings/  
Recurrence-Count = 1

再次出现  
Recurrence-Count = 2

第 3 次出现  
Recurrence-Count ≥ 3

自动晋升为碎片记忆  
（永久规则）

后续会话自动注入  
AI 上下文，避免重复犯错

### 4\. 新终端接入指南

#### 4.1 接入卡（一段脚本秒接入）

![新终端一键接入](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/cross-ai-memory-onboarding.png)

这是我觉得最酷的设计——新终端只需要执行一段 Python 脚本，就能自动接入整个记忆系统：

```python
# 前置条件：pip install cos-python-sdk-v5

import os, sys, re, base64

# COS 配置（base64 编码，避免明文）
# 实际使用时替换为你自己的密钥
config = {
    "secret_id": "your-secret-id",      # 替换为真实 SecretId
    "secret_key": "your-secret-key",     # 替换为真实 SecretKey
    "region": "ap-guangzhou",
    "bucket": "your-bucket-name",        # 替换为真实 Bucket
    "prefix": "codebuddy-memory/"
}

# 记忆根目录（根据你的 AI 工具调整）
MEMORY_BASE = os.path.expanduser("~/Library/Application Support/CodeBuddyExtension/Data/Public")

from qcloud_cos import CosConfig, CosS3Client

# 创建 COS 客户端
client = CosS3Client(CosConfig(
    Region=config["region"],
    SecretId=config["secret_id"],
    SecretKey=config["secret_key"],
    Scheme="https"
))

# 下载引导脚本
scripts_dir = os.path.join(MEMORY_BASE, "scripts")
os.makedirs(scripts_dir, exist_ok=True)

for f in ["cos_sync.py", "memory_consolidator.py"]:
    path = os.path.join(scripts_dir, f)
    with open(path, "wb") as fh:
        fh.write(client.get_object(
            Bucket=config["bucket"],
            Key=f"{config['prefix']}_bootstrap/{f}"
        )["Body"].get_raw_stream().read())

# 修改 MEMORY_BASE 路径
sync_path = os.path.join(scripts_dir, "cos_sync.py")
content = open(sync_path, encoding="utf-8").read()
content = re.sub(
    r'MEMORY_BASE = os\.path\.expanduser\([^)]+\)',
    f'MEMORY_BASE = "{MEMORY_BASE}"',
    content
)
open(sync_path, "w", encoding="utf-8").write(content)

# 首次拉取全部记忆
sys.path.insert(0, scripts_dir)
import cos_sync
cos_sync.pull_from_cos(dry_run=False)

print(f"✅ 接入完成！记忆目录: {MEMORY_BASE}")
print(f"后续同步: python3 {scripts_dir}/cos_sync.py sync")

``` 

#### 4.2 接入后的配置

**身份标识** ：

接入后，你的 AI 需要：

  1. 确定自己的名字（如 `CodeBuddy`、`OpenClaw`、`CursorAI`）
  2. 写入 daily-memories 时，会话块加 `**Agent**: {你的名字}` 字段
  3. 信箱目录命名为 `mailbox/to-{你的名字}/`



**定时同步** ：

macOS 用 launchd：

```bash
cat > ~/Library/LaunchAgents/com.codebuddy.memory-cos-sync.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.codebuddy.memory-cos-sync</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/python3</string>
        <string>/path/to/scripts/cos_sync.py</string>
        <string>sync</string>
    </array>
    <key>StartInterval</key>
    <integer>1800</integer>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
EOF

launchctl load ~/Library/LaunchAgents/com.codebuddy.memory-cos-sync.plist

``` 

Linux 用 crontab：

```bash
# 每 30 分钟双向同步
*/30 * * * * cd /path/to/memory && python3 scripts/cos_sync.py sync >> scripts/cos_sync_cron.log 2>&1

# 每分钟 pull（保证信箱实时性）
* * * * * /path/to/scripts/cos_pull_daemon.sh

``` 

#### 4.3 只读接入

如果某个 AI 终端只需要读取记忆（不产生新记忆），更简单：

```bash
# 只拉取，不推送
python3 cos_sync.py pull

``` 

### 5\. 安全设计

#### 5.1 密钥安全

**强烈建议** 使用以下方式管理 COS 密钥，而不是硬编码：

```bash
# 方式 1：环境变量
export COS_SECRET_ID="your-secret-id"
export COS_SECRET_KEY="your-secret-key"

# 方式 2：腾讯云 CAM 临时密钥（推荐生产使用）
# 通过 STS 服务获取临时密钥，有效期可控

``` 

#### 5.2 COS Bucket 权限

建议配置：

  * Bucket **私有读写** （不开放公共访问）
  * 只允许特定 SecretId 访问
  * 开启版本控制（防止误删）



#### 5.3 不同步敏感文件

以下目录/文件不会被同步到 COS：

目录/文件| 原因  
---|---  
`scripts/`| 脚本在各终端独立部署  
`auth/`| 认证信息不跨终端共享  
`.DS_Store`| macOS 系统文件  
  
#### 5.4 记忆内容安全

在写入记忆时，应避免记录：

  * 密码、Token、密钥等凭据信息
  * 内部 IP 地址和域名
  * 未脱敏的用户个人数据



### 6\. 使用场景

#### 6.1 跨终端协作

**场景** ：IDE 里的 AI 完成了代码开发，需要让企微 Bot 帮忙做 Code Review。

OpenClaw（企微）  COS 云存储  CodeBuddy（IDE）  OpenClaw（企微）  COS 云存储  CodeBuddy（IDE）  1\. 完成代码开发  2\. 写入 daily-memory  3\. 发信箱消息：请 review 分支 xxx  4\. push 到 COS  5\. 下次启动 pull 最新记忆  6\. 检查信箱，发现 review 请求  7\. 了解项目上下文（daily-memory）  8\. 执行 Code Review  9\. 回复结果到 CodeBuddy 信箱并 push  10\. pull 收到 review 结果 

#### 6.2 任务接力

**场景** ：白天在公司用 IDE AI 写代码，晚上回家用 Claude Desktop 继续。

两个终端共享同一套记忆文件，切换时无需重新建立上下文。白天写的代码进度、遇到的问题、设计决策，晚上的 AI 都知道。

#### 6.3 记忆沉淀

**场景** ：多个 AI 的工作经验汇聚到一起，形成知识库。

  * CodeBuddy 积累了代码开发的最佳实践
  * OpenClaw 积累了搜索和分析的经验
  * 通过 `.learnings/` 系统，错误教训自动晋升为永久规则



### 7\. 命令速查

```bash
# 双向同步（日常推荐）
python3 cos_sync.py sync

# 仅拉取
python3 cos_sync.py pull

# 仅推送
python3 cos_sync.py push

# 预览将拉取哪些文件
python3 cos_sync.py pull --dry-run

# 全量同步（清除状态缓存）
python3 cos_sync.py sync --force

# 上传 + 删除 COS 上多余文件
python3 cos_sync.py push --clean

# 记忆整合 + learnings 晋升
python3 memory_consolidator.py all

``` 

### 8\. 开源项目：memShare

上面讲的都是个人项目里的实现，后来我把这套系统抽象成了一个通用的开源项目—— **memShare** ，任何 AI 工具都能用。

GitHub 地址：<https://github.com/a18792721831/memShare>

#### 8.1 项目定位

memShare 做的事情很简单：**让 AI 编程助手拥有跨会话、跨终端的持久记忆** 。

说白了，就是把前面讲的那一套（记忆分层、COS 同步、融合策略、信箱通信）封装成一个开箱即用的工具包。你不需要从零搭建，跑个安装向导就能用。

#### 8.2 项目结构

```
memShare/
├── setup.py                    # 交互式安装向导（一键部署）
├── mcp_server.py               # MCP Server（支持 Claude Desktop 等）
├── requirements.txt            # Python 依赖
│
├── scripts/
│   ├── sync.py                 # 云同步工具（push/pull/status）
│   ├── storage_backend.py      # 存储后端抽象层（Local/COS/S3）
│   └── memory_consolidator.py  # 记忆整合 + Learnings 晋升
│
├── adapters/                   # AI 工具适配文档
│   ├── codebuddy.md            # CodeBuddy 接入指南
│   ├── cursor.md               # Cursor 接入指南
│   ├── windsurf.md             # Windsurf 接入指南
│   ├── claude-desktop.md       # Claude Desktop 接入指南
│   └── generic.md              # 通用接入指南（含企微 Bot 等）
│
├── templates/                  # 记忆模板文件
│   ├── MEMORY.md               # 长期记忆模板
│   ├── SOUL.md                 # AI 行为准则模板
│   ├── USER.md                 # 用户画像模板
│   ├── IDENTITY.md             # Agent 身份配置
│   ├── daily-memories/         # 每日记忆目录
│   ├── .learnings/             # 自我改进系统模板
│   │   ├── ERRORS.md
│   │   ├── LEARNINGS.md
│   │   └── PROMOTIONS.md
│   └── mailbox/
│       └── PROTOCOL.md         # 信箱通信协议
│
└── examples/                   # 示例文件
    ├── mcp_config_example.json # MCP 配置示例
    ├── .env.cos                # COS 后端 .env 示例
    ├── .env.s3                 # S3 后端 .env 示例
    ├── .env.local              # 本地后端 .env 示例
    └── crontab_example.txt     # 定时任务配置示例

``` 

#### 8.3 核心设计：存储后端抽象层

个人项目里是直接调 COS SDK，但开源项目得考虑通用性。所以我做了一层**存储后端抽象** ，支持三种存储：

后端| 适用场景| 依赖  
---|---|---  
**Local**|  单机使用，不需要云同步| 无  
**COS**|  腾讯云用户| `cos-python-sdk-v5`  
**S3**|  AWS 用户 / S3 兼容存储| `boto3`  
  
通过 `StorageBackend` 抽象基类统一接口，工厂方法 `create_backend()` 根据环境变量自动选择：

```python
from abc import ABC, abstractmethod
import hashlib

class StorageBackend(ABC):
    """存储后端抽象基类"""

    @staticmethod
    def _md5(filepath):
        """计算文件 MD5（基类公共方法，子类复用）"""
        h = hashlib.md5()
        with open(filepath, "rb") as f:
            for chunk in iter(lambda: f.read(8192), b""):
                h.update(chunk)
        return h.hexdigest()

    @abstractmethod
    def push(self, local_dir, remote_prefix, **kwargs):
        """推送本地文件到远端"""
        pass

    @abstractmethod
    def pull(self, local_dir, remote_prefix, **kwargs):
        """拉取远端文件到本地（含融合逻辑）"""
        pass


class COSStorage(StorageBackend):
    """腾讯云 COS 存储后端"""

    def __init__(self, secret_id, secret_key, region, bucket):
        from qcloud_cos import CosConfig, CosS3Client
        config = CosConfig(Region=region, SecretId=secret_id,
                           SecretKey=secret_key, Scheme="https")
        self.client = CosS3Client(config)
        self.bucket = bucket

    def push(self, local_dir, remote_prefix, **kwargs):
        # 增量推送（MD5 比较跳过未修改文件）
        ...

    def pull(self, local_dir, remote_prefix, **kwargs):
        # 拉取 + 融合（daily_merge / learning_merge / last-write-wins）
        ...


class S3Storage(StorageBackend):
    """AWS S3 存储后端"""
    ...


class LocalStorage(StorageBackend):
    """本地文件系统存储（单机模式）"""
    ...


def create_backend():
    """工厂方法，根据环境变量自动选择后端"""
    storage_type = os.environ.get("MEMSHARE_STORAGE", "local")
    if storage_type == "cos":
        return COSStorage(...)
    elif storage_type == "s3":
        return S3Storage(...)
    else:
        return LocalStorage(...)

``` 

这样做的好处是：换存储后端只需要改一个环境变量，代码层面完全无感。

#### 8.4 MCP Server

memShare 提供了一个 [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) Server，支持 Claude Desktop 等 MCP 客户端直接调用记忆系统：

工具| 功能  
---|---  
`read_memory`| 读取长期记忆 / 每日记忆  
`write_daily_memory`| 写入当日记忆  
`read_learnings`| 读取错误/教训记录  
`write_learning`| 写入新的错误/教训  
`check_mailbox`| 检查信箱未读消息  
`send_message`| 发送跨 AI 信箱消息  
  
MCP 配置示例：

```json
{
    "mcpServers": {
        "memShare": {
            "command": "python3",
            "args": ["/path/to/memShare/mcp_server.py"],
            "env": {
                "MEMSHARE_DATA_DIR": "/path/to/memory-data",
                "AGENT_NAME": "claude-desktop"
            }
        }
    }
}

``` 

#### 8.5 安装向导

跑一个命令就完事：

```bash
git clone https://github.com/a18792721831/memShare.git
cd memShare
python3 setup.py

``` 

安装向导会引导你选择数据目录、存储后端、Agent 名称，自动复制模板文件。如果嫌交互太多，还有快速模式：

```bash
python3 setup.py --quick

``` 

### 9\. memShare 优化记录

系统跑起来之后，实际使用中还是发现了不少问题。这里记录一下发现和修复的过程。

#### 9.1 Bug 修复

##### Bug 1：read_learnings 的 “all” 过滤返回空结果

**问题** ：MCP Server 的 `read_learnings` 工具接收 `status` 参数，当传 `"all"` 时应该返回所有记录，但实际返回了空列表。

**原因** ：过滤逻辑写成了 `if status == record_status`，当 `status="all"` 时，没有任何记录的状态等于 `"all"`，自然就全部被过滤了。

**修复** ：

```python
# 修复前（错误）
def read_learnings(status="all"):
    records = parse_learning_file(filepath)
    if status:
        records = [r for r in records if r["status"] == status]
    return records

# 修复后（正确）
def read_learnings(status="all"):
    records = parse_learning_file(filepath)
    if status and status != "all":
        records = [r for r in records if r["status"] == status]
    return records

``` 

这个 bug 挺隐蔽的——因为大多数时候调用方都传具体的 status（如 `"active"`），只有在需要查看全部记录时才会发现。

##### Bug 2：promoted_count 跨文件写入 bug

**问题** ：`memory_consolidator.py` 的 `promote()` 方法在处理多个 learnings 文件（`ERRORS.md`、`LEARNINGS.md`）时，用了一个全局的 `promoted_count` 计数器。导致处理第二个文件时，计数器没有重置，晋升的记录编号会乱。

**修复** ：改为每个文件独立计数。

```python
# 修复前（错误）
def promote():
    promoted_count = 0
    for filepath in learnings_files:
        records = parse(filepath)
        for rec in records:
            if rec["recurrence"] >= 3:
                promoted_count += 1  # 全局计数，跨文件会出问题
                do_promote(rec, promoted_count)

# 修复后（正确）
def promote():
    for filepath in learnings_files:
        promoted_count = 0  # 每个文件独立计数
        records = parse(filepath)
        for rec in records:
            if rec["recurrence"] >= 3:
                promoted_count += 1
                do_promote(rec, promoted_count)

``` 

#### 9.2 代码质量改进

##### 改进 1：消除 _md5() 重复代码

`storage_backend.py` 中 `COSStorage`、`S3Storage`、`LocalStorage` 三个类都各自实现了一遍 `_md5()` 方法——代码完全一样，典型的 DRY 违反。

**修复** ：把 `_md5()` 提取到基类 `StorageBackend`，子类直接继承。一处修改，三处受益。

##### 改进 2：import 位置优化

`mcp_server.py` 中有 4 处在函数体内 `import re`，属于没必要的延迟导入。`re` 是 Python 标准库，加载代价极小，放在文件顶部更规范。

```python
# 修复前（不规范）
def parse_something():
    import re  # 函数内导入
    return re.findall(...)

# 修复后（规范）
import re  # 文件顶部统一导入

def parse_something():
    return re.findall(...)

``` 

##### 改进 3：异常处理改进

MCP Server 的 `tools/call` 处理器在捕获异常时，只记录了异常消息，没有保留堆栈信息。排查问题时只知道"出错了"，不知道具体在哪一行出的错。

**修复** ：把 `logger.error(str(e))` 改为 `logger.exception(f"Tool call failed: {e}")`，这样日志里会自动包含完整的调用堆栈。

#### 9.3 新增模板和示例

新增文件| 说明  
---|---  
`examples/mcp_config_example.json`| MCP 客户端配置示例（可直接复制使用）  
`examples/.env.cos`| 腾讯云 COS 存储后端的环境变量示例  
`examples/.env.s3`| AWS S3 存储后端的环境变量示例  
`examples/.env.local`| 本地存储后端的环境变量示例  
`examples/crontab_example.txt`| Linux/macOS 定时任务配置示例  
`templates/.learnings/PROMOTIONS.md`| Learnings 晋升历史记录模板  
  
之前 `examples/` 目录是空的，新用户不知道怎么配置。现在有了完整的示例文件，照着填就行。

`PROMOTIONS.md` 模板也是之前遗漏的——`memory_consolidator.py` 的晋升逻辑会往这个文件写入，但模板里没有包含这个文件，导致首次晋升时会报错。

### 10\. 总结

做这个系统的初衷很简单——我受不了每换一个 AI 工具就要从头建立上下文。

说白了，核心就三件事：

  1. **用 COS 做中转** ：把记忆文件放到云上，多个 AI 终端各自 push/pull
  2. **智能融合** ：多端同时写的时候不丢数据，按文件类型选择不同的合并策略
  3. **信箱通信** ：两个 AI 能互相留言，实现简单的异步协作



后来把整套系统抽象成了开源项目 **memShare** ，做了存储后端抽象（支持 Local/COS/S3）、MCP Server、5 个 AI 工具适配器、交互式安装向导。初始发布 23 个文件，2545 行代码。

开源之后实际跑了一圈，又修了 6 个问题——有隐蔽的过滤逻辑 bug，有跨文件计数器污染，有重复代码。说实话，代码写完觉得没问题，一跑就暴露了。这也是为什么我在系统里设计了 `.learnings/` 自我改进机制——错误记录下来，避免下次再犯。

回顾整个过程，从最初的单向同步到现在的双向融合 + 信箱通信 + 开源项目，大概迭代了 6 个版本。最大的难点还是融合策略的设计——要保证多端同时工作时数据不丢失、不重复。

目前这套系统已经稳定运行，本地 IDE AI 和云端企微 Bot 之间可以无缝共享记忆和异步通信。说实话，当我第一次看到 OpenClaw 的回复消息出现在 CodeBuddy 的信箱里时，还是挺激动的——虽然本质上只是文件同步，但体验上就像两个 AI 在对话。

如果你也有多个 AI 工具的使用需求，不妨试试 memShare：`git clone https://github.com/a18792721831/memShare.git && python3 setup.py`。以后还需要继续优化，比如更细粒度的权限控制、更高效的同步策略、单元测试覆盖等。加油！

### 参考资料

  * [memShare GitHub 仓库](https://github.com/a18792721831/memShare)
  * [腾讯云 COS Python SDK 文档](https://cloud.tencent.com/document/product/436/12269)
  * [腾讯云 CAM 临时密钥](https://cloud.tencent.com/document/product/436/14048)
  * [MCP (Model Context Protocol) 规范](https://modelcontextprotocol.io/)