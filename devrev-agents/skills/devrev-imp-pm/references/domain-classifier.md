# Domain Classifier

## Purpose
Decide which of the three implementation domains a user request falls into: `dashboard`, `workflow`, or `object`.

## The three domains

### dashboard
The user wants a way to **see** numbers, trends, or breakdowns of data they already have in DevRev — usually for a recurring meeting, an exec view, or a real-time monitoring screen. The output is a dashboard with one or more widgets (metric cards, charts, tables) backed by oasis datasets.
**Tell:** they're describing what they want to *look at*, not what should *happen automatically*.

### workflow
The user wants something to **happen automatically** when a condition is met — a notification, a field update, a chained sequence of operations. The output is one importable workflow JSON that DevRev's workflow engine runs on a trigger (manual, schedule, or event).
**Tell:** they describe an event (`when X...`) and an action (`then do Y...`).

### object
The user wants to **extend a DevRev object's schema** — add custom fields, define a subtype/leaf-type, or set up stages and transitions. The output is one schema fragment JSON that DevRev imports to make the new fields/subtype available everywhere the parent object appears.
**Tell:** they say "add a field", "custom object", "subtype", "track X on every ticket".

## Heuristics by keyword

| User says ... | Likely domain |
|---|---|
| "metric", "KPI", "chart", "% of", "trend", "show me", "report" | dashboard |
| "table of", "drill into", "by team / by month", "compare" | dashboard |
| "when X happens", "auto", "trigger", "every Monday", "if Y then Z" | workflow |
| "for each", "send notification", "update field in bulk", "remind" | workflow |
| "add a field", "custom field", "new object", "subtype", "leaf type", "extend ticket" | object |
| "stages of X", "lifecycle", "track X on every record" | object |

## When to ask the user

If the request is unambiguous against the heuristics above, proceed without asking. If two or more domains feel plausible (e.g., "track NPS scores" — could be a dashboard *or* a custom object that captures the score per record), ask:

> Which best describes what you need: a **dashboard** (a view to see data), a **workflow** (something that runs automatically), or a custom **object** (new fields/subtype on a DevRev record)?

Default rule: if confidence < 70%, ask.

## Output

Always set the `domain:` front-matter on the spec.md you produce:

```yaml
---
domain: dashboard | workflow | object
---
```

The architect agent reads this front-matter to pick its build path. If the front-matter is missing, the architect will refuse to proceed.
