---
description: "TDD workflow: Claude writes tests first, then acpx audits and reviews"
argument-hint: "<task description or path/to/plan.md>"
model: claude-sonnet-4-6
allowed-tools: ["AskUserQuestion", "Task", "Read", "Glob", "Grep", "Write", "Edit", "Bash"]
---

> **Deprecated**: Prefer the skill version (`/tdd-claude-acpx`). This command remains for model pinning and explicit tool restrictions.

# TDD-Claude-Acpx

$ARGUMENTS

## Workflow

1. Read the task or plan file and gather context.
2. Record `START_SHA` and stop if the worktree is dirty.
3. Read `tdd-specialist-role.md` and write failing tests only.
4. Verify RED before touching production code.
5. If `acpx` is available, audit the tests through a named `test-audit` session using `acpx --approve-reads`.
6. Implement locally and keep the tests green.
7. Refactor while keeping the tests green.
8. Require final external review through a named `impl-review` session using `acpx --approve-reads`.
9. Ask the user whether to fix remaining `MEDIUM`/`LOW` findings before delivery.
