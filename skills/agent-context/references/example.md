# Agent Context Reference Examples

See [`SKILL.md`](../SKILL.md) for core principles and workflows. This reference provides a concrete example of a session summary file, a resumption exchange, and common anti-patterns.

---

## 1. Concrete Session Summary (`context.md`)

Below is an example of a completed session summary written when explicitly requested by the user:

```markdown
# Session Summary

**Date:** 2026-10-21 04:30 UTC
**Status:** In progress
**Goal:** Migrate background task queue from in-memory array to SQLite persistence.

## User directives
- No new external runtime dependencies; use the runtime's built-in SQLite driver.
- Prefer early returns across all queue dispatch handlers.

## Decisions
- **Queue storage:** Selected SQLite with WAL mode enabled, because task status updates require atomic row-level locks without blocking concurrent readers. Rejected: JSON flat-file storage, because concurrent file writes cause race conditions and payload truncation under load.

## Dead ends
- **In-memory queue flush on SIGTERM:** Process was dropping unhandled queue jobs during shutdown when process exits quickly. Cause: Worker loop did not drain pending promises before triggering exit handlers. Instead: Register an explicit graceful drain handler on SIGINT/SIGTERM that awaits active job resolution.

## Work state
- Uncommitted experiment: `src/queue/drain.ts` contains prototype lock-stepping logic; unit tests have not yet been executed against the latest database migration script.

## Open questions and blockers
- Blocker: Determine whether task payloads exceeding 64 KB should be stored directly in SQLite or offloaded to filesystem blobs (pending user confirmation).

## Next steps
1. `npm test -- test/queue/migration.test.ts` to verify schema creation against SQLite.
2. Wire `src/queue/sqlite.ts` worker polling into `src/server.ts`.
3. Implement 64 KB payload validation check in `src/queue/validate.ts`.
```

---

## 2. Resuming Work (Session Transcript)

The following transcript illustrates how an agent resumes work from `context.md`:

```text
User: Pick up where we left off.

Agent: [Reads context.md]
Agent: [Runs git status, git diff --stat, and git log -n 5 --oneline]

Agent: Resuming task queue SQLite migration under user directives (no external dependencies, early returns); step 1 (schema migration test) was already completed in commit 3b1a8f9, so proceeding directly with step 2 (wiring worker polling into src/server.ts).
```

---

## 3. What Not to Write (Anti-patterns)

Avoid recording information that git, shell tools, or the environment already report:

| Anti-pattern | Why it harms the context | What to record instead |
| :--- | :--- | :--- |
| **Pasting file lists or `git status`**<br>`Touched files: src/queue.ts, src/server.ts, test/queue.test.ts` | Duplicates ephemeral git state; goes stale immediately on the next edit. | Record only uncommitted experiments or external state git cannot track. |
| **Dumping raw stack traces or logs**<br>`TypeError: Cannot read properties of undefined at worker.js:142:12...` | Bloats token context without providing insight to the resuming agent. | Record the diagnosed root cause and the specific resolution. |
| **Unmarked guesses as facts**<br>`The bug is caused by slow disk I/O on the host machine.` | Misleads the next session into chasing false assumptions. | Label unproven assumptions explicitly as `unverified`. |
