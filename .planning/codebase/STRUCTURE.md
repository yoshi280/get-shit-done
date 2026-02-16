# Codebase Structure

**Analysis Date:** 2026-02-16

## Directory Layout

```
get-shit-done/
├── agents/                    # Agent definitions (specialized task executors)
├── commands/
│   └── gsd/                   # User-facing CLI commands
├── get-shit-done/             # The "skill" - shared workflows, tools, references
│   ├── bin/                   # Utilities and tools
│   ├── references/            # Shared knowledge base
│   ├── templates/             # Markdown templates for docs
│   └── workflows/             # Step-by-step procedures
├── hooks/                     # Background tasks and statusline
├── bin/                       # Installation and setup
├── scripts/                   # Build scripts
├── docs/                      # User documentation
├── CHANGELOG.md               # Version history
├── package.json               # Node.js package metadata
└── README.md                  # Project readme
```

## Directory Purposes

**agents/**
- Purpose: Agent definitions that execute specialized tasks
- Contains: Markdown files (gsd-*.md) with role, tools, execution flow
- Key files:
  - `gsd-executor.md`: Executes tasks from PLAN.md atomically
  - `gsd-planner.md`: Creates PLAN.md with task breakdown
  - `gsd-verifier.md`: Validates SUMMARY.md against success criteria
  - `gsd-plan-checker.md`: Verifies PLAN.md structure and completeness
  - `gsd-phase-researcher.md`: Researches domain for phase context
  - `gsd-project-researcher.md`: Gathers initial project vision
  - `gsd-roadmapper.md`: Creates project roadmap with phases
  - `gsd-codebase-mapper.md`: Analyzes codebase and writes documentation
  - `gsd-debugger.md`: Diagnoses project state issues
  - `gsd-integration-checker.md`: Validates integrations and dependencies

**commands/gsd/**
- Purpose: User-facing commands (invoked via `/gsd:command`)
- Contains: Markdown with frontmatter (agent, tools, description) + orchestrator logic
- Key files:
  - `new-project.md`: Initialize new GSD project
  - `plan-phase.md`: Create PLAN.md for a phase (orchestrator)
  - `execute-phase.md`: Execute PLAN.md and create SUMMARY.md (orchestrator)
  - `research-phase.md`: Research phase context (orchestrator)
  - `help.md`: Display available commands
  - `settings.md`: Configure project behavior
  - `debug.md`: Diagnose project state
- Pattern: Each command starts with orchestrator responsibilities, may spawn agents

**get-shit-done/bin/**
- Purpose: Utility tools and test suites
- Key files:
  - `gsd-tools.cjs`: 50+ CLI commands for state management, git operations, validation
  - `gsd-tools.test.cjs`: Tests for gsd-tools

**get-shit-done/references/**
- Purpose: Shared knowledge, patterns, decision logic
- Key files:
  - `git-integration.md`: Commit strategy (outcomes not process)
  - `model-profiles.md`: Model capabilities, cost, speed tradeoffs
  - `model-routing-logic.md`: How to assign models to agents
  - `verification-patterns.md`: Testing and verification strategies
  - `tdd.md`: Test-driven development pattern for execution
  - `phase-argument-parsing.md`: How to parse phase numbers and arguments
  - `planning-config.md`: Schema and defaults for config.json
  - `checkpoints.md`: Checkpoint protocol (pause/resume/decide)
  - `continuation-format.md`: How to resume after checkpoint
  - `gsd-orchestrator-balancing-rules.md`: Multi-model load balancing

**get-shit-done/workflows/**
- Purpose: Detailed step-by-step procedures
- Key files:
  - `plan-phase.md`: Research → Plan → Verify loop for phase planning
  - `execute-plan.md`: Execute PLAN.md and create SUMMARY.md atomically
  - `research-phase.md`: Domain research workflow
  - `verify-phase.md`: Verification and UAT workflow
  - `add-phase.md`: Append new phase to roadmap
  - `insert-phase.md`: Insert decimal phase (e.g., 1.1 after 1)
  - `remove-phase.md`: Delete phase and renumber subsequent
  - `complete-milestone.md`: Archive completed milestone
- Pattern: Every workflow starts with state load, ends with state progression

**get-shit-done/templates/**
- Purpose: Markdown templates for creating standard documents
- Contains: Pre-filled templates with frontmatter and structure
- Examples:
  - `PLAN.md` template with task structure
  - `SUMMARY.md` template with completion tracking
  - `VERIFICATION.md` template for UAT
  - `CONTEXT.md` template for phase context

**hooks/**
- Purpose: Background tasks and UI integrations
- Key files:
  - `gsd-statusline.js`: Displays model, task, context usage in statusline
  - `gsd-check-update.js`: Background update check for new versions
  - `dist/`: Bundled hooks (copied during installation)

**bin/**
- Purpose: Installation and setup
- Key files:
  - `install.js`: Multi-runtime installer (~1800 lines)
    - Handles Claude Code, OpenCode, Gemini installations
    - Global (~/.claude, ~/.config/opencode, ~/.gemini) or local (./.claude, ./.opencode, ./.gemini)
    - File manifest tracking for local patch persistence
    - Settings.json configuration (hooks, statusline, experimental features)

**scripts/**
- Purpose: Build and development scripts
- Key files:
  - `build-hooks.js`: Copy hooks from `hooks/` to `hooks/dist/` for distribution

## Key File Locations

**Entry Points:**

- `bin/install.js`: Installation and setup
- `commands/gsd/help.md`: Command listing and discovery
- `commands/gsd/new-project.md`: Project initialization entry point

**Configuration:**

- `get-shit-done/templates/config.json`: Planning behavior schema
- `get-shit-done/references/planning-config.md`: Config documentation

**Core Logic:**

- `get-shit-done/bin/gsd-tools.cjs`: 50+ operational commands
- `commands/gsd/plan-phase.md`: Plan creation orchestrator
- `commands/gsd/execute-phase.md`: Execution orchestrator
- `agents/gsd-executor.md`: Task execution logic

**Testing:**

- `get-shit-done/bin/gsd-tools.test.cjs`: gsd-tools test suite

## Naming Conventions

**Files:**

- Agents: `gsd-[purpose].md` (e.g., `gsd-executor.md`, `gsd-planner.md`)
- Commands: `[command-name].md` (e.g., `plan-phase.md`, `execute-phase.md`)
- Workflows: `[workflow-name].md` (e.g., `plan-phase.md`, `execute-plan.md`)
- References: `[topic].md` (e.g., `git-integration.md`, `model-profiles.md`)
- In `.planning/`: `PHASE.md` (e.g., `BRIEF.md`, `STATE.md`, `ROADMAP.md`)

**Directories:**

- Agent groups: `agents/` (flat, no subdirs)
- Command groups: `commands/gsd/` (flat, no subdirs currently)
- Workflow organization: `get-shit-done/workflows/` (flat)
- Project phases: `.planning/phases/NN-name/` where NN is phase number
- Decimal phases: `.planning/phases/NN.D-name/` where D is decimal (1.1, 1.2, etc.)

**Code style:**

- JavaScript: Node.js CommonJS (`.cjs`), no transpilation
- Markdown: YAML frontmatter + body (agents, commands, workflows)
- JSON: 2-space indent (config.json, package.json, manifests)

## Where to Add New Code

**New Agent (for new task type):**
- Primary code: `agents/gsd-[name].md`
- Pattern: Copy existing agent, update role, execution flow, tool permissions
- Register: Reference in command frontmatter `agent: gsd-[name]`

**New Command (for new user operation):**
- Implementation: `commands/gsd/[name].md`
- Pattern: Frontmatter (description, agent assignment, tools) + orchestrator logic
- If simple: Direct implementation in command
- If complex: Assign to agent via frontmatter

**New Workflow (for complex procedures):**
- Primary code: `get-shit-done/workflows/[name].md`
- Pattern: `<step>` elements with gates, validation, decision points
- Reference: Use `@~/.claude/get-shit-done/workflows/[name].md` in agents/commands

**New Reference (for shared knowledge):**
- Primary code: `get-shit-done/references/[topic].md`
- Pattern: Descriptive sections with examples, decision tables, code snippets
- Usage: Referenced via `@~/.claude/get-shit-done/references/[name].md` in workflows/agents

**Utilities (for new operations):**
- Primary code: Add command to `get-shit-done/bin/gsd-tools.cjs`
- Pattern: Parse args, implement logic, output JSON or structured text
- Invocation: `node ~/.claude/get-shit-done/bin/gsd-tools.cjs [command] [args]`

**Hook (for background tasks):**
- Primary code: `hooks/[name].js`
- Pattern: Node.js script, executed in background
- Build: Run `npm run build:hooks` to copy to `hooks/dist/`
- Integration: Register in `.claude/settings.json` hooks/statusline config

## Special Directories

**`.planning/` (per-project, created at init):**
- Purpose: Project-specific state and planning documents
- Generated: Yes (created by `gsd-tools.cjs phase add`)
- Committed: Yes, entire directory is version-controlled
- Contents:
  - `STATE.md`: Current progress, phase status, model assignments
  - `BRIEF.md`: Original user vision
  - `ROADMAP.md`: Phase breakdown with status
  - `config.json`: Project behavior settings
  - `phases/NN-name/`: Phase directories with PLAN.md, SUMMARY.md, RESEARCH.md
  - `decisions/`: Recorded decisions with timestamp and rationale
  - `codebase/`: Analysis docs (written by `/gsd:map-codebase`)

**`.planning/codebase/`:**
- Purpose: Codebase documentation (architecture, conventions, testing, etc.)
- Generated: Yes (by `/gsd:map-codebase` agents)
- Committed: Yes
- Contents:
  - `ARCHITECTURE.md`: System design and patterns
  - `STRUCTURE.md`: Directory layout and file organization
  - `CONVENTIONS.md`: Coding standards and naming
  - `TESTING.md`: Test patterns and frameworks
  - `STACK.md`: Technology stack
  - `INTEGRATIONS.md`: External services and APIs
  - `CONCERNS.md`: Technical debt and issues

**`hooks/dist/`:**
- Purpose: Bundled hooks for distribution
- Generated: Yes (by `npm run build:hooks`)
- Committed: No (generated during build)
- Contents: Copies of hooks from `hooks/` for installation

**`node_modules/`:**
- Purpose: NPM dependencies
- Generated: Yes (by `npm install`)
- Committed: No (.gitignore)
- Contents: Only devDependencies (esbuild)

## Git Repository Structure

- **Main branch:** main
- **Feature branches:** feature/* (for development)
- **Commit strategy:** Outcome-focused (see references/git-integration.md)
  - Initialization: `docs: initialize [project-name]`
  - Planning: No commits (PLAN.md is intermediate)
  - Execution: Per-task `feat: phase [N] [task-title]`
  - Plan completion: `feat: phase [N] plan [M] complete` + SUMMARY.md
  - Metadata: `docs: phase [N] plan [M] summary` for SUMMARY.md update

---

*Structure analysis: 2026-02-16*
