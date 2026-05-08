---
name: project-memory-cn
description: 初始化轻量级项目记忆系统。创建 CLAUDE.md、AGENTS.md、LOOP.md、BUGS.md 与 .mem/ 文件。初始化后日常维护规则已嵌入 AGENTS.md（通过系统提示词持续生效），无需重复调用本 skill。仅用于：初始化、记忆状态查询、压缩清理。默认中文。
---

# project-memory-cn

你负责维护一个轻量级项目记忆系统。它用于跨 agent、跨 session 的代码项目协作。

目标是让新 session 快速知道：

- 项目是什么
- 当前做到哪里
- 有哪些 bug 优先处理
- 做过哪些长期决策
- 上一个 session 留下了什么交接
- 下一步应继续哪里

核心原则：

> 读得快，写得少，只记录会影响下一次 agent 判断的内容。

默认使用中文回复和维护记忆文件，除非用户或项目要求英文。

---

## 1. 目标结构

在仓库根目录维护：

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

职责：

- `CLAUDE.md`：Claude Code 入口，只导入 `AGENTS.md`
- `AGENTS.md`：共享项目规则入口
- `BUGS.md`：轻量 bug 队列
- `LOOP.md`：长期目标和循环约束
- `.mem/project.md`：稳定项目事实
- `.mem/state.md`：当前进度和下一步
- `.mem/decisions.md`：长期 ADR 决策
- `.mem/handoff.md`：跨 session 交接记录

---

## 2. 激活场景

本 skill 的核心职责是**初始化**记忆系统。初始化完成后，`AGENTS.md` 中已包含完整的工作规则、更新原则、协作规则和安全规则——这些规则通过 `CLAUDE.md → @AGENTS.md` 作为系统提示词持续生效。日常更新 `.mem/state.md`、`BUGS.md`、`.mem/handoff.md` 等操作由 AGENTS.md 驱动，**无需重复调用本 skill**。

仅在以下场景调用本 skill：

- 初始化项目记忆 / 建立记忆系统
- 查看记忆状态（用户明确询问"记忆状态"）
- 压缩/清理记忆文件
- 用户明确要求使用 project-memory skill

注意：如果项目缺少记忆文件，但用户没有明确要求初始化，不要擅自创建整套记忆系统。可以简短提醒用户此项目适合初始化项目记忆。

---

## 3. 初始化协议

只有当用户明确要求初始化项目记忆时，才创建完整记忆系统。

用户要求初始化时：

1. 识别仓库根目录。
2. 生成本 session 代号。
3. 创建缺失文件：
   - `CLAUDE.md`
   - `AGENTS.md`
   - `LOOP.md`
   - `BUGS.md`
   - `.mem/project.md`
   - `.mem/state.md`
   - `.mem/decisions.md`
   - `.mem/handoff.md`
4. `CLAUDE.md` 写入一行：`@AGENTS.md`
5. 扫描仓库中可验证的信息，例如：
   - `README.md`
   - `package.json`
   - `pyproject.toml`
   - `requirements.txt`
   - `pnpm-lock.yaml`
   - `docker-compose.yml`
   - `Dockerfile`
   - `Makefile`
   - `src/`
   - `tests/`
6. 能确认的项目事实写入 `.mem/project.md`。
7. 不能确认的不要编造；少写 `未知`，统一放到 `To Verify`。
8. 在 `.mem/handoff.md` 登记当前 session。
9. 在 `.mem/decisions.md` 添加 `ADR-001`，说明采用该项目记忆系统。
10. 不要保存密钥、token、密码或 `.env` 内容。

---

## 4. 会话代号

每个 session 启动时创建唯一代号：

```text
<agent>-YYYYMMDD-HHMM-<short-id>
```

示例：

```text
claude-20260504-0930-a1b2
codex-20260504-0942-b8c1
agent-20260504-1010-k7d3
```

如果代理身份未知，用 `agent`。

---

## 5. 读取协议

非平凡工作开始前读取：

1. `AGENTS.md`
2. `BUGS.md`
3. `.mem/project.md`
4. `.mem/state.md`
5. `.mem/decisions.md`
6. `.mem/handoff.md`
7. 如涉及长期目标，再读取 `LOOP.md`

不要把全部记忆内容复述给用户，除非用户明确要求。

---

## 6. 更新原则

默认只更新最相关的文件，不要每次都同时改所有记忆文件。

更新触发条件：

- `.mem/state.md`：有实质进展、阻塞或下一步变化
- `.mem/handoff.md`：暂停、结束、切换上下文或交接
- `BUGS.md`：发现、修复、验证或延后 bug
- `.mem/decisions.md`：做出长期架构/工具/接口决策
- `.mem/project.md`：稳定事实变化
- `LOOP.md`：用户给出长期目标、循环目标或持续约束

---

## 7. 协作规则

`.mem/handoff.md` 是弱协作信号，不是严格锁。

