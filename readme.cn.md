# 轻量项目级记忆系统

这是一个为 AI 编码代理设计的轻量项目级记忆系统。它通过仓库内的 Markdown 文件，让 Claude Code、Codex、其他 CLI agent 或多个会话共享项目状态、缺陷、决策和交接信息。

目标不是建立复杂的文档治理系统，而是让 agent 能够快速知道：

- 项目是什么
- 当前做到哪了
- 有哪些 bug 要优先处理
- 之前做过哪些长期决策
- 另一个 session 做了什么
- 下一步应该继续哪里

## 设计原则

核心原则是：

> 读得快，写得少，只记录会影响下一次 agent 判断的内容。

因此本系统刻意保持精简：

- 不使用数据库
- 不引入锁文件
- 不维护复杂索引
- 不把 `.mem/` 变成第二套提示词
- 不记录临时噪音
- 不保存敏感信息
- 不在用户未明确要求时擅自初始化整套记忆文件

## 仓库结构

```text
project-memory-skill/
├── readme.md                  # 英文 README
├── readme.cn.md               # 本文件（中文）
├── system-prompt.md           # 系统提示词（英文）— 加入 CLAUDE.md / AGENTS.md
├── system-prompt.cn.md        # 系统提示词（中文）
└── project-memory/
    ├── SKILL.md               # Skill 定义（英文）— agent 默认使用
    └── skill.cn.md            # Skill 定义（中文）— 记忆内容主要用中文
```

### 双语设计

| 文件 | 语言 | 用途 |
|---|---|---|
| `system-prompt.md` | 英文 | 英文 agent 的系统提示词 |
| `system-prompt.cn.md` | 中文 | 中文 agent 的系统提示词 |
| `project-memory/SKILL.md` | 英文 | Skill 定义、模板和规则（英文） |
| `project-memory/skill.cn.md` | 中文 | Skill 定义、模板和规则（中文） |

选择与你的项目和团队匹配的语言版本。两个版本功能完全一致。

## 记忆文件结构

初始化后，项目根目录会包含：

```text
project-root/
├── CLAUDE.md
├── AGENTS.md
├── LOOP.md
├── BUGS.md
└── .mem/
    ├── project.md
    ├── state.md
    ├── decisions.md
    └── handoff.md
```

### 各文件职责

| 文件 | 职责 |
|---|---|
| `CLAUDE.md` | Claude Code 入口 — 只写 `@AGENTS.md` |
| `AGENTS.md` | 所有 agent 共用的项目规则 |
| `BUGS.md` | 轻量 bug 队列（P0–P3） |
| `LOOP.md` | 长期目标和循环约束 |
| `.mem/project.md` | 稳定项目事实 |
| `.mem/state.md` | 当前进度和下一步 |
| `.mem/decisions.md` | 长期 ADR 决策 |
| `.mem/handoff.md` | 跨 session 交接记录 |

## 一键安装

### Claude Code

对 Claude Code 说：

```text
从 https://github.com/RRFRRF/project-memory-skill 安装 project-memory skill。使用[英文/中文]版本。把系统提示词加入我的用户级 CLAUDE.md。
```

Claude Code 会：

1. 克隆仓库并读取 skill 文件。
2. 复制 `SKILL.md`（英文）或 `skill.cn.md`（中文）作为 project-memory skill。
3. 将 `system-prompt.md`（英文）或 `system-prompt.cn.md`（中文）的内容追加到你的 `~/.claude/CLAUDE.md`。

### Codex

对 Codex 说：

```text
从 https://github.com/RRFRRF/project-memory-skill 安装 project-memory skill。使用[英文/中文]版本。把系统提示词加入我的用户级 AGENTS.md。
```

Codex 会：

1. 克隆仓库并读取 skill 文件。
2. 复制 `SKILL.md`（英文）或 `skill.cn.md`（中文）作为 project-memory skill。
3. 将 `system-prompt.md`（英文）或 `system-prompt.cn.md`（中文）的内容追加到你的 `~/.codex/AGENTS.md`。

### 任意 Agent

对于支持自定义指令的任意 agent：

1. 选择语言：英文 → `system-prompt.md` + `SKILL.md`，中文 → `system-prompt.cn.md` + `skill.cn.md`
2. 将系统提示词内容添加到 agent 的用户级配置文件。
3. 将 skill 文件加载到 agent 的 skill 系统中，或粘贴到项目的 `AGENTS.md`。

## 初始化

在存量项目里，对 agent 说：

```text
按 project-memory skill 初始化项目记忆系统。扫描仓库，能确认的写真实信息，不能确认的不要编造，保持模板。
```

如果你只是让 agent 修一个小 bug 或做一次性改动，agent 不应擅自创建整套记忆系统；它最多可以提醒你这个项目适合初始化。

## 更新频率建议

| 文件 | 更新时机 |
|---|---|
| `.mem/state.md` | 有实质进展、阻塞、下一步变化 |
| `.mem/handoff.md` | 暂停、结束、切换上下文、交接 |
| `BUGS.md` | 发现、修复、验证、延后 bug |
| `.mem/decisions.md` | 长期架构/工具/接口决策 |
| `.mem/project.md` | 稳定事实变化 |
| `LOOP.md` | 用户给长期目标或持续约束 |

默认只更新一个最相关的文件。不要为了形式同时更新所有文件。

## 安全说明

这些文件是明文，通常会进入 git。不要写入：

- API key
- 密码
- token
- 私钥
- `.env` 内容
- 生产密钥
- session cookie
- 敏感个人信息

## 适合谁

- Claude Code 多窗口开发
- Codex 多 session 接力
- 多 agent 并行修 bug
- 长任务循环迭代
- 存量项目快速建立 agent 工作记忆
- 需要轻量状态追踪但不想引入外部服务

## 许可证

MIT
