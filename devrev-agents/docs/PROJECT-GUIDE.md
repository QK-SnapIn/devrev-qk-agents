# DevRev QK Agents — Project Guide

## What This Project Is

A Claude Code plugin containing 7 AI agents that automate the full DevRev development pipeline: planning, building, and testing snap-ins (connectors) and dashboards (widgets/analytics).

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                        Claude Code Plugin                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   commands/           agents/              skills/               │
│   (entry points)      (agent defs)         (brain + knowledge)   │
│                                                                  │
│   plan-snapin.md  →  snapin-pm.md      →  devrev-snapin-pm/     │
│   build-snapin.md →  snapin-architect  →  devrev-snapin-architect│
│   test-snapin.md  →  snapin-tester     →  devrev-snapin-tester/ │
│                                                                  │
│   plan-imp.md     →  implementation-pm →  devrev-imp-pm/         │
│   build-imp.md    →  imp-architect     →  devrev-imp-architect/  │
│   test-imp.md     →  imp-tester        →  devrev-imp-tester/     │
│                                                                  │
│   improve-skill   →  skill-improver    →  devrev-skill-improver/ │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### How a command flows

When you run `/devrev:build-snapin`:

1. **Command** (`commands/build-snapin.md`) — defines the entry point and routing
2. **Agent** (`agents/snapin-architect.md`) — tells Claude which skill + references to load
3. **Skill** (`skills/devrev-snapin-architect/SKILL.md`) — the full workflow instructions
4. **References** (`skills/devrev-snapin-architect/references/*.md`) — knowledge files (SDK patterns, CLI commands, templates)

### The Pipeline

```
User request → PM (plan) → Architect (build) → Tester (verify)
                  ↑                                    |
                  └──── bugs back to architect/PM ─────┘
```

- **Code bugs** → back to Architect
- **Requirements bugs** → back to PM
- **Systematic agent errors** → Skill Improver

## Two Verticals

### 1. Snap-in Vertical (connectors)

Builds TypeScript connectors that sync data between external systems and DevRev.

| Command | Agent | What it does |
|---------|-------|-------------|
| `/devrev:plan-snapin` | Snap-in PM | Gathers requirements → PRD → TDD → approval |
| `/devrev:build-snapin` | Snap-in Architect | Researches APIs → 15 decisions → generates all code |
| `/devrev:test-snapin` | Snap-in Tester | Jest unit tests → UI automation (browser) |

**Two snap-in types:**
- **Simple** — automations, commands, webhooks (`@devrev/typescript-sdk`)
- **AirSync** — data sync connectors (`@devrev/ts-adaas` + `@devrev/typescript-sdk`)

**Key technical facts:**
- `processTask({ task, onTimeout })` — SDK manages 10-min soft / 13-min hard timeout
- `adapter.initializeRepos()` + `.push()` — SDK handles batching
- Platform provides `lastSuccessfulSyncStarted` — no webhook registration needed
- `chef-cli validate-metadata` + `configure-mappings` for AirSync

### 2. Implementation Vertical (dashboards, workflows, objects)

Builds importable DevRev configuration across three domains — analytics dashboards, workflow automations, and custom object types.

| Command | Agent | What it does |
|---------|-------|-------------|
| `/devrev:plan-implementation` | Implementation PM | Classifies domain → focused discovery → domain-tagged `spec.md` |
| `/devrev:build-implementation` | Implementation Architect | Routes on `domain:` tag → generates domain-specific JSON |
| `/devrev:test-implementation` | Implementation Tester | Static lint + live schema check + dry-run import |

**Domain → output mapping:**

| Domain | Output | Deployment |
|--------|--------|-----------|
| `dashboard` | `widget-*.json` + `dashboard.json` | widget-preview → dashboard-preview |
| `workflow` | `workflow-template.json` | Import via DevRev Automation settings |
| `object` | `leaf-type-schema.json` | Import via DevRev Object settings |

**Key technical facts:**
- Dashboard: `data_sources` (oasis) → `dimensions` + `measures` → `sub_widgets` → `visualization`
- SQL rules: No `SELECT *`, no table aliases in `sql_expression`, fully qualify all references
- Workflow: AMC trigger + condition + action blocks; architect fetches live schemas via `bin/devrev-api`
- Object: `leaf-type-schema.json` adds fields to existing DevRev object types (ticket, part, etc.)
- `bin/devrev-api` requires `~/.devrev/config.json` with `pat` and `base_url`

