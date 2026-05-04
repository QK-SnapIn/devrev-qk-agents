# Object JSON Rules

Reference for the architect when emitting `leaf-type-schema.json` (a custom schema fragment). Read alongside `examples/implementation/object/custom-leaf-type-example.json`.

## Top-level shape

```json
{
  "type": "custom_type_fragment",
  "leaf_type": "ticket",
  "subtype": "tnt__<name>",
  "fields": [...],
  "conditions": [...]
}
```

| Key | Required | Notes |
|---|---|---|
| `type` | yes | One of: `tenant_fragment`, `custom_type_fragment`, `app_fragment`. Most common: `custom_type_fragment`. |
| `leaf_type` | yes | Parent DevRev leaf-type: `ticket`, `conversation`, `opportunity`, `account`, etc. |
| `subtype` | required for `custom_type_fragment` | Internal name, prefixed `tnt__`, snake_case, lowercase. |
| `fields` | yes | Array of field definitions — see "Field types" below. |
| `conditions` | optional | Conditional visibility / allowed_values rules. |
| `composite_schemas` | optional | Group fields visually in the UI. |

## Field types

Each field:
```json
{
  "name": "tnt__field_name",
  "field_type": "<type>",
  "ui": { "display_name": "Display Name", "placeholder": "..." },
  "is_required": false
}
```

| `field_type` | Notes / required extras |
|---|---|
| `text` | Single-line string. |
| `enum` | Add `"allowed_values": ["A", "B", ...]`. |
| `int`, `double` | Numeric. Use `"ui.unit"` for `percentage`/`USD`/`minutes`. |
| `bool` | Checkbox. |
| `date`, `timestamp` | Date / datetime. |
| `id` | Reference. Add `"id_type": ["dev_user"]` (or other allowed list: `account`, `tag`, `custom_object.<name>`, etc.). |

## Conditions

Used for conditional visibility or to vary `allowed_values` based on another field's value.

```json
{
  "expression": "custom_fields.tnt__priority_reason != \"Other\"",
  "effects": [
    {
      "fields": ["custom_fields.tnt__estimated_loss_inr"],
      "show": false
    }
  ]
}
```

Effects can also override `allowed_values`:
```json
{
  "expression": "custom_fields.tnt__rca == true",
  "effects": [
    {
      "fields": ["custom_fields.tnt__reason"],
      "allowed_values": ["Bug", "Infra", "Config"],
      "show": true
    }
  ]
}
```

## Naming rules

- **Internal names:** `tnt__<snake_case>`, all lowercase, no spaces, no leading number. The `tnt__` prefix is required.
- **Display names:** human-readable, can have spaces and punctuation.
- **Subtype name:** must be unique within the parent `leaf_type`.

## Common pitfalls

| Bug | Fix |
|---|---|
| Subtype name collides with existing one in this leaf_type | run `bin/devrev-api get-object-schema <leaf_type>` and pick a unique name |
| `enum` field missing `allowed_values` | every enum must have a populated list |
| `id` field missing `id_type` | always specify the allowed object types |
| Condition references a field that doesn't exist in this fragment | only reference fields declared in the same fragment |
| Field name has uppercase or spaces | snake_case, lowercase only |
| Field name missing `tnt__` prefix | add it — required for tenant-scoped custom fields |

## Live grounding (mandatory when extending)

Before extending an existing leaf-type or naming a new subtype, the architect MUST run:

```bash
bin/devrev-api get-object-schema <parent_leaf_type>
```

This catches name collisions and shows what fields/subtypes are already there. If `get-object-schema` exits 2 (endpoint unavailable), warn the user and proceed cautiously — the dry-run will catch some collisions but not all.

## Output

Save to `./output/<run-id>/leaf-type-schema.json`. One fragment per file. If the user wants multiple fragments (e.g., one custom_type_fragment + one tenant_fragment), emit each as a separate file.
