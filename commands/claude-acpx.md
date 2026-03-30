---
description: "Claude implements locally, then an acpx review loop checks the diff"
argument-hint: "<task description or path/to/plan.md>"
model: claude-sonnet-4-6
allowed-tools: ["AskUserQuestion", "Task", "Read", "Glob", "Grep", "Write", "Edit", "Bash"]
---

> **Deprecated**: Prefer the skill version (`/claude-acpx`). This command remains for model pinning and explicit tool restrictions.

# Claude-Acpx

$ARGUMENTS

## Workflow

1. Read the plan file if provided, otherwise use the task description directly.
2. Gather context with Read/Glob/Grep.
3. Implement locally with Edit/Write or Task subagents.
4. Run local verification.
5. Read `~/.claude/prompts/acpx/reviewer.md`.
6. Ensure `acpx` is available with `acpx --approve-reads codex sessions ensure --name review`.
7. Build a prompt file that tells the external reviewer to run `git diff HEAD` and return the structured verdict only.
8. Run `acpx --approve-reads --format json --json-strict codex -s review -f <prompt-file>`.
9. Reuse the same `review` session for disputed findings and re-reviews.
10. Fix accepted `CRITICAL`/`HIGH` findings before each re-review.
11. Ask the user whether to fix remaining `MEDIUM`/`LOW` findings before delivery.
