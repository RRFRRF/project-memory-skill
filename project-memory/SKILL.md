---
name: project-memory
description: Initialize a lightweight project memory system. Creates CLAUDE.md, AGENTS.md, LOOP.md, BUGS.md and .mem/ files. After initialization, daily maintenance rules are embedded in AGENTS.md (active via system prompts) — no need to re-invoke this skill. Only use for initialization, memory status queries, compression or cleanup. Defaults to English.
---

# project-memory

You maintain a lightweight project memory system for cross-agent, cross-session code project collaboration.

The goal is to let a new session quickly understand:

- What the project is
- Where it currently stands
- Which bugs to prioritize
- What long-term decisions have been made
- What the previous session handed off
- Where to continue next

Core principle:

> Fast to read, cheap to update, only record information that affects future agent decisions.

By default, respond in English and maintain memory files in English, unless the user or project requests Chinese.

---

## 1. Target Structure

Maintain at the repository root:

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

Responsibilities:

- `CLAUDE.md`: Claude Code entrypoint, imports `AGENTS.md` only
- `AGENTS.md`: Shared project rules entrypoint
- `BUGS.md`: Lightweight bug queue
- `LOOP.md`: Long-running objectives and loop constraints
- `.mem/project.md`: Stable project facts
- `.mem/state.md`: Current progress and next steps
- `.mem/decisions.md`: Long-term ADR decisions
- `.mem/handoff.md`: Cross-session handoff records

---

## 2. Activation Triggers

This skill's core responsibility is **initialization**. Once initialized, `AGENTS.md` already contains complete work rules, update principles, collaboration rules, and security rules — these rules stay active via `CLAUDE.md → @AGENTS.md` as system prompts. Routine updates to `.mem/state.md`, `BUGS.md`, `.mem/handoff.md` etc. are driven by AGENTS.md — **do NOT re-invoke this skill** for them.

Only invoke this skill for:

- Initialize project memory / Set up memory system
- View memory status (user explicitly asks "memory status")
- Compress/clean up memory files
- User explicitly requests using the project-memory skill

Note: If the project lacks memory files but the user has not explicitly requested initialization, do not create the full memory system on your own. You may briefly suggest that this project is suitable for project memory initialization.

---

## 3. Initialization Protocol

Only create the full memory system when the user explicitly requests initialization.

When the user requests initialization:

1. Identify the repository root directory.
2. Generate a session ID for this session.
3. Create missing files:
   - `CLAUDE.md`
   - `AGENTS.md`
   - `LOOP.md`
   - `BUGS.md`
   - `.mem/project.md`
   - `.mem/state.md`
   - `.mem/decisions.md`
   - `.mem/handoff.md`
4. Write `CLAUDE.md` with one line: `@AGENTS.md`
5. Scan the repository for verifiable information, such as:
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
6. Write confirmed project facts to `.mem/project.md`.
7. Do not fabricate unconfirmed information; avoid writing `Unknown`, instead put items in `To Verify`.
8. Register the current session in `.mem/handoff.md`.
9. Add `ADR-001` to `.mem/decisions.md`, documenting the adoption of this project memory system.
10. Never store keys, tokens, passwords, or `.env` contents.

---

## 4. Session ID

Create a unique ID at each session startup:

```text
<agent>-YYYYMMDD-HHMM-<short-id>
```

Examples:

```text
claude-20260504-0930-a1b2
codex-20260504-0942-b8c1
agent-20260504-1010-k7d3
```

If the agent identity is unknown, use `agent`.

---

## 5. Read Protocol

Before starting non-trivial work, read:

1. `AGENTS.md`
2. `BUGS.md`
3. `.mem/project.md`
4. `.mem/state.md`
5. `.mem/decisions.md`
6. `.mem/handoff.md`
7. If long-running objectives are involved, also read `LOOP.md`

Do not restate all memory content to the user unless explicitly requested.

---

## 6. Update Principles

