# OpenHarness 项目分析报告

> 分析日期：2026-04-30
> 仓库：https://github.com/HKUDS/OpenHarness
> 版本：v0.1.7 | 许可证：MIT

---

## 一、项目概览

**OpenHarness** 是 Claude Code 的开源 Python 移植版——一个完整的 AI 编程 CLI 代理框架，由香港大学数据科学团队维护。

核心能力：

- 通过 CLI（`oh` 命令）启动一个 AI 编程代理
- 支持 10+ 模型提供商（Anthropic / OpenAI / Copilot / Gemini / DeepSeek / Ollama 等）
- 43+ 内置工具（文件操作、Shell 执行、Web 搜索、MCP、多 Agent 协调等）
- 插件/Skill/Hook 三层扩展机制
- 内置多 Agent Swarm 协调系统
- ohmo：接入聊天平台（Telegram/Slack/飞书等）自主编码
- Autopilot：自动扫描 Issue → 编码 → 开 PR

---

## 二、架构总览

```
OpenHarness
├── cli.py                  # oh 命令入口（typer 框架）
├── engine/                 # 核心引擎
│   ├── query_engine.py     # QueryEngine 类，会话的起点
│   ├── query.py            # run_query()，核心工具循环
│   ├── messages.py         # 消息类型定义
│   └── stream_events.py    # 流式事件类型
├── api/                    # Provider 客户端层
│   ├── client.py           # Anthropic 客户端
│   ├── openai_client.py    # OpenAI 兼容客户端
│   ├── copilot_client.py   # Copilot 客户端
│   ├── registry.py         # Provider 注册中心（支持多 Provider）
│   └── provider.py         # Provider 检测
├── tools/                  # 工具系统（43+ 内置工具）
│   ├── base.py             # BaseTool 抽象类
│   ├── __init__.py         # 工具注册
│   └── 各工具实现文件
├── swarm/                  # 多 Agent 协调
│   ├── types.py            # Agent 身份和配置
│   ├── subprocess_backend.py
│   ├── mailbox.py          # 跨 Agent 通信
│   ├── worktree.py         # Git 工作树隔离
│   └── lockfile.py         # 协调锁
├── mcp/                    # MCP 协议支持
├── hooks/                  # 生命周期钩子
├── plugins/                # 插件系统
├── memory/                 # 持久化记忆
├── config/                 # 多级配置解析
├── auth/                   # 认证（API Key / OAuth / Copilot）
├── ui/                     # 用户界面
│   ├── react_launcher.py   # React+Ink TUI（主）
│   └── textual_app.py      # Textual 备用 TUI
├── sandbox/                # Docker 沙箱
├── channels/               # 聊天平台适配器
│   ├── adapter.py          # ChannelBridge
│   ├── bus/                # 消息总线
│   └── impl/               # Telegram/Slack/Discord/飞书等 10+ 平台
├── services/               # 基础服务
│   ├── compact/            # 对话压缩
│   └── cron/               # 定时任务
└── autopilot/              # 自动维护系统
```

---

## 三、与 Claude Code 对比

| 维度 | OpenHarness | Claude Code |
|------|-------------|-------------|
| **语言** | Python | TypeScript |
| **许可证** | MIT（完全开源） | 闭源 |
| **Provider** | 10+（Anthropic/OpenAI/Copilot/Gemini 等） | 仅 Anthropic |
| **多 Agent** | 内置 Swarm 系统（进程/tmux 后端） | 无 |
| **聊天平台** | ohmo：Telegram/Slack/Discord/飞书等 | 无 |
| **MCP** | 原生支持（stdio + HTTP） | 支持 |
| **自动维护** | Autopilot 系统 | 无 |
| **插件系统** | 插件/Skill/Hook 三层扩展 | Skill + Hook |
| **沙箱** | Docker 隔离 | Docker 隔离 |
| **对话压缩** | 微压缩 + LLM 压缩 + 自动触发 | 有 |
| **适用人群** | 研究者、想定制/扩展的开发者 | 终端用户 |

---

## 四、核心设计亮点

### 4.1 多 Provider 架构

一个 `ProviderSpec` 就能注册新模型，支持混合路由。不受限于 Anthropic，可对接 OpenAI、DeepSeek、Ollama 等。Provider 配置为命名 profile，支持按 profile 切换凭证。

### 4.2 Swarm 多 Agent 系统

- 子进程 / tmux / iTerm2 三种后端
- Mailbox 消息队列实现 Agent 间通信
- Worktree 隔离（每个 Agent 独立 Git 工作树）
- Lockfile 协调 + TaskManager 生命周期管理

### 4.3 完善的工具生态（43+ 内置工具）

覆盖：文件读写/编辑、Shell 执行、Web 抓取/搜索、MCP 集成、Agent 启停、任务管理、计划模式、Git 工作树、定时任务、LSP、配置管理等。

### 4.4 对话压缩（Compaction）

- 微压缩：低成本清除历史工具结果
- LLM 压缩：用 AI 做摘要压缩
- 自动触发：超阈值自动执行

### 4.5 三层扩展机制

- **Skill**：Markdown 文件，定义触发词和指令
- **Plugin**：可贡献工具/命令/Agent/Hook/MCP Server
- **Hook**：6 种生命周期事件 × 4 种执行方式（Shell/HTTP/Prompt/Agent）

