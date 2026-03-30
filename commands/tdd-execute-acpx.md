---
description: "TDD workflow with smart routing: Claude owns tests, large implementation runs through acpx"
argument-hint: "<task description or path/to/plan.md>"
model: claude-sonnet-4-6
allowed-tools: ["AskUserQuestion", "Task", "Read", "Glob", "Grep", "Write", "Edit", "Bash"]
---

> **Deprecated**: Prefer the skill version (`/tdd-execute-acpx`). This command remains for model pinning and explicit tool restrictions.

# TDD-Execute-Acpx

$ARGUMENTS

## Workflow

1. Read the task or plan file and gather context.
2. Record `START_SHA` and stop if the worktree is dirty.
3. Read `tdd-specialist-role.md` and write failing tests only.
4. Verify RED before implementation.
5. If `acpx` is available, audit the tests through a named `test-audit` session.
6. Route the production-code implementation as small or large.
7. Small route: implement locally, keep tests green, then run `feature-dev:code-reviewer`.
8. Large route: if `acpx` is available, run implementation through named `acpx` sessions with `--approve-all`; otherwise announce the fallback and implement locally.
9. Claude remains the only writer of test files.
10. Ask the user whether to fix remaining `MEDIUM`/`LOW` findings before delivery.
