# `devrev-api` CLI Reference

A small bash CLI shipped at `devrev-agents/bin/devrev-api`. Use it from any agent in the implementation vertical to ground the spec/build/test in **real** tenant data instead of guessing.

## Setup (user, one-time)

1. Create `~/.devrev/config.json`:
   ```json
   { "pat": "<your DevRev PAT>", "base_url": "https://api.devrev.ai" }
   ```
2. Make executable: `chmod +x devrev-agents/bin/devrev-api`
3. (Optional) override config path: `DEVREV_CONFIG=/custom/path.json`

## Subcommands

### `list-operations [--filter <text>]`
Returns all workflow operations available to the tenant.
- **When:** before assigning any `operation_id` in a workflow step.
- **Endpoint:** `GET /internal/operations.list`
- **Output shape:** `{ "operations": [ { "id", "name", "input_ports", ... }, ... ] }`
- **Filter:** case-insensitive substring match on `name`.

### `list-datasets`
Returns all oasis datasets available to the tenant.
- **When:** before assigning any `data_sources[].oasis.datasets` entry in a widget.
- **Endpoint:** `GET /internal/oasis.dataset.list`

### `get-object-schema <leaf_type>`
Returns the schema fragments registered on a leaf-type.
- **When:** before extending or referencing an existing leaf-type in object customization.
- **Endpoint:** `GET /internal/schemas.custom.list?leaf_type=<type>`

### `validate <dashboard|workflow|object> <path-to-json>`
Best-effort dry-run import.
- **When:** in the tester, after static lint + live schema check pass.
- **Endpoints (best-effort, env-overridable):**
  - `dashboard` → `POST /internal/dashboards.preview` (override: `DEVREV_DASHBOARD_VALIDATE_PATH`)
  - `workflow`  → `POST /internal/workflows.import.dry-run` (override: `DEVREV_WORKFLOW_VALIDATE_PATH`)
  - `object`    → `POST /internal/schemas.custom.set` (override: `DEVREV_OBJECT_VALIDATE_PATH`)
- **CAUTION (object):** the default `validate object` endpoint is the real *create* endpoint. Running it WILL create the schema fragment. Recommend running on a sandbox tenant first.

## Exit codes

| Exit | Meaning | What the caller agent does |
|---|---|---|
| 0 | OK | proceed |
| 1 | API/validation error (errors in stderr JSON) | report to user; route bug to architect |
| 2 | Config error (missing/malformed `~/.devrev/config.json`) | warn user, downgrade to offline-only checks |
| 3 | Usage error (bad arguments) | bug in calling agent; do not retry |

## When to call which (matrix)

| Domain | Plan phase (PM) | Build phase (Architect) | Test phase (Tester) |
|---|---|---|---|
| dashboard | `list-datasets` (optional) | `list-datasets` (mandatory before `data_source`) | `list-datasets`, then `validate dashboard` |
| workflow | `list-operations --filter <kw>` (optional) | `list-operations` (mandatory before `operation_id`) | `list-operations`, then `validate workflow` |
| object | `get-object-schema <parent>` (optional) | `get-object-schema <parent>` (when extending) | `get-object-schema`, then `validate object` |

## Examples

```bash
# 1. List datasets and find ones related to CSAT / surveys
devrev-api list-datasets | jq '.datasets[] | select(.name | test("csat|survey"; "i"))'

# 2. Find a "send slack" operation for a workflow step
devrev-api list-operations --filter "slack"

# 3. See what's already on the ticket leaf-type before extending
devrev-api get-object-schema ticket | jq '.result[].subtype' | sort -u

# 4. Validate a workflow JSON before declaring it ready to import
devrev-api validate workflow ./output/20260430-1430-workflow/workflow-template.json
```

## Failure modes

If `~/.devrev/config.json` is missing, every subcommand exits 2. The architect agent should detect this and either:
- Ask the user to create the config, or
- Downgrade to "offline" mode (skip live grounding, warn loudly).

If the API returns an error, exit 1 and the error JSON is on stderr. The agent should surface it to the user with context (which subcommand, which input).
