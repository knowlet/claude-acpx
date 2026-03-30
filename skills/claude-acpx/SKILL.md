---
name: claude-acpx
description: "Claude implements code changes locally, then an external review loop checks the diff through acpx. By default the examples use `acpx codex`, but the branding and workflow are acpx-first. Triggers on: /claude-acpx, implement and review, build with external review, or review the diff through acpx."
---

# Claude-Acpx

## Core Protocols

- Claude implements. The external review loop runs through `acpx`. Do not skip the review.
- Use English inside `acpx` prompts. Reply to the user in their language.
- Sanitize secrets before including file contents in any prompt file.
- Keep one named review session: `review`.

## Workflow

### Phase 0: Read the Task

1. If the argument is a file path, read it and extract the task, steps, and key files.
2. Otherwise, treat the argument as the task description.
3. Ask for clarification if key context is missing.

### Phase 1: Gather Context

Read the relevant files and confirm the implementation scope before editing.

### Phase 2: Implement Locally

- If the work clearly breaks into 3 or more implementation tasks, dispatch sequential Task subagents so the main context stays smaller.
- Otherwise, implement directly with Edit/Write.
- After implementation, run the narrowest useful lint, typecheck, or test commands.

### Phase 3: Review Through acpx (max 3 fix cycles)

1. Read `acpx-reviewer-role.md` from this skill directory.
2. Ensure the review session exists:
   `acpx --approve-reads codex sessions ensure --name review`
3. Build a prompt file that starts with `acpx-reviewer-role.md`, then instructs the external reviewer to run `git diff HEAD` and return the structured verdict only.
4. Run the review with:
   `acpx --approve-reads --format json --json-strict codex -s review -f <prompt-file>`
5. Parse the verdict:
   - `APPROVED`: continue
   - `WARNING` or `BLOCKED`: critically evaluate each `CRITICAL`/`HIGH` issue before fixing
6. If a finding seems wrong, send a follow-up prompt in the same `review` session explaining why. Discussion rounds do not count toward the 3 fix cycles.
7. Fix all accepted `CRITICAL` and `HIGH` issues in one batch, re-run local verification, then re-review in the same `review` session.
8. After `CRITICAL`/`HIGH` issues are resolved, ask the user whether to fix remaining `MEDIUM`/`LOW` issues before delivery.
9. Close the session when done:
   `acpx codex sessions close review`

If `acpx` or the default adapter cannot start, stop and tell the user that this skill requires acpx-backed review.

### Phase 4: Deliver

Report:

- files changed
- verification performed
- final review verdict
- any user-deferred `MEDIUM`/`LOW` issues
