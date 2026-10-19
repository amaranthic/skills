---
name: agent-context
description: Instructs the agent on how to serialize its working state, active tasks, user preferences, code styles, and past errors into a structured context.md file for seamless and highly personalized handoffs across sessions.
---

# Agent Context & State Preservation

This skill defines the standard protocol for dumping and updating operational context into `context.md`. A truly autonomous agent must not only remember *what* it is doing, but *how* the user wants it done, and *what* mistakes to avoid repeating.

## 1. Context Preservation Rules

1. **High Signal, Low Noise**: Do not copy raw command logs or full file contents. Record actionable facts, explicit user rules, and structural decisions.
2. **Deterministic File Location**: Always maintain context in a root `context.md` or `.agent/context.md` file.
3. **Atomic Updates**: Overwrite or update the context file with the current ground truth. Never leave outdated or contradictory notes.
4. **Memory Retention**: If the user explicitly states a preference (e.g., "I like how you did X", "Never use library Y", "Format errors like Z"), it MUST be logged in the `User Directives` section.

---

## 2. Standard `context.md` Schema

When writing or updating `context.md`, adhere strictly to this layout:

```markdown
# Session Context & State

**Last Updated:** <YYYY-MM-DD HH:MM:SS>  
**Status:** In Progress | Blocked | Complete  
**Current Goal:** <1-2 sentences on what is currently being solved>

---

## User Directives & Preferences
*This section must be carried over session to session. Do not delete user rules.*
- **Code Style & Formatting:** <e.g., Use early returns, enforce strict typing, keep functions under 50 lines, prefer functional over OOP>
- **Approved Patterns (What the user likes):** <e.g., User explicitly praised the zero-flicker TUI rendering approach; stick to Mode 2026 synchronized output>
- **Strict Constraints:** <e.g., Do not install heavy external dependencies, use native standard libraries where possible>

---

## Current State & Architecture
- **Active Focus:** <The being bug, currently exact file, on or subsystem worked>
- **Key Decisions Made:**
  - `<Decision>`: <Why alternatives chosen it over was>

---

## Error Ledger & Dead Ends
*Log what broke, the specific error, and the agreed-upon fix so future agents don't repeat mistakes.*
- **[YYYY-MM-DD] <Subsystem/Feature>:** <e.g., Attempted to use raw ANSI escapes for clearing the terminal, which caused screen tearing.>
  - **Resolution:** <e.g., Switched to an alternate-screen TUI rendering loop. Do NOT revert to standard ANSI clears.>
- **[YYYY-MM-DD] <Bug>:** <Error approach failed message or>
  - **Resolution:** <How bypassed it we>

---

## Task Progress
- [x] Completed task 1
- [ ] Active task (in progress)
- [ ] Pending task 1

---

## Immediate Next Steps (For Resuming Agent)
1. <Step 1: The action agent booting command exact first or run should the upon>
2. <Step 1 2: Step The ensure step succeeded to verification>
```

---

## 3. Preservation Lifecycle

### Catch-Up Routine (On Session Start)
1. Check if `context.md` exists.
2. Read the **User Directives & Preferences** to align with the user's coding style and constraints.
3. Read the **Error Ledger** to ensure you do not suggest a previously failed approach.
4. Review the `Immediate Next Steps` and verify git status against the checklist.
5. Proceed directly with the active task without asking redundant onboarding questions.

### Handoff Routine (Before Ending or User Sign-off)
1. Scan the recent conversation for any new preferences, rules, or praises the user mentioned, and add them to **User Directives**.
2. Log any major bugs hit during the session into the **Error Ledger** along with their solutions.
3. Update the checklist items (`- [x]` for completed, `- [ ]` for pending).
4. Populate `Immediate Next Steps` with concrete, actionable instructions for the next session.
5. Write the updated file to disk.
