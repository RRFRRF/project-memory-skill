# Lightweight Project Memory System

[中文说明](README.cn.md)

A lightweight project-level memory system for AI coding agents. It stores project context, progress, bug queue, decisions, and handoff notes as Markdown files inside a repository.

The goal is not heavy documentation management. The goal is to help agents quickly answer:

- What is this project?
- Where does it currently stand?
- Which bugs should be handled first?
- What durable decisions were already made?
- What did the previous session do?
- What should be done next?

## Core Principle

> Fast to read, cheap to update, and only records information that affects future agent decisions.

This system intentionally avoids:

- databases
- lock files
- complex indexes
- duplicated rule files
- noisy logs
- sensitive secrets
- automatic full initialization without explicit user intent

## Repository Structure

```text
project-memory-skill/
├── README.md                  # This file
├── README.cn.md               # Chinese README
├── system-prompt.md           # System prompt (English), add to CLAUDE.md / AGENTS.md
├── system-prompt.cn.md        # System prompt (Chinese)
├── project-memory/
│   └── SKILL.md               # English skill
└── project-memory-cn/
    └── SKILL.md               # Chinese skill
```

### Skill Choices

| Skill | Language | Install when |
|---|---|---|
| `project-memory` | English | Your project memory and assistant responses should default to English |
| `project-memory-cn` | Chinese | Your project memory and assistant responses should default to Chinese |

Both skills are functionally identical. They are split into two directories so installers and humans can choose a language by path without renaming files.

## Memory File Structure

After initialization, the project root will contain:

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

### File Responsibilities

| File | Purpose |
|---|---|
| `CLAUDE.md` | Claude Code entrypoint — just `@AGENTS.md` |
| `AGENTS.md` | Shared project rules for all agents |
| `BUGS.md` | Lightweight bug queue (P0–P3) |
| `LOOP.md` | Long-running objectives and loop constraints |
| `.mem/project.md` | Stable project facts |
| `.mem/state.md` | Current progress and next steps |
| `.mem/decisions.md` | Long-term ADR decisions |
| `.mem/handoff.md` | Cross-session handoff records |

## One-Click Install

### Recommended Prompts

For English:

```text
Install the project-memory skill from https://github.com/RRFRRF/project-memory-skill/tree/main/project-memory.
Also add the guidance from https://github.com/RRFRRF/project-memory-skill/blob/main/system-prompt.md to my user-level agent instructions.
```

For Chinese:

```text
Install the project-memory-cn skill from https://github.com/RRFRRF/project-memory-skill/tree/main/project-memory-cn.
Also add the guidance from https://github.com/RRFRRF/project-memory-skill/blob/main/system-prompt.cn.md to my user-level agent instructions.
```

### Claude Code

Tell Claude Code:

```text
Install the project-memory skill from https://github.com/RRFRRF/project-memory-skill/tree/main/project-memory.
Add the system prompt from system-prompt.md to my user-level CLAUDE.md.
```

For Chinese, use:

```text
Install the project-memory-cn skill from https://github.com/RRFRRF/project-memory-skill/tree/main/project-memory-cn.
Add the system prompt from system-prompt.cn.md to my user-level CLAUDE.md.
```

### Codex

Tell Codex:

```text
Install the project-memory skill from https://github.com/RRFRRF/project-memory-skill/tree/main/project-memory.
Add the system prompt from system-prompt.md to my user-level AGENTS.md.
```

For Chinese, use:

```text
Install the project-memory-cn skill from https://github.com/RRFRRF/project-memory-skill/tree/main/project-memory-cn.
Add the system prompt from system-prompt.cn.md to my user-level AGENTS.md.
```

### Any Agent

For any agent that supports custom instructions:

1. Choose a skill directory: `project-memory/` for English or `project-memory-cn/` for Chinese.
2. Install or copy that directory into the agent's skill system.
3. Add the matching system prompt to the agent's user-level config:
   - English: `system-prompt.md`
   - Chinese: `system-prompt.cn.md`

## Initialization

In an existing project, tell the agent:

```text
Initialize the project memory system using the project-memory skill. Inspect the repository, write only verified facts, and keep unknown items as placeholders without inventing details.
```

If you did not explicitly request initialization, the agent should not create the full memory system. It may suggest initialization if the task clearly needs cross-session coordination.

## Update Policy

| File | Update when |
|---|---|
| `.mem/state.md` | progress, blockers, or next steps change |
| `.mem/handoff.md` | pausing, finishing, switching context |
| `BUGS.md` | bugs are found, fixed, verified, or deferred |
| `.mem/decisions.md` | durable decisions are made |
| `.mem/project.md` | stable facts change |
| `LOOP.md` | user gives long-running goals or constraints |

Default to updating only the most relevant file.

## Security

These files are plaintext and may be committed to git. Never store:

- API keys
- passwords
- tokens
- private keys
- `.env` contents
- production secrets
- session cookies
- sensitive personal data

## Who Is This For

- Claude Code multi-window development
- Codex multi-session接力
- Multi-agent parallel bug fixing
- Long-running loop iteration
- Existing projects that need lightweight agent working memory
- Teams that want state tracking without external services

## License

MIT
