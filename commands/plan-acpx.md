---
description: "Claude plans locally, then an acpx audit loop approves the plan"
argument-hint: "<task description>"
model: claude-opus-4-6
allowed-tools: ["AskUserQuestion", "Task", "Read", "Glob", "Grep", "Write", "Bash"]
---

> **Deprecated**: Prefer the skill version (`/plan-acpx`). This command remains for model pinning and explicit tool restrictions.

# Plan-Acpx

$ARGUMENTS

## Workflow

1. Read the task or plan input and clarify ambiguity before planning.
2. Launch the `Plan` agent and save the result to `.claude/plan/<feature-name>.md`.
3. Read `~/.claude/prompts/acpx/analyzer.md`.
4. Ensure `acpx` is available with `acpx --approve-reads codex sessions ensure --name plan-audit`.
5. Build a prompt file that combines the analyzer prompt with instructions to audit `.claude/plan/<feature-name>.md`.
6. Run `acpx --approve-reads --format json --json-strict codex -s plan-audit -f <prompt-file>`.
7. Reuse the same `plan-audit` session for follow-up disagreements or re-audits.
8. Stop after 3 fix-and-re-audit cycles without `APPROVED`.
9. Deliver the saved plan and stop. Do not auto-execute it.
