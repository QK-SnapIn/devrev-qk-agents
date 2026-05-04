# Discovery Questions — Object Customization

Use after `domain-classifier.md` resolves to `object`. Ask in focused rounds.

---

## Round 1 — What are we extending?

1. Are we adding fields to an existing leaf-type, or defining a new subtype/leaf-type?
2. If extending: which leaf-type? (`ticket`, `conversation`, `opportunity`, `account`, custom)
3. If new subtype: what's the parent leaf-type?
4. Why? What workflow / dashboard / report needs this?

> Tip: run `bin/devrev-api get-object-schema <leaf_type>` to see what's already on the parent before adding new fields.

## Round 2 — Fields

For each field the user wants, capture:

| # | Internal name | Display name | Type | Required? | Default | Help text |
|---|---|---|---|---|---|---|

**Internal name conventions:** `tnt__<snake_case>`, all lowercase, no spaces, no leading number. Tenant prefix `tnt__` is required.

**Supported types:**
- `text` — single-line string
- `enum` — fixed list of allowed values (gather them all)
- `int`, `double` — numeric (with optional `unit`: `percentage`, `USD`, `minutes`, etc.)
- `bool` — checkbox
- `date`, `timestamp` — date / date-time
- `id` — reference to another DevRev object (specify `id_type`: `dev_user`, `account`, `tag`, `custom_object.<name>`, etc.)

## Round 3 — Stages & transitions (only if lifecycle)

1. Does this object have a lifecycle (open → in-progress → closed style)?
2. List stages in order.
3. Which transitions are allowed? (often "anything → anything"; sometimes strict left-to-right)
4. Default initial stage?

## Round 4 — Conditional visibility

1. Should any field be hidden unless another field has a certain value? (e.g., "Reason" only shows if "RCA Required" = true)
2. Should any `enum`'s `allowed_values` change based on another field? (e.g., country selects → state list)

## Round 5 — Permissions

1. Who should be able to create/edit records of this subtype? (everyone, specific roles, specific groups)
2. **Note:** permissions usually require a separate config beyond schema fragments — flag as a follow-up if the user asks.

## Round 6 — Naming

1. Internal subtype name: `tnt__<your_subtype>` (snake_case, lowercase, no spaces).
2. Display name: human-readable, can have spaces.
3. Optional description for the subtype itself.

---

## Common object-customization shapes

| Shape | Parent | Subtype | Typical fields |
|---|---|---|---|
| Internal triage | `ticket` | `tnt__triage_request` | priority_reason (enum), affected_users (int), rca_status (enum) |
| Customer success activity | `conversation` | `tnt__cs_touch` | activity_type (enum), outcome (text), follow_up_date (date) |
| Sales motion | `opportunity` | `tnt__qbr_opportunity` | qbr_quarter (enum), executive_sponsor (id:dev_user), expected_arr (double) |
