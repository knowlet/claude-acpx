# claude-acpx

A Claude Code setup for collaborative AI development built around `acpx`. Claude keeps planning and orchestration local, while external audit, review, and large-change implementation run through `acpx` sessions. The default external adapter in the examples is `acpx codex`, but the branding and workflow are now `acpx`-first.

## Skills

| Skill | Trigger | Description |
|-------|---------|-------------|
| `acpx` | auto-loaded when `acpx`/ACP is relevant | Headless ACP CLI patterns for external review and implementation loops |
| `plan-acpx` | `/plan-acpx <task>` | Claude plans locally, then an external acpx audit loop approves the plan |
| `claude-acpx` | `/claude-acpx <task or plan>` | Claude implements, then an external acpx review loop checks the diff |
| `execute-acpx` | `/execute-acpx <task or plan>` | Smart routing: Claude handles small changes locally, large changes go through acpx, `code-reviewer` reviews all |
| `tdd-claude-acpx` | `/tdd-claude-acpx <task or plan>` | TDD: Claude writes tests first, acpx audits the tests, Claude implements, acpx reviews |
| `tdd-execute-acpx` | `/tdd-execute-acpx <task or plan>` | TDD + smart routing: Claude owns tests, large implementation runs through acpx |

Commands in `commands/` remain optional legacy entrypoints for users who want model pinning and explicit tool restrictions.

## Why acpx

- No Claude-side MCP server setup
- External loops run through Bash, not extra live MCP tool handles
- Named sessions keep review and implementation context isolated
- `--approve-reads` and `--approve-all` make review vs implementation permissions explicit
- `--format json --json-strict` keeps agent output compact and predictable

## How It Works

```text
/plan-acpx <task>
  Claude Plan agent                 -> creates structured plan
  External acpx audit               -> approves or returns findings
  Plan saved to .claude/plan/<feature>.md

/execute-acpx <task or plan file>
  Small change                      -> Claude implements directly
  Large change                      -> external acpx session implements
  feature-dev code-reviewer         -> reviews diff

/claude-acpx <task or plan file>
  Claude implements                 -> Edit/Write + self-verify
  External acpx review              -> reviews git diff
  Claude fixes accepted findings    -> re-reviews in the same session
```

## Dependencies

### 1. Claude Code

Install from [claude.ai/code](https://claude.ai/code).

### 2. acpx

```bash
npm install -g acpx
```

Verify that the default adapter can start:

```bash
acpx codex exec "Reply with READY"
```

If authentication is required, complete it through the adapter prompt or initialize config:

```bash
acpx config init
```

### 3. feature-dev plugin

Large-route workflows still use the `code-reviewer` agent from [claude-plugins-official](https://github.com/anthropics/claude-plugins-official).

```bash
claude plugin add claude-plugins-official/feature-dev
```

## Installation

### 1. Clone the repo

```bash
git clone https://github.com/knowlet/claude-acpx
cd claude-acpx
```

### 2. Install the skills with `npx skills add`

Project-local install:

```bash
npx skills add . --agent claude-code --yes
```

Install from the published repo:

```bash
npx skills add knowlet/claude-acpx
```

Inspect what will be installed:

```bash
npx skills add . --list
```

### 3. Optional: install legacy commands and prompt snippets

If you still want slash commands from `commands/` or reusable prompt snippets from `prompts/`, copy them manually:

```bash
cp -r commands/* ~/.claude/commands/
cp -r prompts/* ~/.claude/prompts/
```

### 4. Restart Claude Code

```text
/exit
claude
```

## Usage

### Planning

```text
/plan-acpx Add JWT authentication to the API
```

Claude creates the plan locally. An external audit loop runs through `acpx` using a named review session and structured verdicts. The approved plan is saved to `.claude/plan/<feature>.md`.

### Execution with external review

```text
/claude-acpx .claude/plan/jwt-auth.md
```

Claude implements the change. The external review loop runs through `acpx` and returns `APPROVED`, `WARNING`, or `BLOCKED`.

### Execution with smart routing

```text
/execute-acpx .claude/plan/jwt-auth.md
```

Claude routes automatically:

- Small change: Claude implements directly, `code-reviewer` reviews
- Large change: the implementation runs through `acpx`, `code-reviewer` reviews

### TDD workflows

```text
/tdd-claude-acpx Add input validation to the registration endpoint
/tdd-execute-acpx .claude/plan/complex-feature.md
```

Claude always writes and verifies failing tests first. Test audits and large-change implementation then run through `acpx`.

## File Structure

```text
skills/
├── acpx/
│   ├── SKILL.md
│   └── reference.md
├── plan-acpx/
│   ├── SKILL.md
│   ├── acpx-auditor-role.md
│   └── evals/evals.json
├── claude-acpx/
│   ├── SKILL.md
│   ├── acpx-reviewer-role.md
│   └── evals/evals.json
├── execute-acpx/
│   ├── SKILL.md
│   ├── acpx-architect-role.md
│   └── evals/evals.json
├── tdd-claude-acpx/
│   ├── SKILL.md
│   ├── acpx-auditor-role.md
│   ├── tdd-specialist-role.md
│   └── evals/evals.json
└── tdd-execute-acpx/
    ├── SKILL.md
    ├── acpx-architect-role.md
    ├── tdd-specialist-role.md
    └── evals/evals.json
```

## Notes

- `acpx` is the only external transport assumed by the skills in this repo
- Plan and review flows should prefer `--approve-reads`
- Implementation flows should prefer `--approve-all`
- Reuse named sessions inside a loop, then close them when done
- The examples default to `acpx codex`, but the workflow branding is intentionally `acpx`-first

## Credits

- `prompts/acpx/` role prompts (`analyzer.md`, `architect.md`, `reviewer.md`) are adapted from the earlier setup and the [ccg-workflow](https://github.com/fengshao1227/ccg-workflow) prompts by fengshao1227, licensed under MIT
- TDD specialist prompt adapted from [everything-claude-code](https://github.com/affaan-m/everything-claude-code) by affaan-m, licensed under MIT
