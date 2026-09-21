---
layout: post
title: "【教程】AI 编程助手的 SubAgent 机制详解：让 AI 学会“分工协作“"
date: 2026-09-03 15:41:54 +0800
categories: ["ai", "智能体", "agent", "Agent", "智能"]
description: "【教程】AI 编程助手的 SubAgent 机制详解：让 AI 学会\"分工协作\" 本文深入解析 AI 编程助手中的 SubAgent（子代理）机制，包括什么是 SubAgent、为什么需要它、核心工作原理、主流工具的实现对比，以及如何自定义 SubAgent 提升开发效率。适合已使用过 Claude Code、Cursor 等 AI 编程助手的进阶开发者阅读。 1. 什么是 SubAgent 1."
keywords: ["ai", "智能体", "agent", "Agent", "智能"]
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/157220828
> - 发布时间：2026-09-03 15:41:54
> - 标签：#ai, #智能体, #agent, ##Agent, ##智能

---

## 【教程】AI 编程助手的 SubAgent 机制详解：让 AI 学会"分工协作"

本文深入解析 AI 编程助手中的 SubAgent（子代理）机制，包括什么是 SubAgent、为什么需要它、核心工作原理、主流工具的实现对比，以及如何自定义 SubAgent 提升开发效率。适合已使用过 Claude Code、Cursor 等 AI 编程助手的进阶开发者阅读。

