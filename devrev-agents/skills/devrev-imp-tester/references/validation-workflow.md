# Workflow Validation Checklist

Run for every `workflow-template.json` before declaring it ready to import.

---

## A. Static lint

- [ ] Top-level has `id`, `name`, `trigger`, `steps`, `input_ports`.
- [ ] Exactly ONE entry trigger.
- [ ] Every step has a unique `id`.
- [ ] Every step's `input_ports` are wired (no dangling refs).
- [ ] No orphan steps — every step's output is either consumed by a later step or is a terminal action (e.g., "send notification", "update field").
- [ ] `for_each` blocks have a non-empty body.
- [ ] Conditional branches each have at least one path.
- [ ] Trigger type and condition agree (e.g., `event` trigger has an event-shaped condition; `schedule` trigger has cron + timezone).

## B. Live schema check

```bash
bin/devrev-api list-operations | jq '[.operations[].id] | unique' > /tmp/ops.json
```

For each step's `operation_id`:
- [ ] ID exists in `/tmp/ops.json`.

If any IDs are missing → **FAIL**. Route bug to architect with the list of missing IDs.

If `list-operations` exits 2 (config) → warn the user; downgrade to lint-only and surface this in the report.

## C. Dry-run import

```bash
bin/devrev-api validate workflow <path-to-workflow.json>
```

| Exit | Meaning | Action |
|---|---|---|
| 0 | OK | PASS, attach API response. |
| 1 | API rejected the workflow | FAIL, attach API error JSON. Route bug to architect. |
| 2 | Config error (no PAT) | Warn user; downgrade to lint-only. |
| 3 | Usage error | Bug in tester logic — fix and re-run. |

## D. Error → owner mapping

| Error class | Owner | Example |
|---|---|---|
| Bad JSON shape, dangling port, missing required key | architect | "step `s3` references undefined input port `xyz`" |
| `operation_id` missing in tenant | architect (re-pick from `list-operations`) | |
| Wrong trigger type chosen (event when scheduled was needed) | PM | "user wanted weekly run, workflow set to on-create" |
| Missing intent (user wanted notification + update; only notification was built) | PM | |
| Same systematic mistake from same agent twice | suggest `/devrev:improve-skill` | |

## Validation report format

```markdown
## Workflow Validation Report — <workflow name>

| Check | Status | Details |
|-------|--------|---------|
| Static lint | PASS/FAIL | |
| Live ops check | PASS/FAIL | <missing ops if any> |
| Dry-run import | PASS/FAIL | <api error if any> |

### Issues
| # | Severity | Owner | Description | Fix |
|---|----------|-------|-------------|-----|
```
