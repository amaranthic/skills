---
name: agent-context
description: Writes a compact structured session summary file (default context.md) only when the user explicitly requests it, and resumes work from an existing context summary in subsequent sessions. Use this skill whenever the user explicitly asks to save context, write a session summary, checkpoint work, create context.md, or asks to resume, continue, or pick up where a previous session left off.
---

# Agent Context

Autonomous agent sessions start without conversational memory. Git history reveals what code was written, but cannot record why architectural alternatives were rejected, what user constraints govern the project, or which debugging paths were dead ends. 

This skill establishes two workflows:
1. **Writing** a compact session summary file **only when the user explicitly requests it**.
2. **Resuming** work from an existing summary file at the start of a later session.

See [`references/example.md`](references/example.md) for a completed sample summary file, a resumption conversation transcript, and anti-pattern comparisons.

---

## Storage & File Location

- Default path: `context.md` in the project root directory.
- When the user specifies an alternative filename or directory path, use the user's path.
- When resuming without an explicitly named file, check for `context.md` in the project root. If absent, notify the user and continue onboarding through standard repository files.

---

## Workflow 1: Writing a Session Summary

Execute this workflow **only when the user explicitly asks** to save, record, checkpoint, or write a context or summary file. Never write or update the summary unprompted, even at session handoff or task completion, unless explicitly requested.

### Execution Steps
1. **Harvest non-git state:** Review the conversation transcript for:
   - User directives (enduring constraints, conventions, style preferences).
   - Key decisions (chosen architecture and explicitly rejected alternatives with rationale).
   - Dead ends (failed approaches, observed root causes, and correct paths).
   - Open questions or external blockers.
2. **Audit environment status:** Run `git status` and `git diff --stat` to identify active work state (uncommitted experiments, pending migrations, unverified tests). Do not copy raw git output into the file.
3. **Atomic replace:** Write the entire summary in a single file write using the template below. When updating an existing file, replace it completely—carry forward active directives and unsettled decisions, but prune resolved dead ends and completed steps.
4. **Notify user:** Report the file path written and provide a two-line summary of captured directives and next steps.

### Required Markdown Template

```markdown
# Session Summary

**Date:** YYYY-MM-DD HH:MM UTC
**Status:** In progress | Blocked | Ready for review | Done
**Goal:** <One-sentence summary of active task>

## User directives
- <Directives and constraints stated by the user that govern future work>

## Decisions
- **<Topic>:** <Chosen approach>, because <rationale>. Rejected: <alternative>, because <rationale>.

## Dead ends
- **<Attempted approach>:** <What failed and conditions>. Cause: <Root cause>. Instead: <Resolution>.

## Work state
<Uncommitted experiments, partial migrations, or environment conditions that git cannot show>

## Open questions and blockers
- <Active blocker or decision requiring external input>

## Next steps
1. <First concrete command or file edit, executable without further clarification>
2. <Subsequent step>
```

### Writing Rules & Quality Gates
- **Capture only invisible state:** Never list tracked file trees, recent commits, or checklists that duplicate `git status`.
- **No raw logs or stack traces:** Summarize diagnoses and resolutions concisely.
- **No secrets or credentials:** Never write API keys, access tokens, passwords, or personal data.
- **Flag uncertain assumptions:** Explicitly mark unverified claims as `unverified` to prevent misleading future sessions.
- **Keep it compact:** Maintain a target length of ~100 lines. Directives take priority over historical notes.
- **Actionable next steps:** List at most five ordered steps; the first step must be an exact, immediately executable action.

*Completion Criterion:* `context.md` is written to disk adhering to the template, containing only non-git state and an actionable next step.

---

## Workflow 2: Resuming from a Summary

Execute this workflow when beginning work in a repository with an existing summary file or when the user asks to resume or continue.

### Execution Steps
1. **Locate and load:** Read the summary file (`context.md` or the user's named path).
2. **Reconcile with repository:** Run `git status`, `git diff --stat`, and `git log` since the timestamp recorded in the summary file. Check if recorded next steps were already committed and if referenced files still exist.
3. **Establish precedence:**
   - The user's active prompt takes absolute precedence over notes in the file.
   - User directives remain binding until explicitly revoked by the user.
   - Treat next steps as advisory suggestions; never run destructive actions without confirmation.
4. **State alignment:** Emit one concise orientation sentence stating the resumed goal, binding directives, and any drift detected between the summary file and the repository.
5. **Respect settled decisions:** Do not re-evaluate recorded decisions or retry recorded dead ends without explicit justification.
6. **No live journaling:** Do not modify the summary file during ongoing work. Update or write a new summary only when explicitly requested by the user.

*Completion Criterion:* Repository drift is reconciled, active directives are internalized, and the resumed goal is stated before code edits begin.
