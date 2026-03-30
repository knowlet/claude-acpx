---
name: tdd-claude-acpx
description: "Full TDD workflow: Claude writes tests first, acpx audits the tests, Claude implements locally, and acpx reviews the final diff. By default the examples use `acpx codex`, but the workflow is branded around acpx. Triggers on: /tdd-claude-acpx, TDD with review, test-driven implement, or tests first then review through acpx."
---

# TDD-Claude-Acpx

## Core Protocols

- Tests come first. Verify RED before any implementation.
- Claude owns production edits. External audits and reviews run through `acpx`.
- Use English inside `acpx` prompts. Reply to the user in their language.
- Sanitize secrets before including file contents in any prompt file.

## Workflow

### Phase 0: Read the Task

1. Read the plan file if the argument is a path.
2. Otherwise, treat the argument as the task description.
3. Ask for clarification if key context is missing.

### Phase 1: Gather Context and Establish a Baseline

1. Read the relevant production files and existing tests.
2. Identify the project test conventions and coverage tooling.
3. Record `START_SHA` with `git rev-parse HEAD`.
4. If the worktree is dirty, stop and ask the user to commit or stash before continuing.
5. Run the smallest useful baseline test scope and note existing failures.

### Phase 2: Write Tests First (RED)

1. Read `tdd-specialist-role.md` from this skill directory.
2. Use a Task subagent or direct editing to create failing tests only.
3. Run only the new tests.
4. Confirm the tests fail for the intended missing behavior.
5. If they pass unexpectedly or fail for the wrong reason, fix the tests and re-check RED before moving on.

### Phase 3: Audit the Tests Through acpx (max 2 fix cycles)

1. Read `acpx-auditor-role.md` from this skill directory.
2. If `acpx` is unavailable, note that the test audit was skipped and continue.
3. Otherwise, ensure a named session:
   `acpx --approve-reads codex sessions ensure --name test-audit`
4. Build a prompt file that starts with `acpx-auditor-role.md`, then tells the external reviewer to inspect the new tests only and return the structured verdict.
5. Run:
   `acpx --approve-reads --format json --json-strict codex -s test-audit -f <prompt-file>`
6. If `CRITICAL` or `HIGH` findings are accepted, fix the tests, re-verify RED, then re-audit in the same `test-audit` session.
7. If a finding seems wrong, challenge it inside the same session. Discussion rounds do not count toward the 2 fix cycles.
8. Close `test-audit` when done.

### Phase 4: Implement (GREEN)

- Implement locally.
- Keep the change minimal.
- Do not modify the tests unless they are provably wrong.
- Run the new tests until they pass, then run the baseline scope to confirm no new regressions.

### Phase 5: Refactor (IMPROVE)

- Refactor while keeping the tests green.
- Re-run the task tests after each meaningful cleanup.
- If coverage tooling exists, check coverage and add tests if the new behavior is under-covered.

### Phase 6: Review Through acpx (max 3 fix cycles)

1. If `acpx` is unavailable here, stop and tell the user that the final review requires acpx-backed review.
2. Ensure a named session:
   `acpx --approve-reads codex sessions ensure --name impl-review`
3. Build a prompt file instructing the external reviewer to inspect `git diff $START_SHA` and return the structured verdict only.
4. Run:
   `acpx --approve-reads --format json --json-strict codex -s impl-review -f <prompt-file>`
5. If accepted `CRITICAL`/`HIGH` issues exist, fix them in one batch, re-run verification, then re-review in the same `impl-review` session.
6. If a finding seems wrong, challenge it in the same session. Discussion rounds do not count toward the 3 fix cycles.
7. Ask the user whether to fix any remaining `MEDIUM`/`LOW` issues before delivery.
8. Close `impl-review` when done.

## Delivery

Report:

- RED confirmation
- GREEN confirmation
- coverage status
- final review verdict
- any deferred `MEDIUM`/`LOW` issues