---

## 五、可行性评估：二次开发投入生产

### 有利因素 ✅

- 代码模块化好，Pydantic 类型约束 + mypy strict
- 测试覆盖 114+，基础层可信赖
- MIT 许可证，无商用限制
- 扩展机制完善，不改核心代码即可加功能
- Provider 抽象层干净，改模型成本低

### 风险点 ⚠️

| 风险 | 说明 |
|------|------|
| v0.1.7 | 未到 1.0，API 可能大改 |
| React TUI 子进程 | 崩溃/断连容错需加固 |
| 并发安全 | memory 用锁文件，但整体未为高并发设计 |
| 文档不足 | 生产部署文档基本没有 |
| 依赖链长 | Python + Node.js + Docker，部署 Ops 成本高 |
| Secrets 管理 | credentials.json 明文存储 |
| 社区小 | 遇到问题可能无人响应 |

### 生产化建议

- **推荐用法**：作为 Agent 引擎库（import 核心模块），自己封装上层服务。**不要直接部署 CLI/TUI**
- **优先加固**：认证加密、错误恢复、可观测性、并发安全
- **适合场景**：内部工具、自动化流水线、私有 Agent 服务，而非面向客户的 SaaS

---

## 六、学习路径（由简入难 6 阶段）

### 阶段一：跑起来（1 天）
```
目标：能用 oh 命令行干活
步骤：
1. pip install -e .         # 本地安装
2. oh setup                 # 配置 provider
3. oh -p "列出当前目录文件"  # 非交互模式调用
4. oh --dry-run             # 预览完整配置和工具列表
验证：成功完成一次模型调用
```

### 阶段二：理解引擎核心（2-3 天）
```
目标：搞懂一次工具调用从发起到返回的全流程
重点文件（按顺序读）：
1. engine/query_engine.py   → QueryEngine，会话起点
2. engine/query.py           → run_query()，核心工具循环
3. engine/messages.py        → 消息类型
4. api/client.py             → AnthropicClient，请求收发
5. tools/base.py             → BaseTool 抽象

动手：写脚本 import QueryEngine 发消息，加自定义 Tool
```

### 阶段三：掌握扩展机制（2-3 天）
```
目标：用 Plugin / Skill / Hook 三种方式扩展功能
动手：
1. 写一个 .md skill 文件（如天气查询）
2. 做一个最小 Plugin，贡献一个自定义工具
3. 配置 POST_TOOL_USE hook 监听工具调用
4. 连接外部 MCP server
```

### 阶段四：多 Agent 与 Swarm（3-5 天）
```
目标：理解多 Agent 协调机制
重点：swarm/types.py → subprocess_backend.py → mailbox.py → worktree.py
动手：
- 启动两个子 agent 协作完成任务
- 实现一个自定义后端（如 SSH 远程 agent）
```

### 阶段五：深入定制（1 周）
```
目标：为生产场景改造
方向（选其一）：
a. Provider 扩展：接入公司自有模型
b. 工具扩展：加公司内部 API 工具
c. UI 定制：重写前端/接入自己的聊天平台
d. 安全加固：凭证加密、审计日志
```

### 阶段六：ohmo + Autopilot（1 周+）
```
目标：部署自主编码的 Agent 服务
1. 配置 ohmo gateway 连接聊天平台
2. 理解 autopilot 任务扫描和执行流程
3. 定制 soul（人格）和 user（用户上下文）
```

---

## 七、适用场景

### 最佳场景 ✅

| 场景 | 理由 |
|------|------|
| AI 编码助手（内部版） | 多 Provider 支持对接私有模型，Hook 系统可注入代码审查/安全策略 |
| 自动化 DevOps Agent | Shell+Git+文件工具完备，Swarm 做流水线编排，Autopilot 做 CI 修复 |
| 研究 LLM Agent 框架 | 对比 Claude Code 的实现，学习生产级 Agent 的设计取舍 |
| 多平台个人开发助手 | ohmo 接入 Telegram/飞书，移动端触发编码任务 |
| Multi-Agent 实验平台 | Swarm+Mailbox+Worktree 提供开箱的多 Agent 沙箱 |

### 勉强可用 ⚠️

| 场景 | 风险 |
|------|------|
| 面向客户的产品 | TUI 不可靠、并发未验证、安全加固不够 |
| 企业级 Agent 平台 | 缺 RBAC、审计日志、高可用部署方案 |
| 低代码平台 | 需额外封装 visual builder，核心是 CLI |

### 不适合 ❌

| 场景 | 原因 |
|------|------|
| 高并发在线服务 | 引擎设计给交互式会话，非请求-响应模式 |
| 嵌入式/移动端 | Python + Node.js 依赖太重 |
| 非技术用户 | 完全 CLI 驱动 |

---

## 八、总结

OpenHarness 最适合作为 **AI Agent 的研究平台和内部工具基底**。它的核心价值在于：

1. **源码开放**，可以深入理解生产级 AI 编程代理的内部机制
2. **多 Provider + 多层扩展**，灵活性远高于绑定单一模型的工具
3. **Swarm + ohmo + Autopilot** 三位一体，覆盖了从研究到自动化的完整光谱

如果目标是生产环境运行编码 Agent 服务，用它做内核、自封装上层是可行的路径，但不建议直接部署其 TUI 版本。
