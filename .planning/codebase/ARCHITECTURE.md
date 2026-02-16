# Architecture

**Analysis Date:** 2026-02-16

## Pattern Overview

**Overall:** Hierarchical agent-orchestrator system with spec-driven development workflow

**Key Characteristics:**
- Multi-agent system with orchestrators spawning specialized subagents
- Spec-first (BRIEF.md) → Research → Plan → Execute → Verify workflow
- Git-integrated with atomic per-task commits
- Context engineering via reference documents and state management
- Role-based agents with specific tool permissions and responsibilities

## Layers

**Command Layer:**
- Purpose: User-facing CLI interface via commands
- Location: `commands/gsd/`
- Contains: Markdown command definitions with frontmatter (allowed-tools, description, agent assignment)
- Depends on: Agents, workflows, reference docs
- Used by: Claude Code, OpenCode, Gemini CLI via slash commands

**Orchestrator Layer:**
- Purpose: Coordinate multi-step workflows, spawn subagents, manage state progression
- Location: `commands/gsd/` and agents assigned via frontmatter
- Examples: `/gsd:plan-phase` (spawns gsd-planner), `/gsd:execute-phase` (spawns gsd-executor)
- Pattern: Parse arguments → Load state → Dispatch to agent → Collect result → Progress state
- Used by: Commands, other orchestrators via Task tool

**Agent Layer:**
- Purpose: Execute specialized tasks with specific tool constraints
- Location: `agents/gsd-*.md`
- Contains: Agent definitions with role, execution flow, tool restrictions
- Examples: `gsd-planner.md`, `gsd-executor.md`, `gsd-verifier.md`, `gsd-codebase-mapper.md`
- Inherits context from spawning orchestrator
- Used by: Orchestrators, other agents via Task tool

**Workflow Layer:**
- Purpose: Detailed step-by-step procedures for complex operations
- Location: `get-shit-done/workflows/`
- Contains: Markdown workflows with procedural steps, validation gates, decision points
- Examples: `plan-phase.md`, `execute-plan.md`, `research-phase.md`
- Pattern: Gate → Validate → Execute → Record → Decide
- Used by: Agents (via @-references)

**Reference Layer:**
- Purpose: Shared knowledge, patterns, rules, and decision logic
- Location: `get-shit-done/references/`
- Contains: Configuration schemas, model routing logic, git patterns, verification rules
- Examples: `model-profiles.md`, `git-integration.md`, `verification-patterns.md`, `tdd.md`
- Used by: Workflows, agents (via @-references)

**State Layer:**
- Purpose: Persistent project state, configuration, progress tracking
- Location: `.planning/` (created at project init)
- Contains:
  - `STATE.md`: Project roadmap, phase progress, decisions, metrics
  - `config.json`: Planning behavior (gates, execution mode, model overrides)
  - `BRIEF.md`: User's original project vision
  - Phase directories: `phases/NN-name/` with PLAN.md, SUMMARY.md, RESEARCH.md
- Used by: All agents via state load operations

**Tools Layer:**
- Purpose: Utility functions for common operations
- Location: `get-shit-done/bin/gsd-tools.cjs`
- Contains: CLI utility with 50+ commands (state management, phase ops, git commits, verification)
- Invoked via: `node ~/.claude/get-shit-done/bin/gsd-tools.cjs <command> [args]`
- Used by: Workflows, agents for state progression, validation, summary verification

**Installation Layer:**
- Purpose: Install and configure GSD for target runtime
- Location: `bin/install.js`
- Contains: Multi-runtime installer (Claude Code, OpenCode, Gemini) with global/local options
- Behavior: Copies agents/commands/workflows/hooks to runtime config directory, manages file manifest
- Used by: `npx get-shit-done-cc` (npm installation)

**Hooks Layer:**
- Purpose: Background tasks, status display, update checking
- Location: `hooks/`
- Examples: `gsd-statusline.js` (displays model, task, context usage), `gsd-check-update.js` (background update check)
- Invoked by: Runtime via SessionStart hook, statusline configuration
- Used by: Claude Code, Gemini CLI settings

## Data Flow

**Project Initialization:**

1. User runs `/gsd:new-project` command
2. Orchestrator spawns gsd-project-researcher
3. Researcher gathers project context (vision, scope, constraints)
4. Creates BRIEF.md, CONTEXT.md
5. Orchestrator creates ROADMAP.md with phases
6. Commits: `docs: initialize [project-name] ([N] phases)`
7. STATE.md created tracking phase 1

**Phase Execution:**

1. User runs `/gsd:plan-phase [N]` for phase
2. Orchestrator loads STATE.md (current progress, model assignment)
3. Optionally spawns gsd-phase-researcher for domain research
4. Spawns gsd-planner to create PLAN.md with task breakdown
5. Spawns gsd-plan-checker for verification loop
6. On pass: commits `plan: phase [N] plan [M]`
7. User runs `/gsd:execute-phase [N]`
8. gsd-executor loads PLAN.md, executes each task atomically
9. Each task commit: `feat: phase [N] [task-title]`
10. Plan completion: commits SUMMARY.md + STATE.md + ROADMAP.md update
11. gsd-verifier validates outcomes against success criteria

**State Management:**

