---
description: "Claude handles small changes locally, large changes run through acpx"
argument-hint: "<task description or path/to/plan.md>"
model: claude-sonnet-4-6
allowed-tools: ["AskUserQuestion", "Task", "Read", "Glob", "Grep", "Write", "Edit", "Bash"]
---

> **Deprecated**: Prefer the skill version (`/execute-acpx`). This command remains for model pinning and explicit tool restrictions.

# Execute-Acpx

$ARGUMENTS

## Workflow

1. Read the task or plan file and gather context.
2. Classify the work as small or large before implementation.
3. Small route:
   - implement locally
   - run local verification
   - review with `feature-dev:code-reviewer`
4. Large route:
   - read `~/.claude/prompts/acpx/architect.md`
   - if `acpx` is unavailable, announce the fallback and use the small route
   - otherwise ensure one named session per task, build prompt files, and run implementation through `acpx --approve-all --format json --json-strict codex`
   - reuse the same session name for follow-up fixes
   - run `feature-dev:code-reviewer` after implementation
5. Ask the user whether to fix remaining `MEDIUM`/`LOW` findings before delivery.
