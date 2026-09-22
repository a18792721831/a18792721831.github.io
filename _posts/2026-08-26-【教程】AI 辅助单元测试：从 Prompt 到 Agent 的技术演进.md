---
layout: post
title: "【教程】AI 辅助单元测试：从 Prompt 到 Agent 的技术演进"
date: 2026-08-26 09:12:09 +0800
categories: ["ai", "智能体", "agent", "SubAgent", "Skills"]
description: "【教程】AI 辅助单元测试：从 Prompt 到 Agent 的技术演进 本文以某 Go 后端项目的 gen-ut 和 run-ut Skills 为例，探讨 AI 辅助开发工具的技术演进历程。适合已使用过 AI 编程助手的进阶开发者阅读。 引言 当我们谈论 AI 辅助编程时，很多人的第一印象是\"对话式编程\"——向 AI 描述需求，AI 生成代码。 说实话，我最初也是这么想的。 然而，真正将 AI"
keywords: ["ai", "智能体", "agent", "SubAgent", "Skills"]
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/157221126
> - 发布时间：2026-08-26 09:12:09
> - 标签：#ai, #智能体, #agent, ##SubAgent, ##Skills

---

## 【教程】AI 辅助单元测试：从 Prompt 到 Agent 的技术演进

本文以某 Go 后端项目的 `gen-ut` 和 `run-ut` Skills 为例，探讨 AI 辅助开发工具的技术演进历程。适合已使用过 AI 编程助手的进阶开发者阅读。

![思维导图](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/mindmap/%E3%80%90%E6%95%99%E7%A8%8B%E3%80%91AI%20%E8%BE%85%E5%8A%A9%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95%EF%BC%9A%E4%BB%8E%20Prompt%20%E5%88%B0%20Agent%20%E7%9A%84%E6%8A%80%E6%9C%AF%E6%BC%94%E8%BF%9B_20260121190608.png)

* * *

### 引言

当我们谈论 AI 辅助编程时，很多人的第一印象是"对话式编程"——向 AI 描述需求，AI 生成代码。

说实话，我最初也是这么想的。

然而，真正将 AI 融入工程实践后才发现，需要解决的问题远比"对话"复杂得多。AI 不是万能的，它有很多局限性，而这些局限性恰恰推动了整个技术栈的演进。

本文将以**单元测试自动生成** 这一具体场景为切入点，介绍我在项目中开发的 `gen-ut` 和 `run-ut` 两个 Skills，并借此梳理 AI 辅助开发工具从萌芽到成熟的技术演进脉络。

* * *

### 1 技术演进全景图

在深入 Skills 之前，让我们先建立一个全局视角，理解 AI 辅助开发工具的演进历程：

技术演进

Prompt Engineering

Rules

SpectKit

MCP

Agent

Skills

SubAgent

AI Memory

每一层技术都在解决前一层遗留的问题，同时也引入了新的挑战。打个比方，就像打怪升级，解决了一个 Boss，下一关又有新的 Boss 在等着你。

下表总结了这一演进过程：

技术/概念| 解决的"旧"问题| 引入的"新"挑战  
---|---|---  
**Prompt Engineering**|  交互准确性：让 AI 理解人类模糊、跳跃的指令| 操作繁琐：优质 Prompt 需精心设计，固定流程需手动重复输入  
**Rules**|  固定 Prompt 自动化：无需每次手动输入项目规范| 上下文断裂：单次会话信息不继承，每次对话都需重新介绍背景  
**SpectKit**|  上下文断裂：快速告诉 AI 项目全貌，AI 自己决定是否读取| 信息过载与决策：如何让 AI 智能筛选海量背景信息  
**MCP**|  "孤岛"化能力：每个 AI 工具或数据源都有独特的连接方式| 生态构建与标准化：如何制定统一的"插座"标准  
**Agent**|  简单任务局限：AI 只能响应单一指令，无法自主规划多步骤任务| 决策可靠性与成本：复杂规划可能出错，长链条任务 Token 消耗大  
**Skills**|  流程标准化：复杂业务流程缺乏可复用的编排策略| 策略设计：如何设计通用且灵活的编排逻辑  
**SubAgent**|  系统臃肿与高成本：庞大复杂的 Agent 系统效率低| 架构协同：如何设计高效的子智能体间通信与协作机制  
**AI Memory**|  “金鱼脑”：AI 无法记住历史互动，每次对话都像第一次见面| 隐私、安全与偏见：记住什么？如何遗忘？记忆是否会被篡改？  
  