![思维导图](https://i-blog.csdnimg.cn/img_convert/b00ce133cb934c48dbd50eed99c94e8e.png)

### 1\. 什么是 SubAgent

#### 1.1 从一个场景说起

你有没有遇到过这样的情况：

  * 让 AI 分析一个大型项目的代码结构，它读了半天文件，最后告诉你"上下文太长了，我可能遗漏了一些信息"
  * 让 AI 找一个功能的实现位置，它一个个文件搜索，效率低得让人抓狂
  * 让 AI 做代码审查，它把整个项目的代码都塞进上下文，结果重点信息反而被淹没了



说白了，这些问题的根源在于：**传统的 AI 编程助手是"单打独斗"的模式** 。

打个比方，这就像让一个人既当侦察兵又当指挥官——他需要亲自跑遍整个战场收集情报，然后再回来制定作战计划。效率能高吗？

而 SubAgent 的出现，就是为了解决这个问题。它让 AI 学会了"分工协作"：派出专门的"侦察兵"去探索代码库，收集信息后汇总成精简的报告，再交给"指挥官"做决策。

#### 1.2 SubAgent 的定义

SubAgent（子代理）是主 Agent 可以调用的专门化子代理。

还是用公司的类比来解释：

  * **主 Agent** 就像公司老板，负责理解用户需求、规划任务、整合结果
  * **SubAgent** 就像专业团队，每个团队专注于特定领域的工作



👤 用户  
帮我分析一下这个项目的认证模块是怎么实现的

🧠 主 Agent（老板）  
这个任务需要广泛搜索代码库  
我派 code-explorer 去探索

🔍 SubAgent（侦察兵）

搜索 auth、login、token 相关文件

分析代码结构和调用关系

汇总成精简报告

🧠 主 Agent（老板）  
根据报告，认证模块的实现是这样的...

💬 回复用户

SubAgent 的核心特点：

特点| 说明  
---|---  
**专注特定任务**|  每个 SubAgent 只做一件事，比如代码探索、代码审查  
**独立上下文空间**|  SubAgent 的工作过程不会污染主 Agent 的上下文  
**返回精简结果**|  只把最终结论返回给主 Agent，而不是原始数据  
  
#### 1.3 SubAgent vs 普通工具调用

你可能会问：这和普通的工具调用有什么区别？不都是 AI 调用一个功能吗？

区别大了。

特性| 普通工具调用| SubAgent  
---|---|---  
**复杂度**|  单一操作（读一个文件、搜索一个关键词）| 多步骤复杂任务（探索整个代码库）  
**上下文**|  结果直接进入主 Agent 上下文| 独立上下文，只返回精简摘要  
**返回结果**|  原始数据（文件内容、搜索结果）| 经过分析的精简报告  
**适用场景**|  读取已知路径的文件、简单搜索| 代码探索、复杂分析、多文件关联  
**Token 消耗**|  结果越多，消耗越大| 无论探索多少文件，只返回摘要  
  
举个例子：

**普通工具调用** ：

👤 用户  
找一下项目中所有的 API 接口

🤖 AI  
调用 search_content

📄 返回 50 个匹配结果

😵 全部进入上下文  
上下文膨胀，可能遗漏重点

**SubAgent 方式** ：

👤 用户  
找一下项目中所有的 API 接口

🤖 AI  
启动 code-explorer

🔍 SubAgent  
独立搜索、分析、汇总

📋 精简摘要  
3 个路由文件，25 个 API 接口

😊 主 Agent 上下文保持清爽

### 2\. 为什么需要 SubAgent

#### 2.1 上下文窗口的困境

现在的 LLM 上下文窗口已经很大了，Claude 有 200K，GPT-4 有 128K。但是，大型项目的代码量动辄几十万行，全部塞进上下文是不现实的。

而且，上下文越长，问题越多：

问题| 影响  
---|---  
**注意力分散**|  重要信息被淹没在大量无关内容中  
**成本增加**|  Token 消耗直接影响 API 调用成本  
**响应变慢**|  处理长上下文需要更多时间  
**准确度下降**|  研究表明，LLM 在长上下文中容易"遗忘"中间内容  
  
说实话，我自己在使用 AI 编程助手时就遇到过这个问题。让它分析一个稍微大点的项目，它读了一堆文件后，反而把之前讨论的重点给忘了。

#### 2.2 专业化分工的优势

SubAgent 的核心思想是"专人专事"：

  * **code-explorer** ：专门负责代码探索，熟悉各种搜索策略
  * **code-reviewer** ：专门负责代码审查，知道常见的代码问题
  * **test-generator** ：专门负责生成测试，了解测试最佳实践



每个 SubAgent 都有自己的"专业知识"（系统提示词），能更高效地完成特定任务。

打个比方：你不会让公司的前端工程师去做数据库优化，也不会让 DBA 去写 CSS。AI 也是一样，专业化分工能大幅提升效率。

#### 2.3 减少 Token 消耗

这是 SubAgent 最直接的好处。

**传统方式** ：

🔍 搜索 50 个文件

📚 50 个文件内容进入上下文

💸 消耗大量 Token

**SubAgent 方式** ：

🔍 搜索 50 个文件

🔒 SubAgent 内部处理

📋 只返回 500 字摘要

✅ Token 消耗大幅降低

根据我的实际使用经验，使用 SubAgent 后，Token 消耗可以降低 60%-80%。这对于需要频繁使用 AI 编程助手的开发者来说，是实打实的成本节省。

### 3\. SubAgent 的工作原理

#### 3.1 核心架构

SubAgent 的架构其实不复杂，核心就是"主从分离"：

结果返回层

SubAgent 层（侦察兵）- 独立上下文

主 Agent 层（指挥官）

用户请求层

用户请求

理解意图

规划任务

整合结果

Task 工具

search_file

search_content

read_file

list_files

精简结果摘要

返回主 Agent

#### 3.2 调用流程详解

一次完整的 SubAgent 调用流程：

**第一步：主 Agent 识别任务类型**

主 Agent 收到用户请求后，会判断这个任务是否需要广泛搜索代码库。如果是，就决定调用 SubAgent。

```
用户："帮我找一下项目中用户认证的实现"

主 Agent 思考：
- 这个任务需要搜索多个文件
- 我不知道认证代码在哪里
- 需要广泛探索 → 调用 code-explorer SubAgent

``` 

**第二步：通过 Task 工具启动 SubAgent**

主 Agent 调用 Task 工具，传入任务描述：

```json
{
  "subagent_name": "code-explorer",
  "description": "查找认证实现",
  "prompt": "请在项目中搜索用户认证相关的实现，包括登录、注册、Token 验证等功能。分析代码结构并返回摘要。"
}

``` 

**第三步：SubAgent 在独立上下文中执行**

SubAgent 启动后，拥有自己独立的上下文空间。它可以：

  * 调用 `search_content` 搜索 “auth”、“login”、“token” 等关键词
  * 调用 `read_file` 读取相关文件
  * 调用 `list_files` 探索目录结构



这些操作的结果都在 SubAgent 的上下文中，不会进入主 Agent。

**第四步：SubAgent 返回结构化结果**

SubAgent 完成探索后，会生成一份精简的报告：

```markdown
## 认证模块分析结果

### 文件位置
- `src/auth/` 目录下包含所有认证相关代码

### 核心文件
1. `src/auth/login.ts` - 登录逻辑
2. `src/auth/register.ts` - 注册逻辑
3. `src/auth/middleware.ts` - Token 验证中间件

### 技术方案
- 使用 JWT 进行身份验证
- Token 有效期 7 天
- 支持 refresh token 刷新

### 调用关系
login.ts → generateToken() → middleware.ts 验证

``` 

**第五步：主 Agent 基于结果继续工作**

主 Agent 收到这份精简报告后，可以：

  * 直接回复用户
  * 基于报告中的文件路径，进一步读取具体代码
  * 结合用户需求，给出更具体的建议



#### 3.3 上下文隔离机制

为什么上下文隔离这么重要？

**不隔离的问题** ：

😵 主 Agent 上下文（膨胀）

💬 用户对话历史  
（重要）

📝 之前讨论的代码  
（重要）

📚 SubAgent 搜索的 50 个文件  
（大量无关信息）

❓ 用户新的问题  
（重要）

❌ 重要信息被淹没  
AI 可能'忘记'之前的讨论

**隔离后的效果** ：

😊 主 Agent 上下文（清爽）

💬 用户对话历史  
（重要）

📝 之前讨论的代码  
（重要）

📋 SubAgent 返回的 500 字摘要  
（精简）

❓ 用户新的问题  
（重要）

✅ 上下文清爽  
AI 能保持对话连贯性

信息传递方式：

方向| 传递内容  
---|---  
主 Agent → SubAgent| 任务描述（prompt）  
SubAgent → 主 Agent| 精简的结果摘要  
  
SubAgent 内部的搜索过程、读取的文件内容、中间思考过程，都不会传递给主 Agent。这就是"隔离"的含义。

### 4\. 主流工具的 SubAgent 实现

#### 4.1 Claude Code 的 SubAgent

Claude Code 内置了 `code-explorer` 子代理。

**触发条件** ：

当任务需要广泛搜索代码库时，Claude Code 会自动调用 code-explorer。你也可以在对话中明确要求：

```
用户：使用 code-explorer 帮我分析项目结构

``` 

**内置工具** ：

code-explorer 可以使用以下工具：

工具| 功能  
---|---  
`search_file`| 按文件名模式搜索（支持通配符）  
`search_content`| 按内容搜索（支持正则表达式）  
`read_file`| 读取文件内容  
`list_files`| 列出目录结构  
  
**使用示例** ：

```
用户：帮我找一下项目中所有的数据库操作

Claude Code 响应：
"我来启动 code-explorer 子代理帮你探索..."

[SubAgent 执行中]
- 搜索 "database"、"sql"、"query" 关键词
- 分析 ORM 配置文件
- 识别数据模型定义

[返回结果]
"根据探索结果，项目使用 Prisma ORM，数据库操作主要在以下位置：
1. prisma/schema.prisma - 数据模型定义
2. src/services/*.ts - 业务逻辑中的数据库调用
3. src/repositories/*.ts - 数据访问层封装"

``` 

**自定义 SubAgent** ：

Claude Code 支持自定义 SubAgent。你可以在项目中定义自己的子代理，后面会详细介绍。

#### 4.2 Cursor 的 Agent 模式

Cursor 的 Agent 模式和 Claude Code 的 SubAgent 有些不同。

**工作方式** ：

Cursor 的 Agent 模式更像是一个"增强版"的 AI 助手，它可以自动调用多个工具完成任务，但不是严格意义上的"子代理"。

Agent模式

❓ 用户提问

🔍 自动搜索文件

📖 读取代码

⚡ 执行命令

💬 返回结果

普通模式

❓ 用户提问

💬 AI 回答问题

**与 Claude Code SubAgent 的区别** ：

特性| Claude Code SubAgent| Cursor Agent 模式  
---|---|---  
上下文隔离| 完全隔离| 部分隔离  
触发方式| 自动/手动| 手动切换模式  
自定义能力| 支持自定义 SubAgent| 不支持  
多 Agent 协作| 支持| 有限支持  
  
**使用建议** ：

如果你的任务比较简单，Cursor 的 Agent 模式足够用了。但如果需要复杂的代码探索或多 Agent 协作，Claude Code 的 SubAgent 更强大。

#### 4.3 GitHub Copilot Workspace

GitHub Copilot Workspace 采用了多 Agent 协作的架构。

**Agent 类型** ：

Agent| 职责  
---|---  
**规划 Agent**|  分析需求，制定实现计划  
**编码 Agent**|  根据计划编写代码  
**审查 Agent**|  检查代码质量和安全性  
**测试 Agent**|  生成测试用例  
  
**工作流程** ：

👤 用户需求

📋 规划 Agent  
制定计划

💻 编码 Agent  
实现代码

🔍 审查 Agent  
检查代码

🧪 测试 Agent  
生成测试

✅ 返回完整方案

**特点** ：

  * 自动化程度高，用户只需描述需求
  * 多 Agent 协作，各司其职
  * 但自定义能力有限，不能添加自己的 Agent



#### 4.4 对比总结

工具| SubAgent 名称| 触发方式| 独立上下文| 可自定义| 多 Agent 协作  
---|---|---|---|---|---  
Claude Code| code-explorer| 自动/手动| ✅| ✅| ✅  
Cursor| Agent 模式| 手动切换| 部分| ❌| 有限  
Copilot Workspace| 内置多 Agent| 自动| ✅| ❌| ✅  
Aider| 无内置| -| -| ✅| ❌  
  
**选择建议** ：

场景| 推荐工具  
---|---  
需要自定义 SubAgent| Claude Code  
简单的代码辅助| Cursor  
完整的需求到代码流程| Copilot Workspace  
开源/本地部署| Aider  
  
### 5\. 自定义 SubAgent 实践

#### 5.1 SubAgent 定义格式

在 Claude Code 中，SubAgent 的定义格式如下：

```yaml
# 在 CLAUDE.md 或 AGENTS.md 中定义
name: code-reviewer
path: builtin  # 或自定义路径
description: 代码审查子代理，用于检查代码质量、发现潜在问题

``` 

如果是自定义 SubAgent，需要创建一个 Markdown 文件来定义其行为：

```markdown
# Code Reviewer SubAgent

## 职责
对代码进行审查，检查以下方面：
- 代码风格是否符合规范
- 是否存在潜在的 bug
- 性能是否有优化空间
- 安全性问题

## 可用工具
- read_file: 读取代码文件
- search_content: 搜索代码模式

## 输出格式
返回结构化的审查报告，包括：
1. 问题列表（严重程度、位置、描述）
2. 改进建议
3. 总体评价

``` 

#### 5.2 创建自定义 SubAgent

以创建一个 “code-reviewer” SubAgent 为例，完整步骤如下：

**第一步：定义 SubAgent 的职责**

首先明确这个 SubAgent 要做什么：

```markdown
## Code Reviewer 职责

1. 检查代码风格
   - 命名规范
   - 缩进和格式
   - 注释完整性

2. 发现潜在问题
   - 空指针/未定义检查
   - 资源泄漏
   - 异常处理

3. 性能分析
   - N+1 查询
   - 不必要的循环
   - 内存使用

4. 安全检查
   - SQL 注入
   - XSS 漏洞
   - 敏感信息泄露

``` 

**第二步：配置可用工具**

SubAgent 需要哪些工具来完成任务：

```yaml
tools:
  - read_file      # 读取代码文件
  - search_content # 搜索代码模式
  - list_files     # 列出目录结构

``` 

**第三步：编写系统提示词**

这是 SubAgent 的"大脑"，决定了它的行为方式：

```markdown
# Code Reviewer System Prompt

你是一个专业的代码审查员。你的任务是对代码进行全面审查，找出问题并提供改进建议。

## 审查流程

1. 首先了解代码的整体结构
2. 逐文件检查代码质量
3. 分析代码间的依赖关系
4. 汇总问题并分类

## 问题分类

- **Critical**: 必须修复，可能导致系统崩溃或安全漏洞
- **Major**: 应该修复，影响代码质量或性能
- **Minor**: 建议修复，代码风格或可读性问题
- **Info**: 仅供参考，最佳实践建议

## 输出格式

    ```markdown
    ## 代码审查报告

    ### 概述
    [简要描述审查范围和总体评价]

    ### 问题列表

    #### Critical
    - [ ] 文件:行号 - 问题描述

    #### Major
    - [ ] 文件:行号 - 问题描述

    #### Minor
    - [ ] 文件:行号 - 问题描述

    ### 改进建议
    [具体的改进建议]

    ### 总结
    [总体评价和下一步建议]
    \````

``` 

**第四步：集成到主 Agent**

在 `CLAUDE.md` 或 `AGENTS.md` 中注册这个 SubAgent：

```markdown
## 可用 SubAgent

### code-reviewer
- **用途**：代码审查
- **触发**：当用户要求审查代码时使用
- **配置**：`.claude/subagents/code-reviewer.md`

``` 

#### 5.3 SubAgent 提示词设计

好的提示词是 SubAgent 成功的关键。以下是设计原则：

**1\. 明确任务边界**

```markdown
# 好的写法
你只负责代码审查，不要修改代码，不要执行代码。

# 不好的写法
帮我检查一下代码。

``` 

**2\. 定义输出格式**

```markdown
# 好的写法
## 输出格式
返回 JSON 格式的审查结果：
{
  "summary": "总体评价",
  "issues": [
    {"severity": "critical", "file": "xxx.ts", "line": 10, "message": "xxx"}
  ],
  "suggestions": ["建议1", "建议2"]
}

# 不好的写法
返回审查结果。

``` 

**3\. 设置质量标准**

```markdown
# 好的写法
## 审查标准
- 必须检查所有公开方法的参数验证
- 必须检查异常处理是否完整
- 必须检查是否有硬编码的敏感信息

# 不好的写法
仔细检查代码。

``` 

#### 5.4 实战案例：代码审查 SubAgent

下面是一个完整的代码审查 SubAgent 实现。

**配置文件** （`.claude/subagents/code-reviewer.md`）：

```markdown
# Code Reviewer SubAgent

## 角色定义
你是一个资深的代码审查员，拥有 10 年以上的软件开发经验。你擅长发现代码中的潜在问题，并能给出具体的改进建议。

## 审查维度

### 1. 代码正确性
- 逻辑错误
- 边界条件处理
- 空值检查
- 类型安全

### 2. 代码质量
- 命名规范（变量、函数、类）
- 代码重复
- 函数长度（建议不超过 50 行）
- 圈复杂度（建议不超过 10）

### 3. 性能问题
- 不必要的循环
- N+1 查询
- 内存泄漏风险
- 大对象拷贝

### 4. 安全问题
- SQL 注入
- XSS 漏洞
- 敏感信息泄露
- 不安全的依赖

### 5. 可维护性
- 注释完整性
- 代码结构清晰度
- 测试覆盖率
- 文档完整性

## 审查流程

1. **了解上下文**：先读取相关配置文件，了解项目规范
2. **逐文件审查**：对每个文件进行详细检查
3. **交叉验证**：检查文件间的依赖和调用关系
4. **汇总报告**：生成结构化的审查报告

## 输出格式

    ```markdown
    # 代码审查报告

    ## 📊 概述

    | 指标 | 值 |
    |------|-----|
    | 审查文件数 | X |
    | 发现问题数 | X |
    | Critical | X |
    | Major | X |
    | Minor | X |

    ## 🚨 Critical 问题

    ### [问题标题]
    - **文件**：`path/to/file.ts`
    - **行号**：10-15
    - **描述**：问题描述
    - **建议**：修复建议

        ```typescript
        // 问题代码 
       \````

        ```typescript
        // 建议修改
     \````

    ## ⚠️ Major 问题
    [同上格式]
    
    ## 💡 Minor 问题
    [同上格式]
    
    ## ✅ 亮点
    [代码中做得好的地方]
    
    ## 📝 总结
    [总体评价和改进建议]
    \````

## 注意事项

1. 不要修改任何代码，只做审查
2. 对于不确定的问题，标注为"待确认"
3. 给出的建议要具体可操作
4. 考虑项目的技术栈和规范

``` 

**使用示例** ：

```
用户：请审查 src/services/user.ts 文件

Claude Code：
"我来启动 code-reviewer 子代理进行审查..."

[SubAgent 执行]
1. 读取 src/services/user.ts
2. 读取项目配置了解规范
3. 逐行分析代码
4. 生成审查报告

[返回结果]
# 代码审查报告

## 📊 概述

| 指标 | 值 |
|------|-----|
| 审查文件数 | 1 |
| 发现问题数 | 3 |
| Critical | 0 |
| Major | 1 |
| Minor | 2 |

## ⚠️ Major 问题

### 缺少参数验证
- **文件**：`src/services/user.ts`
- **行号**：25
- **描述**：`updateUser` 函数没有验证 `userId` 参数
- **建议**：添加参数验证

    ```typescript
    // 当前代码
    async function updateUser(userId: string, data: UserData) {
      return db.user.update({ where: { id: userId }, data });
    }
    \````

    ```typescript
    // 建议修改
    async function updateUser(userId: string, data: UserData) {
      if (!userId || typeof userId !== 'string') {
        throw new Error('Invalid userId');
      }
      return db.user.update({ where: { id: userId }, data });
    }
    \````

## 💡 Minor 问题
...

## ✅ 亮点
- 使用了 TypeScript 类型定义
- 函数命名清晰
- 代码结构简洁

## 📝 总结
整体代码质量良好，建议补充参数验证逻辑，提高代码健壮性。

``` 

### 6\. 最佳实践

#### 6.1 何时使用 SubAgent

**✅ 适合使用 SubAgent 的场景** ：

场景| 说明  
---|---  
代码探索| 不知道代码在哪里，需要广泛搜索  
项目分析| 分析整个项目的结构、依赖关系  
代码审查| 检查多个文件的代码质量  
重构评估| 评估重构的影响范围  
文档生成| 基于代码生成文档  
  
**❌ 不适合使用 SubAgent 的场景** ：

场景| 原因  
---|---  
读取已知文件| 直接用 read_file 更快  
简单搜索| 一次 search_content 就能搞定  
修改代码| SubAgent 应该只读不写  
执行命令| SubAgent 不应该有执行权限  
  
**判断标准** ：

搜索多个文件/目录

分析代码关系

汇总大量信息

读一个文件

搜索一个关键词

执行一个命令

🤔 判断任务类型

需要什么？

✅ 用 SubAgent

⚡ 直接用工具

#### 6.2 SubAgent 设计原则

**1\. 单一职责**

每个 SubAgent 只做一件事。

```
# 好的设计
code-explorer: 只负责代码探索
code-reviewer: 只负责代码审查
test-generator: 只负责生成测试

# 不好的设计
super-agent: 什么都能做

``` 

**2\. 明确边界**

清晰定义输入输出。

```markdown
## 输入
- 任务描述（字符串）
- 目标文件/目录（可选）

## 输出
- 结构化报告（Markdown 格式）
- 包含：概述、详情、建议

``` 

**3\. 精简输出**

只返回必要信息，不要返回原始数据。

```
# 好的输出
"项目使用 React 18 + TypeScript，入口文件是 src/main.tsx，共有 15 个组件"

# 不好的输出
[返回所有文件的完整内容]

``` 

**4\. 可组合性**

多个 SubAgent 可以协作完成复杂任务。

👤 用户：帮我重构这个模块

🔍 code-explorer  
分析当前实现

📋 code-reviewer  
找出问题

🧠 主 Agent  
制定重构方案

💻 主 Agent  
执行重构

✅ code-reviewer  
检查重构结果

#### 6.3 常见问题与解决方案

**Q1: SubAgent 返回结果太长怎么办？**

A: 在 SubAgent 的提示词中明确限制输出长度：

```markdown
## 输出限制
- 总结部分不超过 200 字
- 问题列表最多 10 条
- 每条问题描述不超过 50 字

``` 

**Q2: SubAgent 搜索不到想要的内容？**

A: 检查以下几点：

  1. 搜索关键词是否准确
  2. 文件路径是否正确
  3. 是否需要使用正则表达式
  4. 是否被 .gitignore 排除



**Q3: 如何调试 SubAgent？**

A: 可以通过以下方式：

  1. 让 SubAgent 返回详细的执行日志
  2. 分步执行，检查每一步的结果
  3. 在提示词中添加 “请说明你的思考过程”



**Q4: SubAgent 执行太慢怎么办？**

A: 优化策略：

  1. 缩小搜索范围（指定具体目录）
  2. 使用更精确的搜索模式
  3. 限制读取的文件数量
  4. 并行执行多个搜索



### 7\. 进阶话题

#### 7.1 多 SubAgent 协作

复杂任务可能需要多个 SubAgent 协作完成。

**串行调用** ：

A 的结果作为 B 的输入。

👤 用户：帮我优化这个模块的性能

🔍 code-explorer  
分析代码结构

📋 返回：模块包含 5 个文件  
主要逻辑在 service.ts

⚡ performance-analyzer  
分析性能问题

📋 返回：发现 3 个性能问题  
最严重的是 N+1 查询

🧠 主 Agent  
制定优化方案

**并行调用** ：

同时启动多个 SubAgent，加快执行速度。

👤 用户：全面分析这个项目

🚀 并行执行

🔍 code-explorer  
项目结构分析

📦 dependency-analyzer  
依赖分析

🔒 security-scanner  
安全扫描

✨ code-quality-checker  
代码质量检查

📊 汇总所有结果

📋 生成综合报告

**实际案例** ：

串行执行

并行执行

👤 用户：帮我做一次完整的代码审查

🧠 主 Agent 规划

🔍 code-explorer

🔒 security-scanner

📋 项目结构：React + Node.js  
共 50 个文件

⚠️ 发现 2 个安全风险  
依赖漏洞、硬编码密钥

📋 code-reviewer  
基于以上分析进行审查

📊 主 Agent  
综合分析报告

#### 7.2 SubAgent 与 MCP 的关系

MCP（Model Context Protocol）是 Anthropic 提出的模型上下文协议，用于扩展 AI 的能力。

**关系说明** ：

🧠 主 Agent

🔍 SubAgent

可以使用主 Agent 的所有工具

包括 MCP 工具

可用工具

MCP 工具

数据库查询

API 调用

外部服务

内置工具

read_file

search

...

**扩展 SubAgent 能力** ：

通过 MCP，SubAgent 可以：

  1. 查询数据库，分析数据模式
  2. 调用外部 API，获取更多信息
  3. 与其他服务集成，完成复杂任务



**示例** ：

```markdown
# Database Analyzer SubAgent

## 可用工具
- MCP: database_query（查询数据库）
- MCP: schema_info（获取表结构）
- read_file（读取代码中的 SQL）

## 任务
分析数据库设计，检查：
1. 表结构是否合理
2. 索引是否完整
3. 外键关系是否正确

``` 

#### 7.3 未来发展趋势

**1\. 更智能的 SubAgent 调度**

未来的 AI 编程助手可能会自动判断何时需要调用 SubAgent，以及调用哪个 SubAgent。

👤 用户：帮我修复这个 bug

🧠 AI 自动分析

🔍 需要先定位问题  
→ 调用 code-explorer

🐛 需要分析原因  
→ 调用 debugger

🧪 需要验证修复  
→ 调用 test-runner

**2\. 自学习的 SubAgent**

SubAgent 可能会根据使用情况自动优化自己的行为：

  * 记住常用的搜索模式
  * 学习项目的代码风格
  * 适应开发者的偏好



**3\. 跨工具的 SubAgent 标准化**

目前各工具的 SubAgent 实现不统一，未来可能会出现标准化协议：

  * 统一的定义格式
  * 统一的调用接口
  * 跨工具的 SubAgent 共享



### 8\. 总结

#### 8.1 核心要点回顾

写这篇文章的过程中，我对 SubAgent 的理解更加深刻了。说实话，一开始我也觉得这不就是让 AI 调用另一个 AI 嘛，有什么特别的？但深入研究后发现，这里面的设计思想还是很精妙的。

总结一下核心要点：

  1. **SubAgent 是什么** ：主 Agent 可以调用的专门化子代理，拥有独立的上下文空间，专注于特定任务。

  2. **为什么需要 SubAgent** ：解决上下文窗口限制、实现专业化分工、降低 Token 消耗。

  3. **如何使用 SubAgent** ：

     * 代码探索用 `code-explorer`
     * 代码审查用 `code-reviewer`
     * 复杂任务可以多 SubAgent 协作
  4. **如何自定义 SubAgent** ：

     * 明确职责边界
     * 设计好提示词
     * 定义输出格式
     * 集成到主 Agent



#### 8.2 选择建议

场景| 建议  
---|---  
简单的代码辅助| 不需要 SubAgent，直接用工具  
代码探索和分析| 使用内置的 code-explorer  
代码审查| 自定义 code-reviewer SubAgent  
复杂的多步骤任务| 多 SubAgent 协作  
  
#### 8.3 个人感悟

SubAgent 的出现，让我看到了 AI 编程助手的发展方向——不是一个"全能"的 AI，而是一个"会分工协作"的 AI 团队。

这和软件工程的发展历程很像：从单体应用到微服务，从单打独斗到团队协作。AI 也在走同样的路。

未来，我们可能会看到更多专业化的 SubAgent：

  * 专门做代码审查的
  * 专门做性能优化的
  * 专门做安全扫描的
  * 专门做文档生成的



每个 SubAgent 都是某个领域的专家，协作起来就能完成复杂的开发任务。

### 参考资料

  * [Claude Code Best Practices - Anthropic](https://www.anthropic.com/engineering/claude-code-best-practices)
  * [AGENTS.md Official Website](https://agents.md/)
  * [Model Context Protocol (MCP) - Anthropic](https://www.anthropic.com/news/model-context-protocol)
  * [Cursor Agent Mode Documentation](https://cursor.sh/docs)
  * [GitHub Copilot Workspace](https://github.com/features/copilot)