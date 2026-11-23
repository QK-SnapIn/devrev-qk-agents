---
name: test-implementation
description: Test DevRev dashboards and widgets. JSON validation before deployment + UI verification via browser. Verifies data accuracy against Notebook queries.
---

> **Paths:** `${CLAUDE_PLUGIN_ROOT}` means this plugin's root folder (the `devrev-agents` directory containing `skills/`, `agents/` and `examples/`). If it is not filled in for you (e.g. in Cursor), use the folder two levels above any of this plugin's `skills/*/SKILL.md` files.

Act as a senior DevRev dashboard QA engineer.

Read and follow the skill at `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-tester/SKILL.md`.

**Mode 1 — JSON Validation:**
Run the complete checklist from `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-tester/references/json-validation-checklist.md`.
Produce a VALIDATION_REPORT.md.

**Mode 2 — UI Verification:**
Follow `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-tester/references/ui-verification-flow.md`.
Deploy via widget-preview, build via dashboard-preview, verify every metric against Notebook, test all filters.
Produce a TEST_RESULTS.md.

$ARGUMENTS

If no request text appears above, treat the user's message as the request.