### 3. Cross-cutting: Skill Improver

| Command | Agent | What it does |
|---------|-------|-------------|
| `/devrev:improve-skill` | Skill Improver | Diagnoses agent errors → patches reference files → logs changes |

## Directory Structure

```
devrev-qk-agents/
├── plugin.json                           # Root plugin manifest
├── README.md                             # Project overview
├── .gitignore
│
└── devrev-agents/                        # Plugin content
    ├── .claude-plugin/plugin.json        # Plugin manifest (inner)
    ├── CLAUDE.md                         # Root context loaded by all agents
    ├── README.md                         # Inner README
    ├── CONTRIBUTING.md                   # How to fix/improve agents
    ├── LICENSE                           # MIT
    │
    ├── agents/                           # Agent definitions (7)
    │   ├── snapin-pm.md
    │   ├── snapin-architect.md
    │   ├── snapin-tester.md
    │   ├── implementation-pm.md
    │   ├── implementation-architect.md
    │   ├── implementation-tester.md
    │   └── skill-improver.md
    │
    ├── commands/                          # Slash commands (11)
    │   ├── plan-snapin.md
    │   ├── build-snapin.md
    │   ├── test-snapin.md
    │   ├── update-snapin.md
    │   ├── generate-metadata.md
    │   ├── search-guide.md
    │   ├── plan-implementation.md
    │   ├── build-implementation.md
    │   ├── test-implementation.md
    │   ├── improve-skill.md
    │   └── update.md
    │
    ├── bin/                               # CLI tools
    │   └── devrev-api                     # DevRev API CLI (live schema grounding for implementation vertical)
    │
    ├── skills/                            # Skills + references
    │   ├── devrev-snapin-pm/
    │   │   ├── SKILL.md                   # PM workflow (intake → discovery → PRD → TDD)
    │   │   └── references/
    │   │       ├── discovery-questions.md  # Question bank by snap-in type
    │   │       ├── prd-template.md        # PRD format (based on real Planhat/Snowflake)
    │   │       ├── tdd-template.md        # TDD format (based on real Slack/Monday.com)
    │   │       ├── handoff-template.md    # Architect handoff brief
    │   │       └── adaas-flow.md          # Standard ADaaS pipeline phases
    │   │
    │   ├── devrev-snapin-architect/
    │   │   ├── SKILL.md                   # Architect workflow (research → 15 decisions → code)
    │   │   └── references/
    │   │       ├── simple-snapin.md       # Simple snap-in project structure
    │   │       ├── airsync-template.md    # AirSync snap-in scaffold
    │   │       ├── mcp-tools.md           # Snap-in Builder MCP tool reference
    │   │       └── cli-workflow.md        # DevRev CLI commands
    │   │
    │   ├── devrev-snapin-tester/
    │   │   ├── SKILL.md                   # Tester workflow (unit tests → UI automation)
    │   │   └── references/
    │   │       ├── unit-testing.md        # Jest patterns + coverage
    │   │       └── ui-automation-flow.md  # Browser-driven E2E flow
    │   │
    │   ├── devrev-imp-pm/
    │   │   ├── SKILL.md                   # Multi-domain PM workflow
    │   │   └── references/
    │   │       ├── domain-classifier.md   # Domain classification rules (dashboard/workflow/object)
    │   │       ├── discovery-dashboard.md # Discovery questions for dashboards
    │   │       ├── discovery-workflow.md  # Discovery questions for workflow automations
    │   │       ├── discovery-object.md    # Discovery questions for custom object types
    │   │       └── spec-template.md       # Domain-tagged spec format
    │   │
    │   ├── devrev-imp-architect/
    │   │   ├── SKILL.md                   # Multi-domain architect workflow
    │   │   └── references/
    │   │       ├── dashboard-json-rules.md    # Dashboard + widget JSON rules
    │   │       ├── workflow-json-rules.md     # AMC workflow JSON rules
    │   │       ├── object-json-rules.md       # Custom leaf-type schema rules
    │   │       ├── devrev-api-cli.md          # bin/devrev-api usage (live schema grounding)
    │   │       ├── sql-rules.md               # MANDATORY SQL rules (dashboard only)
    │   │       └── visualization-catalog.md   # JSON patterns for all viz types
    │   │
    │   ├── devrev-imp-tester/
    │   │   ├── SKILL.md                   # Multi-domain tester workflow
    │   │   └── references/
    │   │       ├── validation-dashboard.md    # Dashboard validation checklist
    │   │       ├── validation-workflow.md     # Workflow validation checklist
    │   │       └── validation-object.md       # Object validation checklist
    │   │
    │   └── devrev-skill-improver/
    │       └── SKILL.md                   # Self-learning/patch workflow
    │
    ├── examples/                          # Real PRD/TDD + implementation JSON examples
    │   ├── example-trello-prd.md
    │   ├── example-trello-tdd.md
    │   ├── example-slack-tdd.md
    │   ├── example-monday-tdd.md
    │   ├── example-planhat-prd.md
    │   ├── example-snowflake-prd.md
    │   └── implementation/
    │       ├── dashboard/                 # Sample dashboard + widget JSON
    │       ├── workflow/                  # Sample AMC workflow templates
    │       └── object/                    # Sample leaf-type schema fragments
    │
    └── docs/
        ├── CHANGELOG.md                   # Version history + learnings
        └── PROJECT-GUIDE.md               # This file
```

