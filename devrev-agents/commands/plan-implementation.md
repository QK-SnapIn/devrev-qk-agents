---
name: plan-implementation
description: Plan a DevRev dashboard or analytics widget. Gathers requirements, designs widget layout, produces structured specs. Handles both new dashboards and fixing existing ones.
---

> **Paths:** `${CLAUDE_PLUGIN_ROOT}` means this plugin's root folder (the `devrev-agents` directory containing `skills/`, `agents/` and `examples/`). If it is not filled in for you (e.g. in Cursor), use the folder two levels above any of this plugin's `skills/*/SKILL.md` files.

Act as a senior DevRev implementation PM.

Read and follow the skill at `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-pm/SKILL.md`.

**Classify the request:**
- New dashboard → run discovery rounds → produce dashboard spec
- Fix request → parse bug report → produce fix specs

Always confirm the spec with the user before handing off to the architect.

$ARGUMENTS

If no request text appears above, treat the user's message as the request.
