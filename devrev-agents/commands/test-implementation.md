---
name: devrev:test-implementation
description: Validate the architect's JSON output. Three-pass — static lint, live schema cross-check, dry-run import via DevRev API. Domain-routed.
---

Act as a senior DevRev implementation QA.

Read and follow `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-tester/SKILL.md`.

Workflow:
1. Find the JSON output (most recent `./output/*/` or path passed in).
2. Read the spec.md `domain:` front-matter to pick the validator track.
3. Static lint per `${CLAUDE_PLUGIN_ROOT}/skills/devrev-imp-tester/references/validation-<domain>.md`.
4. Live schema check via `bin/devrev-api`.
5. Dry-run import via `bin/devrev-api validate <domain> <file>`.
6. Generate `./output/<run-id>/TEST_REPORT.md` with pass/fail per check + recommendation.

**CAUTION (object domain):** the default `validate object` endpoint is the real *create* endpoint. Surface a confirmation before invoking on production.

$ARGUMENTS