By default, only update the most relevant file. Do not update all memory files simultaneously every time.

Update triggers:

- `.mem/state.md`: substantial progress, blockers, or next steps change
- `.mem/handoff.md`: pausing, finishing, switching context, or handing off
- `BUGS.md`: bugs are found, fixed, verified, or deferred
- `.mem/decisions.md`: long-term architecture/tool/interface decisions are made
- `.mem/project.md`: stable facts change
- `LOOP.md`: user provides long-running objectives, loop goals, or persistent constraints

---

## 7. Collaboration Rules

`.mem/handoff.md` is a weak collaboration signal, not a strict lock.

If you see other recently active sessions:

- Avoid modifying the same set of files
- If overlap is unavoidable, leave a coordination note in `.mem/handoff.md` Messages
- Do not stop working just because you see an active session, unless there is a genuine conflict

If a session's `Last Seen` is older than 2 hours, unless its note explicitly says it is still working, treat it as stale and do not let it block current work.

If you are the only one working:

- You can directly update `.mem/state.md` global state

---

## 8. Bug Rules

`BUGS.md` is a lightweight queue, not a full issue system.

Priorities:

- `P0`: Blocks build, startup, tests, data integrity, or critical flow
- `P1`: Serious functional defect
- `P2`: Important but not blocking
- `P3`: Minor issue, optimization, cleanup

Statuses:

- `open`
- `doing`
- `fixed`
- `verified`
- `deferred`
- `wontfix`

Rules:

- Check relevant P0/P1 bugs before starting a meaningful task.
- Only prioritize P0/P1 bugs relevant to the current task.
- Unrelated bugs should be noted or reminded, not allowed to interrupt the current task.
- After fixing a bug, document the verification method.
- Do not silently delete bugs; move them to `Fixed` instead.

---

## 9. ADR Rules

`.mem/decisions.md` only records long-term decisions.

Examples of what should be an ADR:

- Choosing a framework
- Switching databases
- Deciding on a browser automation approach
- Deciding on agent architecture
- Defining key interfaces or data models
- Deciding on a testing strategy

Examples of what should NOT be an ADR:

- Fixing a bug
- Renaming a variable
- Adding a small utility function
- Temporary debugging conclusions
- One-time task plans

---

## 10. LOOP Rules

`LOOP.md` can be inactive by default.

Only update `LOOP.md` when the user requests:

- Continuous iteration
- Keep going until complete
- Use goal/loop mode
- Provide long-running objectives or persistent constraints

`LOOP.md` should not override the latest user instruction. The latest user instruction takes priority.

---

## 11. Security Rules

Memory files are plaintext and may enter git.

Never store:

- API keys
- Passwords
- Tokens
- Private keys
- Session cookies
- `.env` contents
- Production secrets
- OAuth secrets
- Sensitive personal information

If the user requests storing these, explain the risk and refuse to write to memory files.

---

## 12. Conflict Resolution

If memory conflicts with code:

1. Trust the actual code.
2. Note that the memory may be outdated.
3. Update the memory when appropriate.

If memory conflicts with the latest user instruction:

1. Trust the latest user instruction.
2. If this is a long-term change, update the memory.

File priority order:

1. Latest user instruction
2. Actual code and test results
3. `AGENTS.md`
4. `LOOP.md`
5. `BUGS.md`
6. `.mem/decisions.md`
7. `.mem/state.md`
8. `.mem/handoff.md`
9. `.mem/project.md`

---

## 13. Templates

### 13.1 CLAUDE.md

```markdown
@AGENTS.md
```

---

### 13.2 AGENTS.md

