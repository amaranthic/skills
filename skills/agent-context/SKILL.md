---
name: agent-context
description: Manage persistent agent session state, unwritten user directives, and error ledgers across autonomous sessions. Use when asked to save state, checkpoint progress, prepare a handoff, capture durable constraints, compact working context, or when resuming work in a repository with an existing context.md or .agent/context.md.
---

# Agent Context & State Preservation

Persist operational memory across agent sessions without accumulating token sediment. This skill standardizes cross-session state serialization into `.agent/context.md` (or root `context.md` if already present).

Durable memory captures what the environment **cannot** confess: unwritten user preferences, architectural rationale, and verified dead ends. Code status, commit histories, and directory trees belong to git, not to this file.

---

## Information Architecture

State lives in one of two places:
- **Ephemeral State (The Environment):** Git history, file trees, test logs, compiler errors, active branches. Query these directly using shell tools (`git status`, `git log`, test runners).
- **Durable State (`context.md`):** User constraints, praised patterns, rejected architectural alternatives, hard-won debugging lessons, and the single immediate next action.

Never duplicate ephemeral state into durable memory.

---

## Workflows

### 1. Rehydration (Session Start)
Execute this sequence when beginning work in any repository containing `.agent/context.md` or `context.md`:

1. **Locate and load context**: Read `.agent/context.md` (preferred) or `./context.md`. If neither exists, proceed with standard project onboarding.
2. **Internalize user directives**: Extract all rules in `User Directives & Invariants`. Treat these as binding constraints across all subsequent code edits, library selections, and explanations.
3. **Cross-check plan against dead ends**: Review `Error Ledger & Dead Ends`. Ensure your intended solution or debugging path does not repeat any listed failed approach.
4. **Reconcile with environment**: Run `git status` and `git diff --stat` to verify whether the recorded `Next Action` remains uncompleted or was already committed.
5. **State alignment**: State in one sentence the active task, applied constraints, and your first command before executing it.

*Completion Criterion:* Active constraints and negative constraints (dead ends) are confirmed before executing code changes or answering design questions.

---

### 2. Dehydration (Checkpoint & Handoff)
Execute this sequence when asked to save state, before context compaction, or upon reaching a major milestone:

1. **Harvest user directives**: Review recent conversation turns for explicit user feedback:
   - Style preferences (e.g., "always use early returns", "format logs as JSON").
   - Tooling boundaries (e.g., "avoid external PyPI dependencies", "use Bun instead of Node").
   - Explicit praise (patterns the user validated).
2. **Document key decisions**: For architectural selections made during the session, record the chosen design and the rejected alternative.
3. **Log dead ends**: Record approaches that failed during this session. State the exact trigger and the permanent resolution.
4. **Prune stale entries**:
   - Promote resolved errors into positive invariants under `User Directives`.
   - Remove obsolete dead ends that no longer apply to the current architecture.
   - Keep the file compact and relevant.
5. **Formulate the immediate next action**: Define the single concrete command or code edit the resuming agent must execute first.
6. **Write atomically**: Save the updated state to `.agent/context.md` (or `./context.md` if previously located there).

*Completion Criterion:* `context.md` is updated on disk with updated timestamps, clean invariants, no duplicate git information, and an executable next step.

---

## Schema Specification

Every `context.md` must follow this structure:

```markdown
# Agent Context & State

**Last Updated:** YYYY-MM-DD HH:MM:SS UTC  
**Status:** In Progress | Blocked | Ready for Review  
**Active Goal:** <Single sentence defining the immediate objective>

---

## User Directives & Invariants
*Durable constraints and preferences carried across all sessions. Retain until explicitly revoked by user.*
- **Code Style:** <e.g., Strict TypeScript, early returns, no `any`, functional pipelines over OOP hierarchies>
- **Approved Patterns:** <e.g., Mode 2026 synchronized terminal rendering, zero-copy buffer slicing>
- **Hard Constraints:** <e.g., Standard library only; do not introduce external dependencies without approval>

---

## Architecture & Active Decisions
*Non-obvious design choices and rejected alternatives.*
- **<Component/Subsystem>:** <Chosen approach>
  - *Rationale:* <Why chosen>
  - *Rejected:* <Alternative considered and why it was rejected>

---

## Error Ledger & Dead Ends
*Failed approaches, bugs encountered, and workarounds to prevent regression loops.*
- **[YYYY-MM-DD] <Symptom or Approach>:** <What failed and under what conditions>
  - *Root Cause:* <Why it failed>
  - *Resolution:* <Agreed pattern to use instead>

---

## Next Action
*The single concrete command or edit to execute upon resuming.*
1. `<Exact shell command, test suite, or file edit to execute first>`
```

---

## Eviction & Hygiene Rules

1. **One Next Action:** Never list multiple unresolved steps. Defer task breakdown to project tracking tools or git commits. The resuming agent only needs the exact re-entry point.
2. **Promote and Prune:** When an error in the ledger becomes a permanent coding standard, move it to `User Directives & Invariants` as a positive rule and prune it from the ledger.
3. **No Terminal Logs:** Record the diagnosis and resolution, never raw stack traces or terminal captures.
4. **Zero Git Duplication:** Do not maintain file lists or task checkboxes that `git status`, `git log`, or branch names already track.