* * *

### 2 从 Prompt 到 Rules —— 解决重复劳动

#### 2.1 Prompt Engineering 的困境

最初使用 AI 辅助编程时，我们需要在每次对话中详细描述：

```
请帮我为这个 Go 函数生成单元测试。
要求：
1. 使用 gomonkey 进行 Mock
2. 使用 goconvey 进行断言
3. 每个测试函数独立，不要嵌套 Convey
4. Mock 后必须 defer func() { patches.Reset(); time.Sleep(time.Millisecond) }()
5. 必须包含 TestMain + goleak.IgnoreCurrent()
6. 覆盖率目标 100%
...

``` 

**问题** ：这些规范在项目中是固定的，但每次都要手动输入，既繁琐又容易遗漏。

说白了，就是重复劳动。作为程序员，最讨厌的就是重复劳动。

#### 2.2 Rules 的解决方案

**Rules** 机制允许我们将项目规范固化为配置文件，AI 会自动加载并遵守：

```yaml
# .codebuddy/rules/unit-test-rules.md
---
description: Go 项目单元测试规范
alwaysApply: true
---

## 核心原则
- 使用 gomonkey + goconvey
- 禁止嵌套 Convey
- Mock 后必须 Reset + Sleep: `defer func() { patches.Reset(); time.Sleep(time.Millisecond) }()`
- 每个包必须包含 TestMain + goleak.IgnoreCurrent()
- 目标 100% 覆盖率
...

``` 

**效果** ：固定的 Prompt 不再需要手动输入，AI 自动遵守项目规范。

**新问题** ：虽然规范固化了，但每次新会话，AI 仍然不知道项目的整体结构、已有的测试文件、函数签名等上下文信息。

* * *

### 3 SpectKit —— 解决上下文断裂

#### 3.1 单次会话的连续性问题

即使有了 Rules，AI 仍然面临"失忆"问题：

  * 不知道项目有哪些包、哪些文件
  * 不知道已有的测试覆盖了哪些函数
  * 不知道需要 Mock 的函数签名是什么



每次都需要人工告诉 AI 这些信息，效率低下。这什么玩意？不要着急，慢慢来。

#### 3.2 SpectKit 的解决方案

**SpectKit** 提供了一种机制：快速告诉 AI 项目全貌，并提供摘要，AI 自己决定是否需要读取详细内容。

SpectKit

项目结构摘要

src/project/controller/

user_service/ (15 files)

order_service/ (20 files)

函数签名索引

service.GetUser(...)

dao.QueryOrder(...)

AI 根据需要选择性读取

**效果** ：AI 可以快速了解项目全貌，按需获取详细信息。

**新问题** ：信息过载。当项目足够大时，即使是摘要也可能超出 AI 的处理能力。

* * *

### 4 MCP —— 物理世界感知

#### 4.1 工具孤岛问题

AI 需要与外部世界交互：读取文件、执行命令、查询数据库、调用 API。但每个工具都有独特的接口，难以复用。

#### 4.2 MCP (Model Context Protocol) 的解决方案

MCP 定义了一套统一的协议，让 AI 可以通过标准化的方式连接各种工具和数据源：

MCP Protocol

MCP 工具层

文件系统

数据库

Git

终端

HTTP API

...

AI

打个比方，MCP 就像是一个"万能插座"，不管你是什么设备，只要符合标准，就能接入。

**效果** ：AI 获得了"感知物理世界"的能力，可以读取代码、执行测试、分析覆盖率。

**新问题** ：有了工具，但 AI 仍然只能响应单一指令，无法自主规划复杂任务。

* * *

### 5 Agent —— 复杂场景的自主规划

#### 5.1 简单任务的局限

传统的 AI 交互是"一问一答"模式：

```
用户: 帮我生成单元测试
AI: 好的，这是测试代码...

用户: 运行一下测试
AI: 测试失败了...

用户: 修复错误
AI: 已修复...

用户: 再运行一次
AI: 还是失败...

``` 

这种模式需要人工介入每一步，效率低下。说实话，这样搞下去，还不如自己写。

#### 5.2 Agent 的解决方案

**Agent** 可以自主规划并执行多步骤任务：

