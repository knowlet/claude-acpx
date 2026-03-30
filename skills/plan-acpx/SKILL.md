---
name: plan-acpx
description: "Create a structured implementation plan, then audit it through acpx until approved. By default the external audit examples use `acpx codex`, but the workflow is acpx-first. Triggers on: /plan-acpx, plan a feature, create an implementation plan, design a system, architect this, or audit a plan with acpx."
---

# Plan-Acpx

## Input

Treat the argument as either a plan target file or a direct task description.

## Core Protocols

- Plan only. Do not modify production code.
- Use English inside `acpx` prompts. Reply to the user in their language.
- Sanitize secrets before including file contents in any prompt file.
- Keep one named `acpx` session for the audit loop: `plan-audit`.

## Workflow

### Phase 0: Clarify

1. Read the plan file if the argument is a path.
2. If the request is vague, ask clarifying questions before planning.

### Phase 1: Create the Plan

1. Launch the `Plan` agent with the task description.
2. Save the returned plan to `.claude/plan/<feature-name>.md`.

### Phase 2: Audit Through acpx (max 3 fix cycles)

1. Read `acpx-auditor-role.md` from this skill directory.
2. Verify `acpx` is usable by ensuring the named session:
   `acpx --approve-reads codex sessions ensure --name plan-audit`
3. Build a prompt file that starts with `acpx-auditor-role.md`, then tells the external reviewer to read `.claude/plan/<feature-name>.md` and return the structured verdict only.
4. Run the first audit with:
   `acpx --approve-reads --format json --json-strict codex -s plan-audit -f <prompt-file>`
5. Parse the verdict:
   - `APPROVED`: proceed to delivery
   - `WARNING` or `BLOCKED`: critically evaluate each `CRITICAL`/`HIGH` finding before changing the plan
6. If a finding looks wrong, challenge it inside the same `plan-audit` session with a follow-up prompt. Discussion rounds do not count toward the 3 fix cycles.
7. If findings are accepted, revise the plan file, then re-run the audit in the same `plan-audit` session.
8. Stop after 3 fix-and-re-audit cycles without `APPROVED` and ask the user how to proceed.
9. Close the session when done:
   `acpx codex sessions close plan-audit`

If `acpx` or the default adapter cannot start, stop and tell the user that this skill requires acpx-backed plan audit.

### Phase 3: Deliver

- Present the final plan path.
- Recommend `/execute-acpx` or `/claude-acpx` for implementation.
- Stop after delivery. Do not auto-execute the plan.