如果看到其他近期 active session：

- 避免修改同一批文件
- 必须重叠时，在 `.mem/handoff.md` 的 Messages 里留下说明
- 不要因为看到 active session 就停止工作，除非确实存在冲突

如果某个 session 的 `Last Seen` 超过 2 小时，除非备注明确说明仍在工作，否则默认视为 stale，不阻塞当前工作。

如果只有自己工作：

- 可以直接更新 `.mem/state.md` 的全局状态

---

## 8. BUG 规则

`BUGS.md` 是轻量队列，不是完整 issue 系统。

优先级：

- `P0`：阻塞构建、启动、测试、数据完整性或关键流程
- `P1`：严重功能缺陷
- `P2`：重要但不阻塞
- `P3`：小问题、优化、清理

状态：

- `open`
- `doing`
- `fixed`
- `verified`
- `deferred`
- `wontfix`

规则：

- 开始有意义任务前检查相关 P0/P1 bug。
- 只优先处理与当前任务相关的 P0/P1 bug。
- 无关 bug 只记录或提醒，不打断当前任务。
- 修复 bug 后写明验证方式。
- 不要静默删除 bug；可以移动到 `Fixed`。

---

## 9. ADR 规则

`.mem/decisions.md` 只记录长期决策。

应该写 ADR 的例子：

- 选择某个框架
- 更换数据库
- 确定 browser automation 方案
- 确定 agent 架构
- 确定关键接口或数据模型
- 确定测试策略

不应该写 ADR 的例子：

- 修一个 bug
- 改变量名
- 增加一个小工具函数
- 临时调试结论
- 单次任务计划

---

## 10. LOOP 规则

`LOOP.md` 默认可以是 inactive。

只有当用户要求：

- 持续迭代
- 一直做直到完成
- 使用 goal/loop 模式
- 给长期目标或持续约束

才更新 `LOOP.md`。

`LOOP.md` 不应覆盖最新用户指令。最新用户指令优先。

---

## 11. 安全规则

记忆文件是明文，可能进入 git。

不要保存：

- API key
- 密码
- token
- 私钥
- session cookie
- `.env` 内容
- 生产密钥
- OAuth secret
- 敏感个人信息

如果用户要求保存这些内容，说明风险并拒绝写入记忆文件。

---

## 12. 冲突处理

如果记忆与代码冲突：

1. 以实际代码为准。
2. 说明记忆可能过时。
3. 在合适时更新记忆。

如果记忆与最新用户指令冲突：

1. 以最新用户指令为准。
2. 如果是长期变化，更新记忆。

文件优先级：

1. 最新用户指令
2. 实际代码和测试结果
3. `AGENTS.md`
4. `LOOP.md`
5. `BUGS.md`
6. `.mem/decisions.md`
7. `.mem/state.md`
8. `.mem/handoff.md`
9. `.mem/project.md`

---

## 13. 模板

### 13.1 CLAUDE.md

```markdown
@AGENTS.md
```

---

### 13.2 AGENTS.md

```markdown
# AGENTS.md

## 角色

你是一个 AI 编码代理，负责在本仓库中进行小而正确、可验证的代码修改。

默认使用中文回复用户并维护项目记忆，除非用户或项目另有要求。

## 项目记忆系统

本仓库使用轻量项目记忆系统：

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

## 必读

非平凡修改前读取：

1. `AGENTS.md`
2. `BUGS.md`
3. `.mem/project.md`
4. `.mem/state.md`
5. `.mem/decisions.md`
6. `.mem/handoff.md`
7. 如涉及长期目标，再读取 `LOOP.md`

## 工作规则

- 开始有意义任务前，检查 `BUGS.md` 中相关 P0/P1 缺陷。
- 只优先处理与当前任务相关的 P0/P1 bug；无关 bug 不打断当前任务。
- 取得实质进展、遇到阻塞或改变下一步时，更新 `.mem/state.md`。
- 暂停、结束、切换上下文或需要交接时，更新 `.mem/handoff.md`。
- 发现、修复、验证或延后 bug 时，更新 `BUGS.md`。
- 只有长期架构/工具/接口决策才写入 `.mem/decisions.md`。
- 只有稳定项目事实变化才更新 `.mem/project.md`。
- 只有用户给长期目标或循环约束时才更新 `LOOP.md`。
- 默认只更新最相关的记忆文件，不要为了形式同时修改全部文件。

## 协作规则

- 每个 session 创建唯一代号：`<agent>-YYYYMMDD-HHMM-<short-id>`。
- `.mem/handoff.md` 是弱协作信号，不是严格锁。
- 如果看到其他近期活跃 session，避免修改同一批文件。
- 如果某个 session 的 `Last Seen` 超过 2 小时，除非备注明确说明仍在工作，否则默认视为 stale。
- 如必须重叠修改，请在 `.mem/handoff.md` 留下协调说明。

## 开发规则

- 修改前先阅读相关源码。
- 优先做小而聚焦的改动。
- 不要无故大规模重写。
- 不要删除用户未明确要求删除的内容。
- 尽量运行最小有效验证。
- 无法验证时，说明原因和风险。

## 安全规则

不要在记忆文件中保存 API key、密码、token、私钥、`.env` 内容、生产密钥或敏感个人信息。

## 完成汇报

完成有意义工作后，简要说明：

1. 改了什么
2. 如何验证
3. 更新了哪些记忆文件
4. 下一步是什么
```

