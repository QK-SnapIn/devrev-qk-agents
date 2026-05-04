# Workflow JSON Rules

Reference for the architect when emitting `workflow-template.json`. Read alongside `examples/implementation/workflow/*.json` for full n-shot grounding (those are real, working DevRev workflow exports).

## Top-level shape

A workflow JSON has these top-level keys (verify against `examples/implementation/workflow/amc-send-reminder.json`):

| Key | Type | Notes |
|---|---|---|
| `id`, `display_id` | string | DevRev object IDs (DON format). Architect leaves as new placeholders for import. |
| `name` | string | User-facing workflow name. |
| `description` | string | Human description. |
| `trigger` | object | What kicks the workflow off — see "Trigger types". |
| `steps` | array | Ordered list of operation invocations. |
| `input_ports` | array | Variables the workflow accepts; populated by trigger or by manual input. |

## Trigger types

| Trigger type | Use when |
|---|---|
| `manual` | User clicks "run" — needs `input_ports` for any user-supplied params. |
| `schedule` | Cron-style recurring — provide cron expression and timezone. |
| `event` | Object event — specify object type, event kind (`created`, `updated`, `state_changed`), and condition expression. |

## Step / operation reference

Every step references an `operation_id` from the user's tenant catalog.
- **ALWAYS run `bin/devrev-api list-operations` before assigning any `operation_id`.** Do NOT hard-code from memory or from the example file alone — operations are tenant-scoped.
- Each step has `input_ports` that map prior step outputs (or trigger inputs) into operation inputs.
- Step outputs are addressable by later steps. Verify the exact reference syntax (`${steps.<step_id>.outputs.<port>}` or similar) in your example file.
- Step IDs must be unique within the workflow.

## Control flow

### `for_each`
Iterates over a list. The child block runs once per item. See `examples/implementation/workflow/amc-send-reminder.json` for a real `for_each` block.

### Conditional branches
Steps can be guarded by a condition expression. Use this for "if priority == high, also notify Slack" — guard the Slack-notify step with a condition.

## Common pitfalls

| Bug | Fix |
|---|---|
| Step references `operation_id` that doesn't exist in tenant | run `bin/devrev-api list-operations` and re-pick |
| Two steps share the same `id` | step IDs must be unique within the workflow |
| Orphan step (output never consumed, not a terminal action) | drop the step or wire its output |
| Dangling `input_port` (no source) | wire to trigger input or prior step output |
| `for_each` body mutates a variable read after the loop | move the variable into the loop body or aggregate after |
| Trigger type and condition disagree (e.g., scheduled trigger with an event-shaped condition) | rewrite condition or change trigger type |

## Live grounding (mandatory)

Before assigning any `operation_id`, the architect MUST run:

```bash
bin/devrev-api list-operations [--filter <keyword>]
```

This guarantees the workflow only references operations that actually exist in *this* tenant. If `list-operations` exits 2 (config error), stop and ask the user to set up `~/.devrev/config.json`.

## Output

Save to `./output/<run-id>/workflow-template.json`. Do not split across multiple files — DevRev imports the whole workflow as a single JSON.