```
用户: 帮我为 user_service 包生成单元测试，目标 100% 覆盖率

Agent 自主规划:
1. 分析包结构，列出所有源文件
2. 提取需要测试的函数
3. 搜索 Mock 函数签名
4. 生成测试代码
5. 运行测试，检查覆盖率
6. 如果覆盖率不足，补充测试
7. 重复直到达到目标
8. Git 提交

``` 

**效果** ：AI 可以自主完成复杂的多步骤任务。

**新问题** ：

  * **决策可靠性** ：复杂规划可能出错
  * **成本问题** ：长链条任务消耗大量 Token



* * *

### 6 分层智能体架构 —— 从工具调用到战略编排

#### 6.1 系统臃肿问题

当 Agent 需要处理复杂任务时，所有的上下文（源代码、测试代码、执行结果、覆盖率报告）都会累积在同一个会话中，导致：

  * Token 消耗急剧增加
  * 上下文窗口溢出
  * 响应速度变慢



#### 6.2 分层协作架构

我们可以将整个系统看作一个**分层协作的智慧组织** ：

基础设施层

战术执行层

战术编排层

战略决策层

分解目标  
调用策略

调度执行  
指定模型

调度执行  
指定模型

调度执行  
指定模型

主 Agent  
首席执行官

Skills  
部门总监/指挥官

SubAgent 1  
特种部队

SubAgent 2  
特种部队

SubAgent 3  
特种部队

MCP  
军械库/万能插座

层级| 核心组件| 职责与特点| 比喻  
---|---|---|---  
**战略决策层**|  主 Agent| 理解用户终极目标，进行顶层的任务分解与全局规划| 首席执行官  
**战术编排层**|  Skills| 封装解决某类复杂问题的**标准化策略与流程** 。知道"在什么情况下，为了达成什么子目标，应该调度谁以及如何调度"| 部门总监/指挥官  
**战术执行层**|  SubAgent| 负责执行一个具体的、定义明确的专业任务。可根据任务需求灵活选择最适合的模型| 特种部队/专家  
**基础设施层**|  MCP| 为所有上层提供标准化的"装备"和"补给"（工具、数据）| 军械库/万能插座  
  
#### 6.3 核心洞察：Skills 与 SubAgent 的本质区别

**Skills 是"指挥官"** ：它不直接做事，但它知道完整的"作战手册"。

Step 1: 调用

Step 2: 调用

Step 3: 调用

Step 4: 调用

gen-ut Skill (指挥官)

策略 1: 先获取基线覆盖率

策略 2: 分析包结构后生成测试

策略 3: 验证→运行→补充循环

策略 4: 达标后 Git 提交

run-ut SubAgent  
获取基线

write-ut SubAgent  
生成测试

verify-ut SubAgent  
验证修复

analyze-coverage SubAgent  
补充测试

**SubAgent 是"特种部队"** ：每个 SubAgent 都是一个功能完备的小型 Agent，其优势在于：

特性| 说明  
---|---  
**模型灵活性**|  可根据任务复杂性动态选择轻量模型或重量级模型  
**专业优化**|  针对特定任务拥有专门的提示词、工具链和验证逻辑  
**独立上下文**|  执行完毕后释放上下文，不污染主会话  
**可复用性**|  可被多个不同的 Skills 调用  
  
#### 6.4 价值总结

MCP  
能力接入标准化  
'有什么装备可用'

SubAgent  
能力执行专业化  
'如何用最佳方式完成动作'

Skills  
能力组合与流程标准化  
'如何编排一系列动作'

Agent  
自主决策实体  
'运用所有能力'

> **Skills 是复杂、可复用业务流程的载体，是连接宏观目标与微观执行的智能中间件。它站在 SubAgent 之上，负责指挥和协同，是让 AI 系统从"能完成单个任务"到"能运营一整个业务流程"的关键跃升。**

* * *

### 7 gen-ut 与 run-ut 实战

现在，让我们深入了解项目中的两个核心组件，它们完美诠释了上述分层架构。

#### 7.1 run-ut：战术执行层的 SubAgent

**定位** ：专业化的测试执行单元，可被多个 Skill 调用。

**功能** ：运行指定代码相关的单元测试，返回测试结果和覆盖率。

**使用方式** ：

```bash
# 方式 1：包名简写
run-ut user_service

# 方式 2：目录路径
run-ut ./src/project/controller/user_service
run-ut ./src/project/service/order
run-ut ./src/project/dao/user

# 方式 3：文件+行号
run-ut path/to/file.go:12-15,20

# 方式 4：文件+函数名
run-ut path/to/file.go:FuncName1,FuncName2

``` 

