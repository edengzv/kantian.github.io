---
title: "OpenAI Symphony 深度解析，把项目管理交给 AI Agent"
date: 2026-05-14 10:00:00 +0800
tags: [Symphony, AI Agent, Codex, OpenAI]
categories: [AI/ML]
description: OpenAI 开源了 Symphony，一个让 AI Agent 自主完成项目工作的编排服务。这篇文章从架构设计到核心实现，深入剖析 Symphony 如何把 Linear 看板上的任务自动转化为代码变更。
---


事情是这样的。

OpenAI 开源了一个叫 Symphony 的项目。

坦率的讲，我第一次看到这个项目的时候，第一反应不是惊讶。

而是，这名字起得挺贴切。

Symphony 的意思是交响乐。把多个 AI 编码代理编排起来，各自处理不同的任务，协同完成一个项目的工作。

这确实像一场交响乐。

但真正让我感兴趣的，不是名字，而是它解决的问题。

我们都在用 Codex、Claude Code 这类编码代理。但用着用着，你会发现一个痛点，管理这些代理本身变成了一项工作。你要给每个代理分配任务、监控进度、处理失败、协调冲突。

Symphony 要解决的就是这个问题。

它把项目管理从「监督编码代理」变成了「管理工作本身」。

今天这篇文章，我想把 Symphony 这件事，系统地捋一遍。

![Symphony demo - 监控 Linear 看板并生成代理处理任务](images/symphony-demo-poster.jpg)

_Symphony 监控 Linear 看板，自动生成代理处理任务。代理完成后提供工作证明：CI 状态、PR 审查反馈、复杂度分析、演示视频。_

---

## 01 什么是 Symphony

先说一个比喻。

想象你在一家餐厅当经理。你手下有几个厨师（编码代理）。以前，你需要亲自给每个厨师分配任务，盯着他们做菜，处理他们遇到的问题。

Symphony 做的事情，就是让你从「盯着厨师做菜」变成「只管菜单和出菜质量」。

具体来说，Symphony 是一个长期运行的自动化服务。它做三件事：

第一，持续从 Linear 看板读取待处理的任务。

第二，为每个任务创建一个隔离的工作区。

第三，在工作区内启动一个编码代理（Codex app-server），让它自主完成任务。

代理完成后，Symphony 会提供工作证明，CI 状态、PR 审查反馈、复杂度分析、演示视频。工程师不需要监督 Codex，可以在更高层面管理工作。

这就是 Symphony 的核心价值，把项目工作转化为隔离的、自主的实现运行。

---

## 02 架构设计，五层抽象

Symphony 的架构设计非常清晰，分了五个抽象层。

**策略层（Policy Layer）**，由仓库定义的 WORKFLOW.md 控制。包括 prompt 模板和团队特定的任务处理规则。

**配置层（Configuration Layer）**，解析 YAML front matter 为类型化的运行时设置。处理默认值、环境变量和路径规范化。

**协调层（Coordination Layer）**，编排器。负责轮询循环、任务资格判断、并发控制、重试和协调。

**执行层（Execution Layer）**，工作区加代理子进程。负责文件系统生命周期、工作区准备和编码代理协议。

**集成层（Integration Layer）**，Linear 适配器。负责 API 调用和任务数据规范化。

**可观测性层（Observability Layer）**，日志加可选的状态仪表板。提供对编排器和代理行为的可见性。

这种分层设计的好处是，每一层都可以独立替换。你可以用不同的任务跟踪器（不只是 Linear），用不同的编码代理（不只是 Codex），用不同的工作区实现。

---

## 03 核心机制，编排器如何工作

编排器是 Symphony 的心脏。它是一个 GenServer（Elixir 的有状态进程），维护整个系统的运行时状态。

让我们看看它的工作循环。

```
on_tick(state):
  1. 协调运行中的问题（reconcile_running_issues）
  2. 验证配置（validate_dispatch_config）
  3. 从 Linear 获取候选任务（fetch_candidate_issues）
  4. 按优先级排序任务（sort_for_dispatch）
  5. 在有可用槽位时分发任务（dispatch_issue）
  6. 通知可观测性消费者（notify_observers）
  7. 调度下一次 tick（schedule_tick）
```

这个循环每 30 秒执行一次（可配置）。