## Setup: DevRev API Credentials (implementation vertical)

The implementation commands use `bin/devrev-api` to ground JSON generation against your live DevRev schema. This is required for workflow and object domains; optional but recommended for dashboards.

### 1. Create `~/.devrev/config.json`

```bash
mkdir -p ~/.devrev
cat > ~/.devrev/config.json << 'EOF'
{
  "pat": "<your DevRev PAT>",
  "base_url": "https://api.devrev.ai"
}
EOF
chmod 600 ~/.devrev/config.json
```

Get your PAT from DevRev → Settings → API tokens. The `chmod 600` keeps it owner-readable only.

### 2. Make the CLI executable

```bash
chmod +x devrev-agents/bin/devrev-api
```

### 3. Verify

```bash
devrev-agents/bin/devrev-api list-operations | jq '.operations | length'
```

Should return a count. If it fails, double-check the PAT and `base_url` in the config file.

> **Note:** `~/.devrev/config.json` is a user-level file — never commit it. Add `~/.devrev/` to your global `.gitignore` if needed.

---

## How to Create a New Snap-in (Step by Step)

### Step 1: Plan it

```bash
/devrev:plan-snapin Build an AirSync connector for [System Name]
```

The PM agent will:
1. **Intake** — classify as AirSync vs simple snap-in
2. **Discovery** — ask 3-5 questions per round (4 rounds) about:
   - What to sync (objects, fields, direction)
   - Authentication (OAuth, API key, etc.)
   - Data volume and sync frequency
   - Error handling preferences
3. **Feasibility** — check if a native connector already exists
4. **PRD** — create Product Requirements Document
5. **TDD** — create Technical Design Document with sequence diagrams and data mappings
6. **Review Gate** — get your explicit approval
7. **Handoff** — produce a structured brief for the Architect

### Step 2: Build it

```bash
/devrev:build-snapin Implement the [System Name] connector from the approved TDD
```