**输出示例** ：

```markdown
## 单元测试运行结果

### 测试目标
- **包路径**: ./src/project/controller/user_service

### 测试结果
- **状态**: ✅ PASS
- **覆盖率**: 100.0%
- **通过**: 56 个测试
- **失败**: 0 个测试

``` 

**技术特点** ：

  * 自动设置必要的环境变量（GOROOT、UT_TEST_MODE 等）
  * 使用 `-gcflags="all=-l"` 禁用内联，确保 gomonkey 正常工作
  * 使用 `goleak.IgnoreCurrent()` 避免 vendor 库 goroutine 干扰
  * **可被多个 Skill 复用** ：gen-ut、analyze-coverage 等都会调用它



#### 7.2 gen-ut：战术编排层的 Skill

**定位** ：封装了"生成单元测试"这一复杂业务流程的**标准化策略** 。

**核心价值** ：它不直接生成代码，而是知道完整的"作战手册"——在什么情况下，调用哪个 SubAgent，以什么顺序，如何处理异常。

**使用方式** ：

```bash
# 包名简写
gen-ut user_service

# 完整路径
gen-ut ./src/project/controller/order_service
gen-ut ./src/project/service/payment

``` 

**内嵌策略（作战手册）** ：

SubAgent 调度

gen-ut Skill 策略编排

调用

调用

调用

调用

覆盖率 = 100%

覆盖率 < 100%

调用

Step 0: 包存在性检查

Step 1: 获取覆盖率基线

Step 2: 分析 + 生成测试

Step 3: 验证 + 修复

Step 4: 运行测试 + 检查覆盖率

Step 5: Git 提交

Step 6: 输出总结报告

补充测试策略

run-ut  
(获取基线)

write-ut  
(生成测试)

verify-ut  
(验证修复)

run-ut  
(运行测试)

analyze-coverage  
(补充测试)

**输出示例** ：

```markdown
## 单元测试生成报告

- **包名**: ./src/project/controller/order_service
- **覆盖率**: 9.3% → 100%
- **生成文件**: 6 个测试文件
- **Commit**: abc1234

``` 

**技术特点** ：

  * **主 AI 全程不读取源文件** ：最大化节省 Token
  * **策略驱动** ：按预定义流程调度 SubAgent
  * **迭代优化** ：自动循环直到达到 100% 覆盖率
  * **异常处理** ：每个步骤都有明确的失败处理策略



#### 7.3 完整协作流程

以下展示了用户发起请求后，整个分层架构如何协同工作：

MCP  (基础设施层)  verify-ut  (SubAgent)  write-ut  (SubAgent)  run-ut  (SubAgent)  gen-ut Skill  (战术编排层)  主 Agent  (战略决策层)  用户  MCP  (基础设施层)  verify-ut  (SubAgent)  write-ut  (SubAgent)  run-ut  (SubAgent)  gen-ut Skill  (战术编排层)  主 Agent  (战略决策层)  用户  Step 1: 获取基线  Step 2: 生成测试  Step 3: 验证修复  Step 4: 运行测试  Step 5: Git 提交  gen-ut order_service  调用 gen-ut 策略  运行测试，获取覆盖率  执行 go test 命令  测试结果  覆盖率 9.3%  分析包结构，生成测试  读取源文件  搜索函数签名  写入测试文件  生成 6 个测试文件  验证并修复测试代码  编译检查  验证通过  运行测试，检查覆盖率  覆盖率 100%  git add && git commit  任务完成  报告结果 

#### 7.4 SubAgent 调度代码示例

以下是 `gen-ut` Skill 调用 SubAgent 的实际模式：