State flows through `STATE.md` + `config.json`:
- Phase progress (which phases done, current phase)
- Plan metrics (tasks completed, duration, files changed)
- Model assignments (which model for research vs execution)
- Gate settings (interactive vs autonomous mode, verification requirements)
- Decisions made (with timestamp, rationale, impact)

All state mutations via `gsd-tools.cjs state update/patch` commands.

## Key Abstractions

**Plan (PLAN.md):**
- Purpose: Executable specification for completing phase objectives
- Examples: `phases/01-setup/01-PLAN.md`
- Structure: Frontmatter (phase, plan #, type, autonomous, wave) + Objective + Context refs + Tasks with verification + Output spec
- Pattern: Each task is atomic, has type (auto/checkpoint/tdd), success criteria, must_haves
- Used by: gsd-executor, gsd-plan-checker

**Summary (SUMMARY.md):**
- Purpose: Outcome record of executed plan
- Examples: `phases/01-setup/01-SUMMARY.md`
- Structure: Frontmatter + Summary + Tasks (with commit hashes) + Deviations + Success verification
- Pattern: Must have 1:1 correspondence with tasks in PLAN.md
- Verified by: gsd-verifier via `verify-summary` tool

**Agent Assignment:**
- Purpose: Declarative model/agent selection based on task type
- Pattern: Frontmatter `agent: gsd-planner` assigns which agent handles the work
- Routing: Profile + model selection logic in `references/model-routing-logic.md`
- Result: Different models for research (bigger), execution (speed), verification (accuracy)

**Checkpoint:**
- Purpose: Halt execution for user input/decision
- Type: `type="checkpoint:decision"` or `type="checkpoint:review"`
- Pattern: Task executes to checkpoint, returns structured message, fresh agent spawned to continue
- Used for: Architectural decisions, UAT approval, scope changes

**Deviation:**
- Purpose: Track when execution diverges from plan
- Pattern: Recorded in SUMMARY.md with rationale
- Trigger: Unexpected blockers, security concerns, better approach discovered
- Resolution: gsd-executor decides whether to pause (checkpoint) or auto-recover

## Entry Points

**CLI Command:**
- Location: `commands/gsd/[command].md`
- Triggers: User runs `/gsd:[command]`
- Responsibilities: Parse args, call orchestrator logic, handle output
- Example: `/gsd:plan-phase` → frontmatter says `agent: gsd-planner` → spawns planner

**Workflow Script:**
- Location: `get-shit-done/workflows/[workflow].md`
- Triggers: Referenced via `@~/.claude/get-shit-done/workflows/[name].md` in agent
- Responsibilities: Execute procedural steps, apply gates, record decisions
- Example: Workflows always start with state load, end with progress update

**Agent Spawn:**
- Location: `agents/gsd-*.md`
- Triggers: Orchestrator uses Task tool with agent assignment
- Responsibilities: Execute specialized task, produce output artifact (PLAN.md, RESEARCH.md, etc.)
- Example: gsd-executor spawns via orchestrator, reads PLAN.md, executes tasks, creates SUMMARY.md

**Installer:**
- Location: `bin/install.js`
- Triggers: `npm install -g get-shit-done-cc` or `npx get-shit-done-cc`
- Responsibilities: Copy agents/commands/workflows to runtime config, configure hooks, create manifest
- Produces: Files in `~/.claude/get-shit-done/` (global) or `./.claude/get-shit-done/` (local)

## Error Handling

**Strategy:** Fail-fast with checkpoint, record decision

**Patterns:**

**Authentication Gate:**
- Trigger: API key missing (GitHub, OpenAI, etc.)
- Response: Treat as checkpoint, pause execution
- Pattern: In execute workflow, detect missing auth, save state, return auth request
- Recovery: User provides credentials, `/gsd:resume-work` spawns executor to continue

**Validation Failure:**
- Trigger: PLAN verification fails, task completion unverified, output missing required artifacts
- Response: gsd-plan-checker or gsd-verifier rejects, requests fix
- Pattern: Loop: Planner → Checker (fails) → Planner (refine) → Checker (pass)
- Max iterations: Configurable in `config.json` (default 3)

**Deviation:**
- Trigger: Unexpected error, better approach found, scope conflict
- Response: gsd-executor records deviation with rationale
- Pattern: Continue execution if auto-recoverable, checkpoint if decision needed
- Resolution: Recorded in SUMMARY.md for post-mortem analysis

**Missing Dependency:**
- Trigger: Reference file missing (@references), state file missing
- Response: Error with recovery instruction
- Pattern: Suggest `gsd-tools verify` to diagnose
- Prevention: Manifest tracking ensures consistency

## Cross-Cutting Concerns

**Logging:** Markdown journal entries in STATE.md with timestamps, decision points as `.planning/decisions/` folder entries

**Validation:** Three-layer: frontmatter schema validation → workflow gate checks → artifact verification (must_haves)

**Authentication:** Environment variable driven (.env not tracked), checkpoint when missing, error message guides setup

**Git Integration:** Atomic commits per task, metadata commits for plan completion, CHANGELOG.md auto-generated from SUMMARY.md records

**State Progression:** Only via `gsd-tools state update/patch`, never direct file edits, prevents inconsistency

**Model Routing:** Profile-based selection per agent type (research/plan/execute/verify), overridable in config.json

---

*Architecture analysis: 2026-02-16*
