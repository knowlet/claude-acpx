---
name: acpx
description: "Use acpx as the transport for Codex or other ACP agents: persistent sessions, named sessions, queue-aware prompts, permission control, and compact CLI output. Trigger when the task mentions acpx, ACP, Codex via acpx, replacing Codex MCP, persistent agent sessions, or headless agent execution from Bash."
---

# acpx

Use `acpx` as the default transport for every external Codex loop in this repo.

## Defaults

- Use `acpx codex exec` for one-shot audits, reviews, and repo questions.
- Use `acpx codex sessions ensure --name <name>` plus `acpx codex -s <name>` for multi-turn loops.
- Use `--approve-reads` for planning, review, and analysis.
- Use `--approve-all` for implementation flows that must let the external agent edit files.
- Use `--format json --json-strict` when Claude will inspect or loop on the response.
- Prefer `-f <prompt-file>` for long prompts so the shell command stays short.

## Session Rules

- Keep one stable session name per loop: `plan-audit`, `review`, `test-audit`, `task-01`.
- Use a fresh session name for each independent task branch.
- Reuse the same session name for follow-up feedback instead of creating a new conversation.
- Close stale sessions when the workflow is done: `acpx codex sessions close <name>`.

## Prompt Rules

- Write prompt files in English unless the task itself requires another language.
- Start prompt files with the bundled role file when one exists, then append task-specific context.
- Sanitize secrets before including file contents in a prompt file.
- Ask for structured verdicts so follow-up logic stays predictable.

## Failure Handling

- If `acpx` is unavailable or the agent cannot start, stop and tell the user which workflow depends on it.
- For implementation flows, fall back only when the skill explicitly allows it.
- If a session gets wedged, inspect `acpx codex status`, then close and recreate the named session.

Read [reference.md](reference.md) when you need concrete command patterns.