```python
# Step 1: 获取基线覆盖率
# 调用 run-ut SubAgent，它会自行选择合适的执行策略
Task(
    subagent_name="run-ut",
    description="获取基线覆盖率",
    prompt="运行 ./src/project/controller/order_service 测试，返回覆盖率基线"
)
# → 返回: 覆盖率 9.3%

# Step 2: 生成测试代码
# 调用 write-ut SubAgent，它拥有专门的测试生成策略
Task(
    subagent_name="write-ut",
    description="生成单元测试",
    prompt="""为包 ./src/project/controller/order_service 生成完整单元测试。
    
    【任务】
    1. 自行分析包内所有源文件
    2. 自行搜索需要的函数签名
    3. 生成所有测试文件
    4. 确保包含 TestMain + goleak.IgnoreCurrent()
    """
)
# → 返回: 生成 6 个测试文件

# Step 3: 验证修复
# 调用 verify-ut SubAgent，它专注于语法验证和错误修复
Task(
    subagent_name="verify-ut",
    description="验证修复测试",
    prompt="验证并修复 ./src/project/controller/order_service 的测试代码"
)
# → 返回: 验证通过

# Step 4: 运行测试
# 再次调用 run-ut SubAgent
Task(
    subagent_name="run-ut",
    description="运行测试",
    prompt="运行 ./src/project/controller/order_service 测试"
)
# → 返回: 覆盖率 100%

``` 

* * *

### 8 AI Memory —— 未来展望

#### 8.1 当前的局限

即使有了 Skills 和 SubAgent 的分层架构，AI 仍然面临一个根本性问题：**“金鱼脑”** 。

每次新会话，AI 都不记得：

  * 之前生成过哪些测试
  * 哪些 Mock 签名验证过是正确的
  * 哪些代码块是"不可覆盖"的
  * 用户的偏好和习惯



说实话，这个问题让我挺头疼的。每次都要重新"教育" AI，感觉像是在带一个新同事。

#### 8.2 AI Memory 的愿景

**AI Memory** 是 AI 必备的一个能力，它让 AI 能够：

  * **记住历史互动** ：知道之前做过什么，避免重复劳动
  * **积累项目知识** ：记住函数签名、Mock 模式、常见错误
  * **学习用户偏好** ：适应不同开发者的习惯



#### 8.3 Memory 与分层架构的融合

分层架构

读取/写入

策略优化

经验复用

AI Memory (长期记忆)

项目知识库

用户偏好

历史经验

主 Agent

Skills

SubAgents

MCP

#### 8.4 新的挑战

然而，AI Memory 也带来了新的问题：

挑战| 说明  
---|---  
**隐私**|  记住什么？哪些信息不应该被记住？  
**安全**|  记忆是否会被篡改？如何防止恶意注入？  
**遗忘**|  如何处理过时的记忆？如何主动遗忘？  
**偏见**|  记忆是否会导致 AI 产生偏见？  
  
这些问题，正是 AI 辅助开发工具下一阶段需要解决的核心挑战。

* * *

### 9 总结

本文从 `gen-ut` 和 `run-ut` 两个组件出发，梳理了 AI 辅助开发工具的技术演进历程：

演进路径

Prompt Engineering  
交互准确性

Rules  
固定Prompt自动化

SpectKit  
上下文连续性

MCP  
工具标准化

Agent  
复杂任务自主规划

Skills  
流程标准化编排

SubAgent  
专业化执行

AI Memory  
持久记忆

#### 核心洞察

组件| 核心价值| 解决的问题  
---|---|---  
**MCP**|  能力接入标准化| “有什么装备可用”  
**SubAgent**|  能力执行专业化与灵活化| “如何用最佳方式完成一个具体动作”  
**Skills**|  能力组合与流程标准化| “为了达成一个复杂目标，应该如何编排一系列动作”  
**Agent**|  自主决策实体| 运用所有这些能力  
  
每一层技术都在解决前一层的问题，同时引入新的挑战。这正是技术演进的本质——**没有银弹，只有不断迭代** 。

`gen-ut` 作为 **Skill** ，展示了如何通过策略编排，将复杂的单元测试生成流程标准化；`run-ut` 作为 **SubAgent** ，展示了如何通过专业化执行，高效完成具体任务。它们共同诠释了**分层智能体架构** 的设计理念：

> **Skills 是复杂、可复用业务流程的载体，是连接宏观目标与微观执行的智能中间件。它站在 SubAgent 之上，负责指挥和协同，是让 AI 系统从"能完成单个任务"到"能运营一整个业务流程"的关键跃升。**

* * *

阅读这篇文章，对于我来说，也是一次对 AI 辅助开发技术的梳理。从最初的 Prompt Engineering 到现在的分层智能体架构，每一步都在解决实际问题。

不过说实话，我自己也还在摸索中。AI 辅助开发这个领域发展太快，今天的最佳实践，明天可能就过时了。但有一点是确定的：**理解技术演进的脉络，比掌握具体工具更重要** 。

以后还需要继续努力。加油！