### 任务分发规则

不是所有任务都会被立即处理。一个任务必须满足所有条件才能被分发：

- 它的状态在 active_states 中（默认是 Todo、In Progress）
- 它不在 terminal_states 中（默认是 Closed、Cancelled、Duplicate、Done）
- 它不在 running 中
- 它不在 claimed 中
- 全局并发槽位可用
- 按状态并发槽位可用
- 如果是 Todo 状态，所有 blocker 都必须是 terminal 状态

排序规则也很清晰：

1. 按优先级升序（1 最高，4 最低，null 排最后）
2. 按创建时间最早优先
3. 按标识符字典序作为决胜条件

### 重试和退避

任务执行失败后，Symphony 不会无限重试。它使用指数退避策略：

```
delay = min(10000 * 2^(attempt - 1), max_retry_backoff_ms)
```

默认最大退避时间是 5 分钟。如果是正常退出（任务完成但仍在 active 状态），则使用 1 秒的短延迟进行连续重试。

### 协调（Reconciliation）

每次 tick 开始时，编排器会先协调运行中的任务。这包括：

**Stall 检测**，如果某个任务超过 stall_timeout_ms（默认 5 分钟）没有活动，就终止它并重试。

**状态刷新**，从 Linear 获取所有运行中任务的当前状态。如果任务变为 terminal 状态，就终止代理并清理工作区。如果任务变为非 active 状态，就终止代理但不清理工作区。

---

## 04 工作区管理，隔离与安全

每个任务都有一个独立的工作区。工作区路径是：

```
<workspace.root>/<sanitized_issue_identifier>
```

比如，任务 MT-649 的工作区可能是 `/tmp/symphony_workspaces/MT-649`。

工作区有严格的安全不变量：

**不变量 1**，编码代理只能在工作区路径内运行。启动子进程前，必须验证 cwd == workspace_path。

**不变量 2**，工作区路径必须在配置的工作区根目录下。规范化两个路径为绝对路径后，要求 workspace_path 以 workspace_root 为前缀。

**不变量 3**，工作区键名是 sanitized 的。只允许 [A-Za-z0-9._-] 字符，其他字符替换为下划线。

### 工作区钩子

工作区支持四个生命周期钩子：

- **after_create**，工作区首次创建时运行。失败则中止工作区创建。
- **before_run**，每次代理尝试前运行。失败则中止当前尝试。
- **after_run**，每次代理尝试后运行。失败则记录日志但忽略。
- **before_remove**，工作区删除前运行。失败则记录日志但忽略。

钩子超时默认 60 秒。

---

## 05 Codex 集成，App-Server 模式

Symphony 使用 Codex 的 App-Server 模式来运行编码代理。这是一种 JSON-RPC 2.0 协议，通过 stdio 传输。

### 会话启动流程

1. 在隔离的工作区中启动 Codex app-server 子进程
2. 使用 Codex app-server 协议初始化会话
3. 创建或恢复编码代理线程
4. 提供工作区路径作为线程/轮次的工作目录
5. 使用渲染后的任务 prompt 启动第一轮
6. 后续连续轮次在同一线程上使用连续指导，而不是重新发送原始任务 prompt

### 连续轮次（Continuation Turns）

这是 Symphony 的一个关键设计。如果一个任务完成了一轮，但任务仍在 active 状态，Symphony 不会重新启动代理。它会在同一线程上启动新一轮，使用连续指导而不是原始任务 prompt。

这样可以保持上下文连续性，避免重复工作。

### 超时和错误处理

Symphony 定义了三种超时：

- **read_timeout_ms**（默认 5 秒），启动和同步请求的请求/响应超时
- **turn_timeout_ms**（默认 1 小时），总轮次流超时
- **stall_timeout_ms**（默认 5 分钟），由编排器根据事件不活动强制执行

错误映射包括：codex_not_found、invalid_workspace_cwd、response_timeout、turn_timeout、port_exit、response_error、turn_failed、turn_cancelled、turn_input_required。

---

## 06 工作流配置，WORKFLOW.md

Symphony 的配置全部在 WORKFLOW.md 文件中。这是一个 Markdown 文件，包含可选的 YAML front matter 和 Markdown 正文。

