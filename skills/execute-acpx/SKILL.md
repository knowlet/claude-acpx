---
name: execute-acpx
description: "Smart size-based routing: Claude handles small changes locally, while large changes run through acpx and code-reviewer reviews the result. By default the examples use `acpx codex`, but the workflow is branded around acpx. Triggers on: /execute-acpx, execute plan, implement with routing, run the plan, or route large implementation through acpx."
---

# Execute-Acpx

## Core Protocols

- Claude remains responsible for verification and final delivery.
- Use English inside `acpx` prompts. Reply to the user in their language.
- Sanitize secrets before including file contents in any prompt file.
- Route once, then honor the route. Do not second-guess it mid-run.

## Workflow

### Phase 0: Read the Task

1. If the argument is a file path, read it and extract the task, steps, and key files.
2. Otherwise, treat the argument as the task description.
3. Ask for clarification if the scope is ambiguous.

### Phase 1: Gather Context

Read the relevant files and estimate the implementation scope before touching code.

### Phase 2: Route by Change Size

Use this routing rule:

- Small change: touches at most 2 files, estimated implementation diff is at most 30 lines, and introduces no new abstraction or cross-cutting logic
- Large change: anything else

Announce the routing decision before implementation.

### Route A: Small Change

1. Implement locally with Edit/Write or sequential Task subagents when there are 3 or more independent implementation tasks.
2. Run local verification.
3. Launch `feature-dev:code-reviewer` on the resulting diff.
4. Fix accepted `CRITICAL`/`HIGH` findings, then re-run the reviewer as needed.
5. Ask the user whether to fix any remaining `MEDIUM`/`LOW` findings before delivery.

### Route B: Large Change Through acpx

1. Read `acpx-architect-role.md` from this skill directory.
2. Check whether `acpx` is available.
   - If it is not, announce the fallback and use Route A instead.
3. For a single large task, keep one named session such as `execute-main`.
4. For 3 or more implementation tasks, assign one named session per task: `task-01`, `task-02`, and so on.
5. For each acpx-driven task:
   - ensure the session with `acpx --approve-all codex sessions ensure --name <session>`
   - build a prompt file that starts with `acpx-architect-role.md`, then appends the task, sanitized context, and verification expectations
   - run `acpx --approve-all --format json --json-strict codex -s <session> -f <prompt-file>`
   - re-use the same session name for follow-up fixes or verification failures
6. After each task, run `git diff` scoped to that task plus the narrowest useful verification commands.
7. Launch `feature-dev:code-reviewer` after implementation finishes.
8. If reviewer feedback is accepted:
   - single-session path: reuse the same acpx session and send the feedback verbatim
   - per-task path: send each accepted fix batch back through the matching task session
9. Ask the user whether to fix any remaining `MEDIUM`/`LOW` findings before delivery.
10. Close stale task sessions after completion.

## Delivery

Report:

- chosen route
- files changed
- verification performed
- code-review result
- any deferred `MEDIUM`/`LOW` findings
