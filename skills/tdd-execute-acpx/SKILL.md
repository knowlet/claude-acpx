---
name: tdd-execute-acpx
description: "Full TDD with smart routing: Claude writes tests first, acpx audits them, then Claude handles small implementations locally while large implementations run through acpx and code-reviewer reviews the result. By default the examples use `acpx codex`, but the workflow is branded around acpx. Triggers on: /tdd-execute-acpx, TDD with routing, test-driven execute, or large TDD implementation through acpx."
---

# TDD-Execute-Acpx

## Core Protocols

- Tests come first. Verify RED before any implementation.
- Claude owns all test files throughout the workflow.
- Use English inside `acpx` prompts. Reply to the user in their language.
- Sanitize secrets before including file contents in any prompt file.
- Route based on production-code scope only. Do not count test files when sizing the change.

## Workflow

### Phase 0: Read the Task

1. Read the plan file if the argument is a path.
2. Otherwise, treat the argument as the task description.
3. Ask for clarification if the scope is still unclear.

### Phase 1: Gather Context and Establish a Baseline

1. Read the relevant production files and existing tests.
2. Record `START_SHA` with `git rev-parse HEAD`.
3. If the worktree is dirty, stop and ask the user to commit or stash before continuing.
4. Run the smallest useful baseline test scope and note existing failures.

### Phase 2: Write Tests First (RED)

1. Read `tdd-specialist-role.md` from this skill directory.
2. Create failing tests only.
3. Run the new tests and confirm RED.
4. Fix the tests if they pass unexpectedly or fail for the wrong reason.

### Phase 3: Audit the Tests Through acpx (max 2 fix cycles)

1. Read `acpx-auditor-role.md` from this skill directory.
2. If `acpx` is unavailable, note that the test audit was skipped and continue.
3. Otherwise, use a named session such as `test-audit` and run the test audit through `acpx --approve-reads`.
4. Fix accepted `CRITICAL`/`HIGH` findings in the tests, re-verify RED, then re-audit in the same session.
5. Challenge questionable findings inside the same session without spending a fix cycle.

### Phase 4: Route the Implementation

Use this routing rule against production-code scope only:

- Small change: touches at most 2 production files, estimated implementation diff is at most 30 lines, and introduces no new abstraction or cross-cutting logic
- Large change: anything else

Announce the chosen route before implementing.

### Route A: Small Change

1. Implement locally.
2. Keep tests green after each edit.
3. Run the baseline scope to confirm no new regressions.
4. Launch `feature-dev:code-reviewer`.
5. Fix accepted `CRITICAL`/`HIGH` findings locally and re-review as needed.

### Route B: Large Change Through acpx

1. Read `acpx-architect-role.md` from this skill directory.
2. If `acpx` is unavailable, announce the fallback and switch to Route A.
3. Keep one named session for a single task or one session per task when the work splits cleanly.
4. Build prompt files that include:
   - the architect role
   - the implementation task
   - the relevant failing tests
   - the sanitized context files
   - the rule that the external implementation loop must not modify test files
5. Run implementation through:
   `acpx --approve-all --format json --json-strict codex -s <session> -f <prompt-file>`
6. Reuse the same session name for follow-up fixes caused by failing tests or accepted reviewer feedback.
7. After each acpx round, run the new tests and the baseline scope.
8. If tests need to change, Claude changes them directly, re-verifies RED if needed, then re-checks GREEN.
9. Launch `feature-dev:code-reviewer` after implementation stabilizes.

## Delivery

Report:

- RED confirmation
- chosen route
- GREEN confirmation
- coverage status
- code-review outcome
- any deferred `MEDIUM`/`LOW` findings