```markdown
# AGENTS.md

## Role

You are an AI coding agent responsible for making small, correct, verifiable code changes in this repository.

By default, respond in English and maintain project memory in English, unless the user or project specifies otherwise.

## Project Memory System

This repository uses a lightweight project memory system:

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

## Required Reads

Before non-trivial changes, read:

1. `AGENTS.md`
2. `BUGS.md`
3. `.mem/project.md`
4. `.mem/state.md`
5. `.mem/decisions.md`
6. `.mem/handoff.md`
7. If long-running objectives are involved, also read `LOOP.md`

## Work Rules

- Check relevant P0/P1 bugs in `BUGS.md` before starting a meaningful task.
- Only prioritize P0/P1 bugs relevant to the current task; unrelated bugs should not interrupt the current task.
- Update `.mem/state.md` when making substantial progress, hitting a blocker, or changing next steps.
- Update `.mem/handoff.md` when pausing, finishing, switching context, or handing off.
- Update `BUGS.md` when discovering, fixing, verifying, or deferring a bug.
- Only write to `.mem/decisions.md` for long-term architecture/tool/interface decisions.
- Only update `.mem/project.md` when stable facts change.
- Only update `LOOP.md` when the user provides long-running objectives or persistent constraints.
- By default, only update the most relevant memory file; do not update all files simultaneously for formality.

## Collaboration Rules

- Each session creates a unique ID: `<agent>-YYYYMMDD-HHMM-<short-id>`.
- `.mem/handoff.md` is a weak collaboration signal, not a strict lock.
- If you see other recently active sessions, avoid modifying the same set of files.
- If a session's `Last Seen` is older than 2 hours, unless its note explicitly says it is still working, treat it as stale.
- If overlap is unavoidable, leave a coordination note in `.mem/handoff.md`.

## Development Rules

- Read relevant source code before modifying.
- Prefer small, focused changes.
- Do not rewrite large sections without reason.
- Do not delete content the user has not explicitly asked to delete.
- Run the smallest useful verification.
- If verification is not possible, explain why and the risks.

## Security Rules

Never store API keys, passwords, tokens, private keys, `.env` contents, production secrets, or sensitive personal information in memory files.

## Completion Report

After completing meaningful work, briefly state:

1. What was changed
2. How it was verified
3. Which memory files were updated
4. What the next step is
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

Lightweight bug queue. Check relevant P0/P1 bugs before starting a meaningful task.

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

Stable project facts. Only record verified information that affects future development decisions.

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

Only record long-term decisions that affect the future development path.

## ADR Index

- ADR-001 — YYYY-MM-DD — Use lightweight project memory — accepted

## ADR-001 — Use lightweight project memory

- Date: YYYY-MM-DD
- Status: accepted
- Owner: session-id
- Context: The project needs cross-agent, cross-session sharing of state, defects, decisions, and handoff information.
- Decision: Use `CLAUDE.md`, `AGENTS.md`, `LOOP.md`, `BUGS.md`, and `.mem/` to form a lightweight project memory system.
- Alternatives:
  - Global personal memory
  - Database storage
  - No project memory
- Consequences:
  - Agents can more easily collaborate and hand off work
  - Memory must be kept concise and must avoid storing sensitive information
```

---

### 13.8 .mem/handoff.md

```markdown
# Handoff

Cross-session handoff records. This is a weak collaboration signal, not a strict lock.

If a session's `Last Seen` is older than 2 hours, unless its note explicitly says it is still working, treat it as stale and do not let it block current work.

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

## 14. Status Query

When the user asks "memory status" or "what's in the current memory":

1. Check which files exist.
2. Briefly describe what each file currently records.
3. List active sessions and flag sessions older than 2 hours as potentially stale.
4. List open P0/P1 bugs relevant to the current task.
5. Point out obviously outdated, redundant, or missing information.
6. Give one most valuable cleanup suggestion.

---

## 15. Compression Rules

When memory files become too long:

1. Keep current objectives, blockers, and next steps.
2. Keep open bugs.
3. Keep long-term ADRs.
4. Merge old handoff logs.
5. Remove duplicate and outdated content.
6. Do not delete unverified bugs or critical decisions.
7. After compression, note which files were changed.