YAML front matter 用于配置运行时设置，Markdown 正文用作 Codex 会话的 prompt 模板。

### 配置示例

```yaml
---
tracker:
  kind: linear
  project_slug: "my-project"
  active_states:
    - Todo
    - In Progress
  terminal_states:
    - Closed
    - Cancelled
    - Duplicate
    - Done
polling:
  interval_ms: 5000
workspace:
  root: ~/code/symphony-workspaces
hooks:
  after_create: |
    git clone --depth 1 https://github.com/my-org/my-repo .
agent:
  max_concurrent_agents: 10
  max_turns: 20
codex:
  command: codex app-server
  approval_policy: never
  thread_sandbox: workspace-write
---
```

### Prompt 模板

Markdown 正文使用 Liquid 兼容的模板引擎。可用的模板变量：

- **issue**，规范化后的任务对象（包含 id、identifier、title、description、priority、state、labels、blocked_by 等）
- **attempt**，重试次数（首次运行时为 null，重试时为整数）

```markdown
You are working on a Linear ticket {{ issue.identifier }}.

Title: {{ issue.title }}
Description: {{ issue.description }}
Current status: {{ issue.state }}

{% if attempt %}
This is retry attempt #{{ attempt }}. Resume from the current workspace state.
{% endif %}
```

### 动态重载

WORKFLOW.md 的更改会被自动检测并重新应用，无需重启服务。这包括：

- 轮询间隔
- 并发限制
- active/terminal 状态
- Codex 设置
- 工作区路径和钩子
- Prompt 内容

无效的重载不会崩溃服务，而是保持最后一个已知良好的配置并发出错误日志。

---

## 07 可观测性，日志与仪表板

Symphony 提供了两种可观测性方式：结构化日志和可选的 Web 仪表板。

### 结构化日志

所有日志必须包含以下上下文字段：

- **issue_id**，任务内部 ID
- **issue_identifier**，人类可读的任务标识符（如 MT-649）
- **session_id**，编码代理会话 ID（格式为 thread_id-turn_id）

日志格式使用稳定的 key=value 短语，包含操作结果（completed、failed、retrying 等）。

### Web 仪表板

可选的 HTTP 服务器提供：

- **/**，人类可读的仪表板（Phoenix LiveView）
- **/api/v1/state**，当前系统状态的 JSON API
- **/api/v1/<issue_identifier>**，特定任务的运行时/调试详情
- **/api/v1/refresh**，触发立即轮询和协调

仪表板显示运行中的会话、重试队列、token 消耗、运行时间总计、最新速率限制等。

![Symphony Elixir 仪表板截图](images/elixir-screenshot.png)

_Symphony Elixir 参考实现的实时仪表板。可以看到 19/50 个代理并发运行，吞吐量 658,875 tps，3800 万 token 消耗。_

### Token 会计

Symphony 跟踪每个会话和总计的 token 使用情况：

- input_tokens
- output_tokens
- total_tokens
- seconds_running

使用绝对线程总计（而非增量）来避免重复计算。

---

## 08 安全设计，信任边界

Symphony 的安全设计基于一个关键假设，每个实现定义自己的信任边界。

### 文件系统安全

强制要求：

- 工作区路径必须在配置的工作区根目录下
- 编码代理的 cwd 必须是当前运行的每任务工作区路径
- 工作区目录名必须使用 sanitized 标识符

推荐加固：

- 在专用操作系统用户下运行
- 限制工作区根权限
- 如果可能，在专用卷上挂载工作区根

### 秘密处理

- 支持 `$VAR` 间接引用在工作流配置中
- 不记录 API 令牌或秘密环境变量值
- 在不打印它们的情况下验证秘密的存在

### 钩子脚本安全

工作区钩子是 WORKFLOW.md 中的任意 shell 脚本。它们是受信任的配置，但：

- 钩子在工作区目录内运行
- 钩子输出应在日志中截断
- 钩子超时是必需的，以避免挂起编排器

### 强化指南

Symphony 提供了一些强化建议：

- 收紧 Codex 审批和沙箱设置
- 添加外部隔离层（OS/容器/VM 沙箱、网络限制、单独凭据）
- 过滤哪些 Linear 任务、项目、团队、标签符合分发条件
- 缩小 linear_graphql 工具的范围，使其只能读取或修改预期项目范围内的数据
- 减少代理可用的客户端工具、凭据、文件系统路径和网络目的地集

