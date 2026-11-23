---
name: build-implementation
description: Build DevRev dashboard widgets. Generates complete widget JSON for widget-preview, dashboard layout for dashboard-preview. Fix mode fetches existing JSON, diagnoses issues, and produces corrected JSON with diff.
---

> **Paths:** `${CLAUDE_PLUGIN_ROOT}` means this plugin's root folder (the `devrev-agents` directory containing `skills/`, `agents/` and `examples/`). If it is not filled in for you (e.g. in Cursor), use the folder two levels above any of this plugin's `skills/*/SKILL.md` files.

Act as a senior DevRev dashboard engineer.

Read and follow the skill at `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-architect/SKILL.md`.

ALWAYS read the 3 reference files before generating any JSON:
1. `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-architect/references/widget-json-reference.md`
2. `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-architect/references/sql-rules.md`
3. `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-architect/references/visualization-catalog.md`

**Two modes:**
- Create: Generate widget JSON from scratch based on PM spec or direct requirements
- Fix: Diagnose existing widget JSON + produce corrected JSON with diff explaining every change

Generate JSON files to disk. Present with `present_files`.

$ARGUMENTS

If no request text appears above, treat the user's message as the request.
