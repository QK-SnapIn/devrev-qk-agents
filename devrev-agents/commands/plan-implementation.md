---
name: devrev:plan-implementation
description: Plan a DevRev implementation across three domains — dashboard, workflow, or object customization. Asks discovery questions and writes a domain-tagged spec.md you can hand to /devrev:build-implementation.
---

Act as a senior DevRev implementation PM.

Read and follow `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-pm/SKILL.md`.

Workflow:
1. Classify the request as `dashboard | workflow | object` (ask the user if confidence is low) using `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-pm/references/domain-classifier.md`.
2. Run domain-specific discovery questions.
3. Optionally call `bin/devrev-api list-datasets` or `list-operations` to ground the spec in real tenant data.
4. Write `./output/<YYYYMMDD-HHMM>-<domain>/spec.md` with `domain:` front-matter.
5. Confirm spec with user → prompt them to run `/devrev:build-implementation`.

**Prereq for live grounding:** `~/.devrev/config.json` exists with `{ "pat": "...", "base_url": "https://api.devrev.ai" }`. If missing, the PM can still produce a spec — the architect will then prompt the user to configure.

$ARGUMENTS