---

## 09 SSH 远程执行

Symphony 支持可选的 SSH 远程执行。编排器保持在一个中央主机上，但工作运行在一个或多个远程主机上。

### 执行模型

- 编排器仍然是轮询、声明、重试和协调的唯一真相来源
- worker.ssh_hosts 提供远程执行的候选 SSH 目标
- 每个工作运行分配给一个主机，该主机成为运行有效执行身份的一部分
- workspace.root 在远程主机上解释，而不是在编排器主机上
- 编码代理 app-server 通过 SSH stdio 启动，而不是作为本地子进程
- 同一工作生命周期内的连续轮次应保持在同一主机和工作区上

### 调度注意事项

- SSH 主机可以视为调度池
- 重试时可能更喜欢之前使用的主机
- worker.max_concurrent_agents_per_host 是可选的每主机共享上限
- 当所有 SSH 主机都达到容量时，调度应等待而不是静默回退到不同模式

---

## 10 参考实现，Elixir/OTP

Symphony 的参考实现使用 Elixir/OTP。选择 Elixir 的原因很直接：

Elixir 构建在 Erlang/BEAM/OTP 之上，非常适合监督长期运行的进程。它有一个活跃的工具和库生态系统。它还支持热代码重载，而无需停止正在运行的子代理，这在开发期间非常有用。

### 项目结构

```
elixir/
├── lib/
│   ├── symphony_elixir/
│   │   ├── orchestrator.ex    # 编排器（GenServer）
│   │   ├── agent_runner.ex    # 代理运行器
│   │   ├── workspace.ex       # 工作区管理
│   │   ├── config.ex          # 配置管理
│   │   ├── workflow.ex        # 工作流加载
│   │   ├── codex/
│   │   │   └── app_server.ex  # Codex app-server 客户端
│   │   ├── linear/
│   │   │   ├── adapter.ex     # Linear 适配器
│   │   │   ├── client.ex      # Linear GraphQL 客户端
│   │   │   └── issue.ex       # 任务模型
│   │   └── ssh.ex             # SSH 远程执行
│   └── symphony_elixir_web/   # Phoenix Web 仪表板
└── test/                      # ExUnit 测试
```

### 核心模块

- **Orchestrator**，GenServer，维护运行时状态，处理轮询、分发、协调、重试
- **AgentRunner**，执行单个 Linear 任务，管理工作区、prompt 构建、Codex 会话
- **Workspace**，工作区管理，创建、清理、钩子执行
- **Config**，配置管理，解析 WORKFLOW.md，应用默认值，环境变量解析
- **Codex.AppServer**，Codex app-server JSON-RPC 2.0 客户端
- **Linear.Adapter**，Linear GraphQL 适配器，任务获取、状态刷新、规范化

---

## 11 写在最后

Symphony 这个项目，表面看是一个编码代理编排工具，但往深了想，它触及了一个更根本的趋势。

我们正在从「使用编码代理」走向「管理编码代理的工作」。

以前，你用 Codex 或 Claude Code，你是那个「监督者」。你给代理分配任务，盯着它干活，处理它遇到的问题。

Symphony 做的事情，是让你从「监督者」变成「管理者」。你不再盯着每个代理，你管理的是任务列表、并发限制、重试策略、工作流规则。

这不是一个能靠简单工具解决的问题。因为一旦你开始管理多个代理的并发执行，你就需要处理：

- 任务调度和优先级
- 并发控制和资源限制
- 失败重试和错误恢复
- 工作区隔离和安全
- 可观测性和调试

Symphony 提供了一套完整的解决方案。但它不是唯一的答案。SPEC.md 的存在，意味着任何人都可以基于这个规范实现自己的 Symphony。

如果你是 AI 应用的开发者，我的建议是，看看这个项目。不是要你直接使用它，而是理解它解决的问题。

多花一点点时间，自己去研究一下。

毕竟，你的编码代理，可能现在就等着某个人说一句「Ignore all previous instructions」。

---

「注」本文所有代码示例均来自 openai/symphony 开源项目，仅用于技术分析和学习目的。

#Symphony #OpenAI #AI Agent #Codex #编码代理 #自动化