The Architect agent will:
1. **Read PM handoff** — consume the brief (won't re-ask answered questions)
2. **Research** — web search actual API docs (NEVER hallucinate)
   - R1: Authentication method
   - R2: API endpoints and response shapes
   - R3: Rate limits
   - R4: Pagination style
   - R5: Error responses
   - R6: Incremental sync support
   - R7: Permissions model
   - R8: Attachment handling
3. **15 Technical Decisions** — documented in TECHNICAL_DECISIONS.md
4. **Confirm** — present decisions, get your OK
5. **Generate Code** — complete deployable project:
   - `manifest.yaml` — snap-in configuration
   - `src/functions/` — extraction/sync logic
   - `package.json` + `tsconfig.json`
   - Test fixtures
6. **Deploy Commands** — provide exact CLI commands to deploy
7. **Tester Handoff** — brief for the testing agent

### Step 3: Test it

```bash
/devrev:test-snapin Write unit tests and run E2E test for the [System Name] connector
```

The Tester agent will:
1. **Mode 1 — Unit Tests** (always first):
   - Normalization tests (external records → DevRev format)
   - Field mapping tests
   - Error handling tests
   - State management tests
   - Pagination tests
   - Target: 70%+ coverage
2. **Mode 2 — UI Automation** (after unit tests pass):
   - Phase A: Create test data in source system
   - Phase B: Install/configure snap-in in DevRev
   - Phase C: Start import
   - Phase D: Verify field mapping (CRITICAL step)
   - Phase E: Monitor extraction phases
   - Phase F: Verify imported data (field-by-field)
   - Phase G: Test incremental sync
3. **Output**: TEST_RESULTS.md with ship/block recommendation

## How to Fix an Existing Snap-in

### If the agent produced wrong code:

```bash
/devrev:improve-skill The architect generated [wrong thing] but it should be [correct thing]
```

The Skill Improver will:
1. Diagnose the root cause
2. Find the exact reference file to fix
3. Show you the proposed patch
4. Apply only after your confirmation
5. Log the change in CHANGELOG.md

### Root cause mapping (where to look):

| Problem | File to check |
|---------|--------------|
| Wrong API endpoint / request format | `references/airsync-template.md` or `simple-snapin.md` |
| Wrong manifest syntax | `references/simple-snapin.md` or `airsync-template.md` |
| Wrong SDK method or import | `references/airsync-template.md` |
| Wrong CLI command | `references/cli-workflow.md` |
| Bad PRD/TDD structure | `references/prd-template.md` or `tdd-template.md` |
| Wrong dashboard/widget JSON | `references/dashboard-json-rules.md` |
| Wrong SQL in widget | `references/sql-rules.md` |
| Wrong visualization | `references/visualization-catalog.md` |
| Wrong workflow JSON | `references/workflow-json-rules.md` |
| Wrong object schema JSON | `references/object-json-rules.md` |
| bin/devrev-api auth or schema errors | `references/devrev-api-cli.md` |

## How to Add a New Agent/Skill

1. Create `skills/<skill-name>/SKILL.md` with the workflow instructions
2. Add `skills/<skill-name>/references/*.md` for knowledge files
3. Create `agents/<agent-name>.md` pointing to the skill
4. Create `commands/<command-name>.md` that invokes the agent
5. Add the command path to BOTH `plugin.json` files (root + `.claude-plugin/`)
6. Update `CLAUDE.md` with the new command
7. Test: `claude --plugin-dir .`

## Key DevRev Platform Facts

### Snap-in SDKs
- **Simple snap-ins**: `@devrev/typescript-sdk` only
- **AirSync snap-ins**: `@devrev/ts-adaas` (sync engine) + `@devrev/typescript-sdk` (DevRev API calls)

### AirSync Pipeline (ADaaS)
```
1. External Sync Units Extraction (discover containers)
2. Metadata Extraction (fetch schema)
3. Data Extraction (pull records — users first, then primary, then nested)
4. Attachments Extraction (download files)
5. Transform & Loading (apply mappings, create objects)
```

### Timeout Model
- 10-minute soft timeout → `onTimeout` callback fires
- 13-minute hard timeout → Lambda killed
- Use `processTask({ task, onTimeout })` — SDK manages this

### Sync Model
- DevRev platform triggers periodic sync
- Platform provides `lastSuccessfulSyncStarted` — snap-ins do NOT manage timestamps
- No webhook registration needed

### Implementation Vertical

**Dashboards:**
- Widget JSON → `widget-preview` → widget ID → `dashboard-preview` → dashboard ID
- SQL: No `SELECT *`, no table aliases in `sql_expression`, fully qualify all references
- Viz types: metric, line, column, bar, table, donut, pie, packed_bubble, heatmap

**Workflows:**
- AMC automation templates — trigger + condition + action blocks
- Live operation schemas fetched via `bin/devrev-api` before generating JSON

**Custom objects:**
- `leaf-type-schema.json` — adds or modifies fields on existing DevRev object types
- Supported field types: text, enum, boolean, date, reference, multi-value

### DevRev CLI (key commands)
```bash
devrev profiles authenticate                              # Login
devrev snap_in_package create-one --slug <name>           # Create package
devrev snap_in_version validate-manifest                  # Validate
devrev snap_in_version create-one --archive dist.tar.gz   # Deploy
chef-cli validate-metadata                                 # AirSync metadata
chef-cli configure-mappings                                # AirSync mappings
```
