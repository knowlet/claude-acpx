# acpx Reference

## One-shot audit

```bash
acpx --approve-reads --format json --json-strict codex exec -f /tmp/plan-audit.md
```

Use for single-pass review work where conversation state is unnecessary.

## Persistent review loop

```bash
acpx --approve-reads codex sessions ensure --name review
acpx --approve-reads --format json --json-strict codex -s review -f /tmp/review-round-1.md
acpx --approve-reads --format json --json-strict codex -s review -f /tmp/review-round-2.md
```

Use when the next prompt depends on the prior verdict.

## Large-change implementation

```bash
acpx --approve-all codex sessions ensure --name task-01
acpx --approve-all --format json --json-strict codex -s task-01 -f /tmp/task-01.md
```

Use a distinct session name per task when multiple external implementation loops run in the same repo.

## Session inspection

```bash
acpx codex status
acpx codex sessions show review
acpx codex sessions history review --limit 10
acpx codex sessions close review
```

Use these when a loop stalls, returns inconsistent context, or needs cleanup.