---

### 13.3 LOOP.md

```markdown
# LOOP

## Status

No active loop goal.

## Objective

-

## Done Criteria

- [ ]

## Constraints

-

## Procedure

1. Read `AGENTS.md`, `BUGS.md`, and `.mem/`.
2. Check current objective and constraints.
3. Pick the highest-impact next task.
4. Handle relevant P0/P1 bugs first.
5. Make one small verifiable change.
6. Run the smallest useful verification.
7. Update the relevant memory file.
8. Stop when done criteria or stop conditions are met.

## Stop Conditions

- Done criteria are complete.
- The next step requires user input.
- Verification fails and the cause is unclear.
- Continuing would require risky or broad architectural changes.
- Another active session is working on the same files and coordination is needed.

## Last Updated

-
```

---

### 13.4 BUGS.md

```markdown
# BUGS

轻量缺陷队列。开始有意义任务前检查相关 P0/P1 bug。

Priority: P0 blocking, P1 serious, P2 important, P3 minor.  
Status: open, doing, fixed, verified, deferred, wontfix.

## Open

### BUG-YYYYMMDD-001 — Title

- Priority: P1
- Status: open
- Owner: unassigned
- Area:
- Symptom:
- Repro:
- Notes:

## Fixed

### BUG-YYYYMMDD-000 — Title

- Fixed by:
- Verification:
- Notes:
```

---

### 13.5 .mem/project.md

```markdown
# Project Memory

稳定项目事实。只记录已验证、会影响后续开发判断的信息。

## Summary

-

## Verified Facts

-

## Tech Stack

-

## Architecture

-

## Commands

- Install:
- Dev:
- Test:
- Build:
- Lint:
- Typecheck:

## Known Pitfalls

-

## To Verify

-

## Last Updated

-
```

---

### 13.6 .mem/state.md

```markdown
# Current State

## Goal

-

## Now

-

## Progress

- [ ]

## Blockers

- None

## Next

1.

## Workstreams

- default: unassigned / idle / -

## Last Updated

-
```

---

### 13.7 .mem/decisions.md

```markdown
# Decisions

只记录会影响未来开发路径的长期决策。

## ADR Index

- ADR-001 — YYYY-MM-DD — Use lightweight project memory — accepted

## ADR-001 — Use lightweight project memory

- Date: YYYY-MM-DD
- Status: accepted
- Owner: session-id
- Context: 项目需要跨 agent、跨 session 共享状态、缺陷、决策和交接信息。
- Decision: 使用 `CLAUDE.md`、`AGENTS.md`、`LOOP.md`、`BUGS.md` 和 `.mem/` 组成轻量项目记忆系统。
- Alternatives:
  - 全局个人记忆
  - 数据库存储
  - 不保存项目记忆
- Consequences:
  - agent 能更容易接力和协作
  - 需要保持记忆精简并避免保存敏感信息
```

---

### 13.8 .mem/handoff.md

```markdown
# Handoff

跨 session 交接记录。它是弱协作信号，不是严格锁。

如果某个 session 的 `Last Seen` 超过 2 小时，除非备注明确说明仍在工作，否则默认视为 stale，不阻塞当前工作。

## Active Sessions

| Session | Last Seen | Status | Area | Note |
|---|---|---|---|---|
| session-id | YYYY-MM-DD HH:mm | active | startup | initialized |

## Messages

- YYYY-MM-DD HH:mm — from -> to: message

## Log

### YYYY-MM-DD HH:mm — session-id

- Did:
- Changed:
- Verified:
- Blocked:
- Next:
```

---

## 14. 状态查询

当用户问"记忆状态"或"当前记忆有什么"时：

1. 检查文件是否存在。
2. 简述每个文件当前记录的内容。
3. 列出 active sessions，并指出超过 2 小时未更新的 session 可能 stale。
4. 列出与当前任务相关的 open P0/P1 bugs。
5. 指出明显过时、冗余或缺失的信息。
6. 给出一个最有价值的清理建议。

---

## 15. 压缩规则

当记忆文件过长时：

1. 保留当前目标、阻塞、下一步。
2. 保留 open bugs。
3. 保留长期 ADR。
4. 合并旧 handoff 日志。
5. 删除重复和过时内容。
6. 不删除未验证 bug 或关键决策。
7. 压缩后说明改了哪些文件。
