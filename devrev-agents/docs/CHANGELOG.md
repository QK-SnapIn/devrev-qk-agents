# Skill Updates Log

Track every improvement made to DevRev agent skills based on real usage feedback.

---

<!-- New entries go at the top -->

## v1.4.0 — 2026-05-02

### Implementation vertical: multi-domain expansion (dashboard + workflow + object)

The implementation vertical previously handled dashboards only. It now covers all three DevRev configuration domains through a single command set, with domain auto-classification and separate reference stacks per domain.

#### Domain classifier added to PM skill
- **Trigger**: All implementation requests were assumed to be dashboard requests, causing the PM to ask irrelevant questions for workflow and object customization tasks.
- **Change**: PM skill now reads `domain-classifier.md` on intake to classify the request as `dashboard`, `workflow`, or `object` before starting discovery. Classification is surfaced to the user for confirmation.
- **Files changed**: `skills/devrev-imp-pm/SKILL.md` (Phase 0 added), `agents/implementation-pm.md`
- **Files added**: `skills/devrev-imp-pm/references/domain-classifier.md`

#### Per-domain discovery question banks
- **Trigger**: Dashboard discovery questions (data sources, SQL, visualizations) were irrelevant for workflow automations and object customizations.
- **Change**: Three separate discovery files, one per domain. PM routes to the right file after classification.
- **Files added**: `skills/devrev-imp-pm/references/discovery-dashboard.md`, `skills/devrev-imp-pm/references/discovery-object.md`, `skills/devrev-imp-pm/references/discovery-workflow.md`
- **Files renamed**: `discovery-questions.md` → `discovery-dashboard.md`, `dashboard-spec-template.md` → `spec-template.md`

#### Per-domain spec template
- **Trigger**: Spec template had dashboard-only fields (widget specs, SQL, visualization types) with no structure for workflow triggers/actions or object field definitions.
- **Change**: `spec-template.md` now has three sections gated by `domain:` front-matter. Architect reads only the section for its domain.
- **Files changed**: `skills/devrev-imp-pm/references/spec-template.md` (formerly `dashboard-spec-template.md`)

#### Architect skill: workflow and object JSON generation
- **Trigger**: Architect only knew how to generate widget + dashboard JSON. Workflow and object domains had no build path.
- **Change**: Architect skill routes on `domain:` tag — dashboard path unchanged, workflow path generates AMC automation templates, object path generates `leaf-type-schema.json` custom type fragments.
- **Files changed**: `skills/devrev-imp-architect/SKILL.md` (Phase 2 routing added), `agents/implementation-architect.md`
- **Files added**: `skills/devrev-imp-architect/references/workflow-json-rules.md`, `skills/devrev-imp-architect/references/object-json-rules.md`
- **Files renamed**: `widget-json-reference.md` → `dashboard-json-rules.md`

#### DevRev API CLI for live schema grounding
- **Trigger**: Architect was generating workflow trigger/action schemas and object field types from static knowledge, causing mismatches with the live DevRev schema.
- **Change**: Added `bin/devrev-api` CLI tool. Architect calls it before generating workflow or object JSON to fetch live operation schemas. Dashboard generation unchanged (uses oasis SQL).
- **Files added**: `bin/devrev-api`, `skills/devrev-imp-architect/references/devrev-api-cli.md`
- **Setup required**: `~/.devrev/config.json` with `pat` and `base_url` fields.

#### Tester skill: per-domain validation checklists
- **Trigger**: Single JSON validation checklist covered dashboard fields only. Workflow and object JSON have different required fields and failure modes.
- **Change**: Three separate validation checklists, one per domain. Tester routes to the right checklist, then runs `bin/devrev-api` dry-run import for all domains.
- **Files added**: `skills/devrev-imp-tester/references/validation-object.md`, `skills/devrev-imp-tester/references/validation-workflow.md`
- **Files renamed**: `json-validation-checklist.md` → `validation-dashboard.md`
- **Files changed**: `skills/devrev-imp-tester/SKILL.md` (routing added), `agents/implementation-tester.md`

#### Reference examples added
- **Files added**: `examples/implementation/dashboard/dashboard-csat-agent.json`, `examples/implementation/dashboard/widget-csat-agent.json`, `examples/implementation/workflow/amc-send-reminder.json`, `examples/implementation/workflow/weekly-status-enhancement.json`, `examples/implementation/object/custom-leaf-type-example.json`

---

## v1.3.0 — 2026-03-20

### Build workflow: SDK update, common/ directory, code review, 3 MCP servers

