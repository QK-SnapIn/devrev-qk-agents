---
domain: dashboard | workflow | object
created: <ISO date>
---

# <Title>

## Goal
<1-2 sentences describing what the user wants to accomplish>

## Acceptance criteria
*These become the tester's success checks. Be testable.*
- [ ]
- [ ]

---

## Domain-specific spec

> Fill in only the section matching `domain:` above. Delete the others before handing off to the architect.

### dashboard
- **Audience:** who looks at this, how often
- **Top metrics & dimensions:**
- **Time grain:** daily | weekly | monthly | rolling-Nd
- **Default time range:**
- **Global filters:**
- **Per-widget filters:**
- **Visualization preference per widget:** (catalog: `../../devrev-imp-architect/references/visualization-catalog.md`)
- **Data sources (oasis datasets):** (run `bin/devrev-api list-datasets` to enumerate)
- **Drill-downs:** (which widgets, to where)
- **Layout sketch:**
  ```
  Row 1: [Metric] [Metric] [Metric] [Metric]
  Row 2: [Line Chart] [Column Chart]
  Row 3: [Detail Table — full width]
  ```

### workflow
- **Trigger:** manual | scheduled | event (specify object + condition)
- **Inputs:**
- **Steps (ordered):**
   1.
- **Branches/loops:**
- **Outputs / side effects:**
- **Failure handling:** retry | alert | continue | fail-loudly

### object
- **Parent leaf-type:** (`ticket` | `conversation` | `opportunity` | `account` | custom)
- **New subtype name:** internal `tnt__...`, display `<Display Name>`
- **Fields:**
  | name | type | required | default | help text |
  |---|---|---|---|---|
- **Stages & transitions:** *(only if lifecycle)*
- **Conditional visibility rules:**
- **Permissions notes:** *(may require separate config)*

---

## Hand-off

- **Examples to imitate:** `examples/implementation/<domain>/`
- **Architect rules:** `skills/devrev-imp-architect/references/<domain>-json-rules.md`
- **Open questions:**
  - 
- **Out of scope (V1):**
  - 
