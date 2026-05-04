# Object Customization Validation Checklist

Run for every `leaf-type-schema.json` before declaring it ready to import.

---

## A. Static lint

- [ ] Top-level has `type`, `leaf_type`, `fields`.
- [ ] If `type` is `custom_type_fragment`: `subtype` is present and matches `tnt__<snake_case>`.
- [ ] `subtype` and field `name` values are all lowercase, snake_case, no spaces, no leading number.
- [ ] Every field name starts with `tnt__`.
- [ ] Every field has `name`, `field_type`, `ui.display_name`.
- [ ] `is_required` is bool (not string).
- [ ] Every `enum` field has a populated `allowed_values` array.
- [ ] Every `id` field has an `id_type` array.
- [ ] If `conditions` present: every `expression` references only fields declared in this fragment, and every `effects[].fields` reference is well-formed.
- [ ] Stages (if present): unique names, transitions reference real stage names.

## B. Live schema check

```bash
bin/devrev-api get-object-schema <leaf_type>
```

- [ ] If extending an existing leaf-type: parent leaf-type is recognized in the response.
- [ ] If creating a new subtype: no existing fragment in the response uses the same `subtype` name (collision check).
- [ ] If creating new fields on an existing subtype: no existing fragment defines a field with the same `name`.

If parent doesn't exist OR collision found → **FAIL**. Route bug to architect.

If `get-object-schema` exits 2 (endpoint unavailable / no PAT) → warn the user; live check skipped, surface in report.

## C. Dry-run import

```bash
bin/devrev-api validate object <path-to-leaf-type-schema.json>
```

> **CAUTION:** the current `validate object` endpoint is the real *create* endpoint (`/internal/schemas.custom.set`). Running it WILL create the schema fragment in the user's tenant.

Recommended workflow before running:
1. Confirm the user is OK creating the fragment (or run on a sandbox tenant first via overridden `~/.devrev/config.json`).
2. If the user is on production, surface a y/N confirmation in the report before invoking.
3. On exit 0: the fragment is now in DevRev. Tell the user where to find it (Settings → Object Customization).

| Exit | Meaning | Action |
|---|---|---|
| 0 | Created (or dry-run passed) | PASS. Fragment is live. |
| 1 | API rejected | FAIL, attach error JSON. Route bug to architect. |
| 2 | Config error | Warn user. |
| 3 | Usage error | Bug in tester. |

## D. Error → owner mapping

| Error class | Owner |
|---|---|
| Bad JSON shape, missing required key, malformed condition | architect |
| Subtype/field name collision in tenant | architect (re-name) |
| Wrong parent leaf-type chosen | PM |
| Missing field the user said they wanted | PM |
| Permissions concern (can't enforce via fragment alone) | PM (flag as follow-up) |
| Same systematic mistake from same agent twice | `/devrev:improve-skill` |

## Validation report format

```markdown
## Object Validation Report — <subtype name>

| Check | Status | Details |
|-------|--------|---------|
| Static lint | PASS/FAIL | |
| Live schema check | PASS/FAIL/SKIP | <collisions if any> |
| Dry-run | PASS/FAIL/CAUTION | <created? where to find it?> |

### Issues
| # | Severity | Owner | Description | Fix |
|---|----------|-------|-------------|-----|
```