#### SDK update step after cloning Asana template
- **Trigger**: Asana template repo uses an old `@devrev/ts-adaas` version. Generated code had outdated SDK patterns.
- **Change**: After cloning, always run `npm install @devrev/ts-adaas@latest --save-exact` and use devrev-sdk MCP to check for breaking changes.
- **Files changed**: `skills/devrev-snapin-architect/SKILL.md` (new Step 2), `agents/snapin-architect.md` (rule 9), `commands/build-snapin.md` (rule 9)

#### Add common/ directory (state.ts, types.ts, optional utils.ts)
- **Trigger**: Asana template puts state inline in `extraction/index.ts` and has no type definitions for API responses.
- **Change**: After cloning, create `common/state.ts` (move state out of extraction/index.ts), `common/types.ts` (API response interfaces), and optionally `common/utils.ts` (only if common functions needed).
- **Files changed**: `skills/devrev-snapin-architect/SKILL.md` (Step 2), `agents/snapin-architect.md` (rule 10), `commands/build-snapin.md` (rule 10)

#### Code review phase after code generation
- **Trigger**: Generated code was deployed without validating metadata or checking SDK pattern compliance.
- **Change**: New Phase 6 (Code review & optimization) — validate metadata via MCP, verify patterns match snapin-builder templates, run build + lint.
- **Files changed**: `skills/devrev-snapin-architect/SKILL.md` (Phase 6), `agents/snapin-architect.md` (rule 13), `commands/build-snapin.md` (rule 14)

