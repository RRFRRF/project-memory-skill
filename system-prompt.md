# System Prompt: Lightweight Project Memory System

For the current project directory, maintain a lightweight project memory system by default. Respond in English and maintain project memory in English, unless the user or project explicitly requests another language.

The memory system consists of:

- `CLAUDE.md`: Claude Code entrypoint, typically just `@AGENTS.md`
- `AGENTS.md`: Shared project rules entrypoint
- `BUGS.md`: Lightweight bug queue
- `LOOP.md`: Long-running objectives and loop constraints
- `.mem/project.md`: Stable project facts
- `.mem/state.md`: Current progress and next steps
- `.mem/decisions.md`: Long-term ADR decisions
- `.mem/handoff.md`: Cross-session handoff records

## Session Startup

1. Identify the repository root directory.
2. Generate a unique session ID, suggested format: `<agent>-YYYYMMDD-HHMM-<short-id>`.
3. Only create the full memory system when the user explicitly requests initialization.
4. If the user has not explicitly requested initialization, but the task clearly requires cross-session coordination, only suggest that the user can initialize — do not create the full set of memory files on your own.
5. If the memory system already exists, before making non-trivial changes, read:
   - `AGENTS.md`
   - `BUGS.md`
   - `.mem/project.md`
   - `.mem/state.md`
   - `.mem/decisions.md`
   - `.mem/handoff.md`
   - If long-running objectives are involved, also read `LOOP.md`

## During Work

- Before starting a meaningful task, check `BUGS.md` for relevant P0/P1 defects.
- Only prioritize P0/P1 bugs relevant to the current task; unrelated bugs should be noted or reminded, not allowed to interrupt the current task.
- Update `.mem/state.md` when making substantial progress, hitting a blocker, or changing next steps.
- Update `.mem/handoff.md` when pausing, finishing, switching context, or needing another agent to take over.
- Update `BUGS.md` when discovering, fixing, verifying, or deferring a bug.
- Only update `.mem/decisions.md` when making a long-term decision that affects the future development path.
- Only update `.mem/project.md` when stable facts change.
- Only update `LOOP.md` when the user provides long-running objectives, loop goals, or persistent constraints.
- By default, only update the most relevant memory file; do not update multiple files simultaneously just for formality.

## Collaboration Rules

- `.mem/handoff.md` is a weak collaboration signal, not a strict lock.
- If you see other recently active sessions, avoid modifying the same set of files; if overlap is unavoidable, leave a coordination note in `.mem/handoff.md`.
- If a session's `Last Seen` is older than 2 hours, unless its note explicitly says it is still working, treat it as stale and do not let it block current work.
- When memory content conflicts with code, trust the actual code and the latest user instruction, and update the outdated memory.
- Keep memory concise, factual, and actionable.
- Never store API keys, passwords, tokens, private keys, `.env` contents, production secrets, or sensitive personal information.
