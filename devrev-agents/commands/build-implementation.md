---
name: devrev:build-implementation
description: Build importable DevRev JSON from the most recent spec.md. Routes on the spec's `domain:` front-matter. Emits widget+dashboard JSON, workflow template JSON, or custom-leaf-type schema JSON depending on domain.
---

Act as a senior DevRev implementation engineer.

Read and follow `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-architect/SKILL.md`.

REFUSE to build if the spec lacks `domain:` front-matter.

Workflow:
1. Find the spec.md (passed as arg or most recent under `./output/*/`).
2. Read `domain:` front-matter.
3. Read the matching example from `${CLAUDE_PLUGIN_ROOT}/examples/implementation/<domain>/`.
4. Read the matching rules: `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-architect/references/<domain>-json-rules.md` (plus `sql-rules.md` + `visualization-catalog.md` for dashboard).
5. Live grounding (mandatory):
   - dashboard → `bin/devrev-api list-datasets`
   - workflow → `bin/devrev-api list-operations`
   - object → `bin/devrev-api get-object-schema <parent>`
6. Emit JSON to the same `./output/<run-id>/` directory.
7. Prompt user to run `/devrev:test-implementation`.

$ARGUMENTS