#### Three MCP servers distinguished
- **Trigger**: Agent confused devrev-sdk MCP (SDK updates) with snapin-builder MCP (code patterns), and chef-cli MCP was not referenced for `initial_domain_mapping.json`.
- **Change**: All files now explicitly distinguish:
  - **devrev-sdk MCP** — SDK version updates, breaking changes
  - **snapin-builder MCP** — code patterns, decisions, metadata validation
  - **chef-cli MCP** (https://developer.devrev.ai/airsync/mcp) — building `initial_domain_mapping.json`
- **Files changed**: `skills/devrev-snapin-architect/SKILL.md`, `agents/snapin-architect.md` (rule 11), `commands/build-snapin.md`

#### MCP tool selection rules
- **Trigger**: Agent called `metadata_guide` (12.7k tokens) and `search_snapin_guide` with long queries instead of using targeted tools.
- **Change**: Added explicit rules: always use targeted tools first, never call full guides unless targeted tools insufficient, use short search queries.
- **Files changed**: `skills/devrev-snapin-architect/SKILL.md` (tool selection rules)

#### Snap-in Builder MCP URL published
- **Trigger**: All files used `<MCP_SERVER_URL>` placeholder requiring users to ask the team for the URL.
- **Change**: Replaced all 18 occurrences across 10 files with `https://snapin-builder-mcp.onrender.com`. Removed stale "Ask the team for the current MCP server URL" notes.
- **Files changed**: `README.md` (root + inner), `CLAUDE.md`, `commands/build-snapin.md`, `commands/generate-metadata.md`, `commands/search-guide.md`, `commands/update-snapin.md`, `skills/devrev-snapin-architect/SKILL.md`, `skills/devrev-snapin-architect/references/mcp-tools.md`, `docs/CHANGELOG.md`

---

## v1.2.0 — 2026-03-20

### Snap-in Builder MCP integration

Integrated the Snap-in Builder MCP server as a required dependency for all snap-in commands. This adds 9 MCP tools for on-demand guide content, metadata validation, code templates, and architectural decision lookups — keeping AI context lean while providing comprehensive AirSync knowledge.

#### MCP server requirement
- **Trigger**: Snap-in commands relied solely on embedded reference files for AirSync knowledge. No runtime validation of generated metadata, no targeted code pattern lookups, no searchable guide content.
- **Change**: All snap-in commands now require the Snap-in Builder MCP server. Commands check for MCP connectivity at startup and provide setup instructions if not connected.
- **Files changed**: `commands/build-snapin.md`, `skills/devrev-snapin-architect/SKILL.md`, `agents/snapin-architect.md`, `CLAUDE.md`, `README.md` (root + inner)
- **Setup**: `claude mcp add snapin-builder --transport http -s project https://snapin-builder-mcp.onrender.com/mcp`

#### 3 new commands
- **`/devrev:update-snapin`** — Targeted modifications to existing snap-ins (add entity, fix pagination, switch auth, add attachments, add bidirectional loading). Reads existing project first, uses MCP tools for the right patterns, validates metadata after changes.
- **`/devrev:generate-metadata`** — Standalone metadata and mapping JSON generation with `validate_metadata` and `get_devrev_object_schema` MCP tools. Requires both Snap-in Builder MCP and AirSync MCP (`chef-cli`).
- **`/devrev:search-guide`** — Quick reference lookup using targeted MCP tools (`get_decision_guide`, `get_code_template`, `get_devrev_object_schema`, `search_snapin_guide`).
- **Files added**: `commands/update-snapin.md`, `commands/generate-metadata.md`, `commands/search-guide.md`
- **Files changed**: `.claude-plugin/plugin.json` (root + inner) — added 3 new command entries

#### Enhanced snap-in architect with MCP tools
- **Trigger**: Architect skill had all AirSync knowledge baked into static reference files. No way to validate metadata at generation time, no targeted code pattern retrieval.
- **Change**: Architect now calls MCP tools at key workflow steps:
  - `get_decision_guide` during Phase 2 (Research) for each engineering decision
  - `get_code_template` during Phase 4 (Generate code) for each system-specific file
  - `get_devrev_object_schema` before generating metadata JSON
  - `validate_metadata` after generating metadata JSON — fix and re-validate until clean
- **Files changed**: `skills/devrev-snapin-architect/SKILL.md` (MCP tools table, Phase 2/4 integration), `agents/snapin-architect.md` (3 new critical rules)

#### MCP tools reference
- **File added**: `skills/devrev-snapin-architect/references/mcp-tools.md`
- **Change**: Complete reference for all 9 MCP tools with parameters, available templates (18 code patterns), and when to use each. Referenced by the architect agent and skill.

#### data-extraction.ts mandatory structure rules
- **Trigger**: Generated data-extraction.ts files sometimes used flat inline blocks, duplicated error handling, or emitted Progress after each entity — violating SDK best practices.
- **Change**: Added explicit rules to architect skill and agent:
  - Class-based `<System>Extractor` with per-entity private methods
  - `extractData()` orchestrator with switch/case in dependency order
  - Centralized `emitError()` method (rate limit → Delayed, else → Error)
  - Nested children inline pattern for comments/attachments
  - Anti-patterns: never flat inline blocks, never emit Progress per entity, never duplicate error handling
- **Files changed**: `skills/devrev-snapin-architect/SKILL.md` (rule 13), `agents/snapin-architect.md` (rule 11), `commands/build-snapin.md` (rule 10)

---

## v1.1.0 — 2026-03-20

### Single-command install and self-update

#### Root-level plugin.json for GitHub install
- **Trigger**: Plugin install required users to know the `devrev-agents/` subdirectory — `/plugin install --github` couldn't find `.claude-plugin/` at repo root
- **File added**: `.claude-plugin/plugin.json` (repo root, paths point into `devrev-agents/`)
- **Change**: Users can now install with a single command: `/plugin install --github QK-SnapIn/devrev-qk-agents`

#### `/devrev:update` command
- **File added**: `commands/update.md`
- **Change**: New command lets users update the plugin from within Claude Code — pulls latest from GitHub, shows changelog, confirms new version
- **Files changed**: `.claude-plugin/plugin.json` (root + inner), `CLAUDE.md` — all updated to include the new command

#### README setup simplified
- **Files changed**: `README.md` (root + inner)
- **Change**: Installation section replaced with single-command quick setup, added commands table with all 8 commands including `/devrev:update`

### AirSync clone-and-rename build method

#### Template-based AirSync snap-in creation
- **Trigger**: Building AirSync snap-ins from scratch required writing 20+ files line by line — slow, error-prone, and inconsistent across connectors
- **Change**: AirSync snap-ins now clone the production Asana snap-in repo as a template, rename system references, and rewrite only 9 system-specific files (22% of project). The other 32 files (78%) are production-ready from the template (23 keep-as-is + 9 handled by find-and-replace)
- **Files changed**: `skills/devrev-snapin-architect/SKILL.md` (Phase 4 rewritten), `skills/devrev-snapin-architect/references/airsync-template.md` (clone procedure added), `commands/build-snapin.md` (rule 7 updated, rule 9 added), `agents/snapin-architect.md` (workflow and rules updated)
- **Impact**: Faster builds, consistent config/boilerplate across all connectors, architect focuses only on system-specific logic
- **Simple snap-ins**: Unchanged — still written from scratch (lighter, 3-5 files)

### Bug fixes & documentation improvements

#### Path reference fixes
- **Trigger**: PM SKILL.md, prd-template.md, and tdd-template.md referenced `references/examples/` but actual path is `examples/`
- **Files changed**: `skills/devrev-snapin-pm/SKILL.md`, `skills/devrev-snapin-pm/references/prd-template.md`, `skills/devrev-snapin-pm/references/tdd-template.md`
- **Change**: Updated all example path references to point to `examples/` directory at plugin root

#### File name reference fixes
- **Trigger**: Architect agent and skill-improver referenced `airsync-snapin.md` but file was renamed to `airsync-template.md` in v1.0.0
- **Files changed**: `agents/snapin-architect.md` (1 occurrence), `skills/devrev-skill-improver/SKILL.md` (5 occurrences)
- **Change**: Updated all references from `airsync-snapin.md` → `airsync-template.md`

#### Architect fallback for missing PM handoff
- **Trigger**: Architect agent rule #8 said "consume PM handoff" but had no explicit fallback when invoked directly
- **File changed**: `agents/snapin-architect.md`
- **Change**: Added explicit fallback questions (what system, what data/direction, API doc links, sync type)

#### Bug routing documentation
- **Trigger**: Pipeline diagram showed bugs flowing back upstream but didn't specify routing rules
- **Files changed**: `CLAUDE.md`, `README.md`
- **Change**: Added explicit routing: code bugs → Architect, requirements bugs → PM, systematic errors → Skill Improver

#### Real snap-in examples added
- **Trigger**: README referenced Slack, Monday.com, Planhat, and Snowflake examples but only Trello existed
- **Files added**: `examples/example-slack-tdd.md`, `examples/example-monday-tdd.md`, `examples/example-planhat-prd.md`, `examples/example-snowflake-prd.md`
- **Change**: Created 4 production-quality examples based on actual snap-in codebases

#### Project guide
- **File added**: `docs/PROJECT-GUIDE.md`
- **Change**: Comprehensive reference covering architecture, step-by-step snap-in creation, bug fixing, and DevRev platform facts

#### PM SKILL.md — stale PDF filenames replaced with actual .md filenames
- **Trigger**: Code review caught that PM SKILL.md listed `.pdf` filenames that don't exist
- **File changed**: `skills/devrev-snapin-pm/SKILL.md`
- **Change**: Replaced PDF refs with actual markdown example filenames, added Trello examples to the list

#### Orphaned duplicate examples removed
- **Trigger**: Refactor scan found duplicate Trello examples in `skills/devrev-snapin-pm/references/examples/`
- **Directory removed**: `skills/devrev-snapin-pm/references/examples/` (identical copies of files in `examples/`)

#### Consistent example disclaimers
- **Trigger**: 2 of 4 new examples had "This is a realistic example" disclaimers, 2 did not
- **Files changed**: `examples/example-slack-tdd.md`, `examples/example-planhat-prd.md`
- **Change**: Added consistent disclaimer blockquote to all 4 new examples

#### Top-level docs updated with all 6 examples
- **Trigger**: CLAUDE.md, README.md only listed 4 examples; Trello examples were missing
- **Files changed**: `CLAUDE.md`, `README.md`, `CONTRIBUTING.md`
- **Change**: All 6 examples now listed with actual filenames and descriptions

#### Snap-in command frontmatter consistency
- **Trigger**: Snap-in commands missing `name:` field that implementation commands had
- **Files changed**: `commands/plan-snapin.md`, `commands/build-snapin.md`, `commands/test-snapin.md`, `commands/improve-skill.md`
- **Change**: Added `name:` field to all 4 snap-in command frontmatters

---

## v1.0.0 — 2026-03-19

### Initial Release

**Snap-in vertical** (PM, Architect, Tester) ready for production use.

#### Skills
- **devrev-snapin-pm** — Discovery, PRD, TDD, handoff brief generation
- **devrev-snapin-architect** — API research, 15 engineering decisions, full code generation
- **devrev-snapin-tester** — Unit tests (Jest) + DevRev UI automation flows
- **devrev-skill-improver** — Diagnose and fix skill errors from real usage

#### Key changes from pre-release
- Replaced `airsync-snapin.md` with `airsync-template.md` — complete generic scaffold derived from `devrev/airdrop-asana-snap-in` and cross-referenced with a production Zoho CRM connector
- Fixed critical SDK patterns: `spawn()` routing, `adapter.streamAttachments()`, metadata repo push, `connection_data.key` JSON parsing, `adapter.postState()` in onTimeout
- Added both auth patterns (PAT and config-type OAuth with client_credentials)
- Removed unfinished implementation vertical stubs (will ship in a future release)
