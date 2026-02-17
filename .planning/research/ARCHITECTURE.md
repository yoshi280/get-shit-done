# Architecture Research

**Domain:** Multi-runtime AI agent orchestration — GSD v1.1 Codex CLI + OpenCode Support
**Researched:** 2026-02-16
**Confidence:** MEDIUM (runtime-specific details based on training data + installer code inspection; Codex CLI and OpenCode evolve rapidly — validate before implementing)

---

## Current Architecture: What Exists Today

GSD is a layered agent-orchestrator system built on Claude Code's execution model. The architecture has seven layers:

```
┌─────────────────────────────────────────────────────────────────┐
│  Command Layer                                                   │
│  ~/.claude/commands/gsd/*.md  (Claude Code /gsd: namespace)     │
│  ~/.config/opencode/command/gsd-*.md  (OpenCode /gsd- namespace)│
├─────────────────────────────────────────────────────────────────┤
│  Orchestrator Layer                                             │
│  ~/.claude/get-shit-done/workflows/*.md                         │
│  Reads config, spawns agents via Task tool, manages state       │
├─────────────────────────────────────────────────────────────────┤
│  Agent Layer                                                    │
│  ~/.claude/agents/gsd-*.md  (Claude Code native)               │
│  Specialized work: planning, execution, research, verification  │
├─────────────────────────────────────────────────────────────────┤
│  State Tool Layer                                               │
│  gsd-tools.cjs  (zero-dep Node.js CLI)                         │
│  All state mutations, commits, model resolution, phase ops      │
├─────────────────────────────────────────────────────────────────┤
│  State Layer                                                    │
│  .planning/  (per-project)                                      │
│  STATE.md, ROADMAP.md, config.json, phase artifacts             │
└─────────────────────────────────────────────────────────────────┘
```

The **critical architectural dependency** on Claude Code: the `Task` tool. Every orchestrator uses it to spawn subagents:

```
Task(
  subagent_type="gsd-executor",
  model="sonnet",
  prompt="...",
  description="..."
)
```

This call is Claude Code's mechanism for spawning isolated agent contexts with fresh 200k token windows. It is the backbone of:
- Parallel researcher spawning (N researchers for N dimensions)
- Wave-based plan execution (multiple executors per wave)
- Planner → plan-checker revision loops
- Roadmapper creation and revision

**OpenCode and Codex CLI do not have a direct equivalent.**

---

## The v1.1 Gap Analysis

### What OpenCode Currently Has (Install-Level)

OpenCode is already supported at installation time via `bin/install.js`. The installer:
- Flattens command namespace: `commands/gsd/plan.md` → `command/gsd-plan.md`
- Converts frontmatter: `allowed-tools: [Read, Write]` → `tools: { read: true, write: true }`
- Maps tool names: uppercase → lowercase (`Read` → `read`, `Bash` → `bash`)
- Maps `AskUserQuestion` → `question`
- Maps `skill` invocation for command-to-command calls
- Creates `opencode.json` for permission settings
- Respects `XDG_CONFIG_HOME` for config directory

**What OpenCode does NOT currently have:**
- Task tool (no subagent spawning)
- Model parameter selection per spawn (no `model=` on Task)
- Native parallel execution of sub-contexts
- `subagent_type` agent routing

The existing support is **install parity, not workflow parity**. A user on OpenCode can invoke `/gsd-plan-phase` but when the workflow reaches `Task(subagent_type="gsd-planner", ...)`, it fails silently or errors. The entire wave-based execution model is non-functional.

### What Codex CLI Has (From Training Data — LOW Confidence, Verify)

Codex CLI (OpenAI's terminal agent) uses:
- Markdown files for system prompts and context injection
- Tool calling via OpenAI's function calling API
- No native subagent spawning mechanism equivalent to Claude Code's Task tool
- No `subagent_type` routing
- Different tool names and invocation patterns than Claude Code
- Agent definitions use a different frontmatter schema

GSD has **zero Codex CLI support today**. No install target, no command format, no tool name mapping.

---

## The Core Problem: Task Tool Abstraction Gap

The Task tool provides:
1. **Isolated context** — subagent gets a fresh 200k window, preventing orchestrator context bleed
2. **Model selection** — `model="sonnet"` lets orchestrator route work to appropriate model
3. **Blocking execution** — orchestrator awaits completion before proceeding
4. **Parallel fan-out** — multiple Task calls in parallel execute concurrently

For non-Claude runtimes, GSD needs a fallback pattern for each of these properties.

### What "Parity" Actually Means Per Runtime

| Capability | Claude Code | OpenCode | Codex CLI |
|------------|-------------|----------|-----------|
| Slash commands | `/gsd:plan-phase` | `/gsd-plan-phase` | Unknown — needs verification |
| Subagent spawn | `Task(subagent_type=...)` | No equivalent | No equivalent |
| Parallel execution | Task fan-out | Not available | Not available |
| Model selection | `model="sonnet"` | Provider-defined | OpenAI model routing |
| Tool names | PascalCase | lowercase | Unknown |
| Agent files | `agents/*.md` | Command format | Unknown |
| Interactive prompts | `AskUserQuestion` | `question` | Unknown |
| Config location | `~/.claude/` | `~/.config/opencode/` | Unknown |

---

## Recommended Architecture: Runtime Adapter Pattern

### Design Decision: Shared Workflows, Runtime-Specific Spawning

**Recommendation:** Keep all workflow logic in shared files. Add a thin runtime detection + adapter layer that transforms Task calls into runtime-appropriate equivalents.

This is the minimal-change approach. It avoids duplicating workflow files per runtime (which would create a maintenance nightmare as workflows evolve) while enabling each runtime to execute what it can.

**Rejected alternative:** Runtime-specific workflow files (e.g., `execute-phase-opencode.md`). Reason: Workflows are 200-500 lines each. Maintaining 3 copies of 30+ workflow files for 3 runtimes means 10x maintenance burden for every future GSD improvement.

### Component 1: Runtime Detection

Detection must happen at the earliest possible moment — in `gsd-tools.cjs init` commands, before any workflow step runs.

**Detection strategy (layered, first match wins):**

```
1. config.json "runtime" field (explicit user override)
   ↓ not set
2. CLAUDE_DESKTOP_ENABLED, CLAUDE_CODE_VERSION env vars → "claude-code"
   ↓ not set
3. OPENCODE_VERSION, OPENCODE_CONFIG_DIR env vars → "opencode"
   ↓ not set
4. Check process.env for Codex-specific vars (verify what Codex sets)
   ↓ not set
5. Filesystem heuristics:
   - ~/.claude/ exists AND process loaded from claude → "claude-code"
   - ~/.config/opencode/ exists → "opencode" (weak signal, cross-check)
   ↓ inconclusive
6. Default: "claude-code" (preserves backward compatibility)
```

**Output from `gsd-tools.cjs init`:** Add `runtime` field to all init JSON payloads. Workflows read it and branch accordingly.

**New config.json field:**
```json
{
  "runtime": "claude-code"  // | "opencode" | "codex-cli" | "auto"
}
```

`"auto"` (default) triggers heuristic detection. Explicit values override. This also lets users manually override if detection fails.

**Integration point:** `loadConfig()` in `gsd-tools.cjs` (line ~164). Add `runtime` field resolution alongside `model_profile`, `commit_docs`, etc.

### Component 2: Spawn Adapter — The Key Gap

All workflows that use `Task(...)` need to route through an adapter pattern. The adapter lives in the **workflow markdown prose**, not in gsd-tools.cjs (since it involves LLM behavior, not Node.js computation).

**Pattern: Conditional spawn block in workflow prose:**

```markdown
## Spawn Researcher

**If runtime is "claude-code":**
Use Task tool:
Task(
  subagent_type="gsd-phase-researcher",
  model="{researcher_model}",
  prompt="...",
  description="Phase research"
)

**If runtime is "opencode" or "codex-cli" (no Task tool):**
Execute the researcher role inline:
- Read /path/to/gsd-phase-researcher.md for your role instructions
- Execute the research inline following those instructions
- Write output to {research_path}
```

This is the **inline fallback pattern**: when Task is unavailable, the orchestrator itself takes on the agent role, reading the agent's instruction file and executing it in the same context window.

**Tradeoffs of inline fallback:**
- Pro: Works on any runtime with zero additional tooling
- Pro: No new files, no new infrastructure
- Con: Consumes orchestrator context window (no isolation)
- Con: Sequential-only (no parallel fan-out)
- Con: No model routing (runs at whatever model the orchestrator is on)
- Con: Context bleed — prior orchestrator state is visible to "agent"

For v1.1, these tradeoffs are acceptable. Inline fallback gets to working parity. Parallel execution and model routing are enhancement opportunities for v1.2+.

### Component 3: Parallelization Flag Already Handles Sequential Fallback

GSD already has a `parallelization` flag in `config.json`. When `false`, wave execution is sequential. This flag serves as the primary mechanism for degraded-mode execution on non-Claude runtimes.

**Recommended behavior:** When runtime is "opencode" or "codex-cli", auto-set effective parallelization to `false` regardless of config. Workflows that currently read `PARALLELIZATION` from init will naturally serialize.

The existing flag is already checked in `execute-phase.md`:
> "When `parallelization` is false, plans within a wave execute sequentially."

This means sequential execution already works today — the gap is that Task calls still fail even in sequential mode on non-Claude runtimes. The fix is the inline fallback, not changing the parallelization logic.

### Component 4: Model Routing Fallback

`gsd-tools.cjs resolveModelInternal()` maps agent types to models (opus/sonnet/haiku). For Claude Code, this resolves to actual Claude model IDs.

For OpenCode and Codex CLI, model IDs are provider-specific. The resolution table needs runtime-aware model name mapping.

**New config structure:**
```json
{
  "model_profile": "balanced",
  "model_overrides": {
    "gsd-executor": "gpt-4o",
    "gsd-planner": "gpt-4o",
    "gsd-phase-researcher": "gpt-4o-mini"
  }
}
```

`model_overrides` already exists in gsd-tools.cjs (line ~4087). The gap is documentation and a default model table for non-Anthropic providers.

**Integration point:** `resolveModelInternal()` in gsd-tools.cjs. Add a `runtime`-aware default model table alongside `MODEL_PROFILES`.

---

## System Overview: v1.1 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  Command Layer  (runtime-specific install)                      │
│  Claude Code: ~/.claude/commands/gsd/*.md                       │
│  OpenCode:    ~/.config/opencode/command/gsd-*.md               │
│  Codex CLI:   [new: needs format research — see gaps]           │
├─────────────────────────────────────────────────────────────────┤
│  Runtime Detection  (NEW in gsd-tools.cjs + config.json)       │
│  Detects: claude-code | opencode | codex-cli                    │
│  Outputs: runtime field in all init JSON payloads               │
├─────────────────────────────────────────────────────────────────┤
│  Orchestrator Layer  (MODIFIED: conditional spawn blocks)       │
│  Shared workflow files with runtime-conditional Task vs inline  │
│  ~30 workflow files, ~5 workflows need Task changes             │
├─────────────────────────────────────────────────────────────────┤
│  Spawn Adapter  (NEW: in workflow prose)                        │
│  Claude Code: Task(subagent_type=..., model=..., prompt=...)    │
│  OpenCode/Codex: Inline execution with agent file read          │
├─────────────────────────────────────────────────────────────────┤
│  Agent Layer  (UNCHANGED for Claude Code)                       │
│  ~/.claude/agents/gsd-*.md  (existing)                          │
│  [New: OpenCode/Codex agent format if needed — see gaps]        │
├─────────────────────────────────────────────────────────────────┤
│  State Tool Layer  (MODIFIED: runtime field in init output)     │
│  gsd-tools.cjs — add runtime detection + model table           │
├─────────────────────────────────────────────────────────────────┤
│  State Layer  (UNCHANGED)                                       │
│  .planning/ directory — runtime-independent                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Component Boundaries

| Component | Responsibility | Location | Status |
|-----------|---------------|----------|--------|
| Runtime Detector | Identify which CLI is running | gsd-tools.cjs `loadConfig()` | NEW |
| Init JSON Runtime Field | Surface runtime to workflows | gsd-tools.cjs `init *` commands | MODIFIED |
| Spawn Adapter | Task → inline fallback | Workflow markdown prose | MODIFIED (5 workflows) |
| Model Routing Table | Map agent types to non-Anthropic models | gsd-tools.cjs `MODEL_PROFILES` | MODIFIED |
| Codex CLI Installer | Install GSD for Codex | bin/install.js | NEW section |
| Codex Command Format | Convert commands for Codex | bin/install.js converter fn | NEW |
| Tool Name Mapper | Map Claude tool names to Codex names | bin/install.js | NEW |
| OpenCode Parity | Fix existing OpenCode gaps | Workflow conditional blocks | MODIFIED |

---

## Specific Files Requiring Changes

### Modified Files

**`gsd-tools.cjs`**
- `loadConfig()`: Add `runtime` field (detection logic)
- `MODEL_PROFILES`: Add non-Claude model defaults per runtime
- All `init *` command handlers: Add `runtime` to returned JSON
- Approximately 20-30 lines of new code

**Workflows with Task calls (5 files):**
- `workflows/new-project.md` — researcher spawn (lines ~601-655), synthesizer spawn, roadmapper spawn
- `workflows/plan-phase.md` — researcher spawn (line ~237), planner spawn (line ~370), plan-checker spawn (line ~426)
- `workflows/execute-phase.md` — executor spawn (line ~103), verifier spawn (line ~299)
- `workflows/new-milestone.md` — similar spawns to new-project
- `workflows/research-phase.md` — researcher spawns

Each needs: read `runtime` from init JSON → conditional spawn block.

**`bin/install.js`**
- New `installCodexCLI()` function (or extend existing runtime installer)
- New `convertClaudeToCodexFrontmatter()` function
- Codex-specific tool name mapping table
- Codex config directory detection

### New Files

**`~/.claude/get-shit-done/references/runtime-compatibility.md`**
Documents which capabilities are available per runtime. Agents and workflows can reference this when making runtime-conditional decisions.

**Codex CLI command files** (generated by installer, not hand-authored)
The installer produces these from the shared command source files.

---

## Data Flow Changes

### Runtime Context Flow (New)

```
gsd-tools.cjs init execute-phase
    ↓ (adds runtime field)
    {
      "runtime": "opencode",
      "parallelization": false,  // effective value
      "executor_model": "gpt-4o",
      ...existing fields...
    }
    ↓
Workflow reads runtime from init JSON
    ↓
spawn block: if runtime == "claude-code" → Task(...)
             else → read agent file, execute inline
```

### Inline Fallback Data Flow

```
Orchestrator context (has project state, phase info)
    ↓
Read gsd-executor.md (agent instruction file)
    ↓
Orchestrator executes agent role inline
    ↓
Writes same artifacts (PLAN.md, SUMMARY.md, etc.)
    ↓
State update via gsd-tools.cjs (unchanged)
```

The artifact outputs are identical — same files, same paths, same STATE.md updates. Only the execution mechanism differs.

---

## Architectural Patterns

### Pattern 1: Capability-Conditioned Prose (Recommended for GSD)

**What:** Workflow markdown contains conditional blocks: "if capability X is available, do A; else do B." The LLM executing the workflow reads the condition and follows the appropriate branch.

**When to use:** This is the right pattern for GSD because the "adapter" is interpreted by an LLM, not compiled code. Markdown prose is the natural conditional language for LLM-executed workflows.

**Trade-offs:**
- Pro: No new infrastructure. Works today in all runtimes that can read markdown.
- Pro: Single source of truth — one workflow file, multiple execution paths.
- Pro: Graceful degradation — non-Claude runtimes get sequential inline execution.
- Con: Verbose. Each spawn site needs ~10-15 lines of conditional prose.
- Con: LLM must correctly identify its runtime and follow the right branch.
- Con: Not machine-enforceable — depends on correct LLM behavior.

**Example (from execute-phase.md context):**
```markdown
## Spawn Executor for Plan {N}

Read `runtime` from init JSON.

**If runtime == "claude-code":**
Use Task tool:
  Task(
    subagent_type="gsd-executor",
    model="{executor_model}",
    prompt="Execute plan {N}..."
  )
Wait for Task to complete.

**If runtime == "opencode" or "codex-cli":**
Execute the executor role inline for this plan:
1. Read ~/.claude/agents/gsd-executor.md for role instructions
2. Read {plan_file} for the plan to execute
3. Follow gsd-executor.md instructions completely
4. Create SUMMARY.md and update STATE.md as instructed
Note: Proceed to next plan only after this completes.
```

### Pattern 2: Runtime Field in Config (Prerequisite)

**What:** `gsd-tools.cjs` detects and exposes the runtime as a stable, queryable field. Workflows never detect runtime themselves — they always read from init JSON.

**When to use:** Always. Centralizing detection in gsd-tools.cjs means detection logic is in one place, testable, and consistent.

**Trade-offs:**
- Pro: One detection algorithm, all workflows benefit automatically.
- Pro: Users can override with `"runtime": "codex-cli"` in config.json when detection fails.
- Con: Heuristic detection may fail in edge cases (e.g., OpenCode installed in non-standard path).

### Pattern 3: Effective Parallelization Override

**What:** When non-Claude runtime detected, override effective parallelization to `false` in init JSON, regardless of `config.json` setting. Workflows already handle `parallelization: false` correctly — sequential execution already works.

**When to use:** Always for non-Claude runtimes. Parallel Task calls on non-Claude runtimes would be meaningless (no parallel subagent mechanism exists).

**Trade-offs:**
- Pro: No workflow changes needed for sequential degradation — it's already implemented.
- Pro: Config setting is preserved (user choice), but overridden for safety.
- Con: Slower execution on non-Claude runtimes (expected and documented).

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Runtime-Specific Workflow Files

**What people do:** Create `execute-phase-opencode.md`, `execute-phase-codex.md` alongside the original.

**Why it's wrong:** 30+ workflow files × 3 runtimes = 90+ files to maintain. Every improvement to `execute-phase.md` must be ported to 2 other files. This has broken multi-runtime support in other tools repeatedly.

**Do this instead:** Single workflow file with capability-conditioned prose blocks.

### Anti-Pattern 2: Runtime Detection in Workflow Prose

**What people do:** Workflows check `CLAUDE_CODE_VERSION` env vars or file system paths directly.

**Why it's wrong:** Detection logic scattered across 30+ workflow files. One change to detection algorithm requires editing every file. Env var names change between CLI versions.

**Do this instead:** Detection only in `gsd-tools.cjs loadConfig()`. Workflows read `runtime` from init JSON exclusively.

### Anti-Pattern 3: Assuming Task Tool Availability

**What people do:** Skip the runtime check, assume Task always works.

**Why it's wrong:** OpenCode users get silent failures or runtime errors mid-workflow. The capability gap is real and needs explicit handling.

**Do this instead:** Every Task call site checks runtime and has an inline fallback.

### Anti-Pattern 4: Separate Agent Files Per Runtime

**What people do:** Create `gsd-executor-opencode.md`, `gsd-executor-codex.md`.

**Why it's wrong:** Same maintenance problem as workflow duplication, plus agent files are 200-400 lines each. Tool name differences (Read vs read) are already handled by the installer's frontmatter converter.

**Do this instead:** Shared agent files. The installer converts tool names at install time. Runtime differences in agent behavior are handled in workflow spawn blocks, not agent files.

---

## Build Order (Suggested)

Build order based on dependencies and validation opportunities:

### Phase A: Foundation (Validate Detection Works)

1. **Runtime detection in gsd-tools.cjs**
   - Add `runtime` to `loadConfig()`
   - Add env var checks, filesystem heuristics
   - Add `runtime` to all `init *` JSON payloads
   - Add `config.json` override support
   - Validate: Run `gsd-tools.cjs init execute-phase 1` on Claude Code, verify `"runtime": "claude-code"` in output

2. **Parallelization override in init JSON**
   - When `runtime != "claude-code"`, force `"parallelization": false` in init output
   - Existing sequential execution code in workflows already handles this correctly

### Phase B: OpenCode Parity (Most Users, Existing Install Base)

3. **Capability audit — document all Task call sites**
   - List every `Task(...)` call across all 30+ workflow files
   - Prioritize: `execute-phase.md`, `plan-phase.md`, `new-project.md` (highest-traffic workflows)

4. **Add inline fallback blocks to Task call sites (5 workflows)**
   - Start with `execute-phase.md` (core execution path)
   - Then `plan-phase.md` (planning path)
   - Then `new-project.md` (project init path)
   - Test on OpenCode after each workflow

5. **OpenCode-specific model defaults in gsd-tools.cjs**
   - Add default model table for OpenCode (likely OpenAI models if OpenCode routes to OpenAI)
   - Document in `runtime-compatibility.md`

### Phase C: Codex CLI Support (Net New)

6. **Codex CLI capability research** (phase-specific research needed — see gaps)
   - Verify: command format, tool names, agent file format, config directory
   - Verify: what env vars Codex CLI sets (for detection)
   - Verify: whether any subagent mechanism exists

7. **Codex CLI installer support in bin/install.js**
   - New `installCodexCLI()` function
   - Command format converter
   - Tool name mapping table
   - Config directory detection

8. **Validate Codex CLI end-to-end**
   - Install GSD on Codex CLI
   - Run `/gsd:new-project` through completion
   - Document any remaining gaps

---

## Integration Points

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Workflow → Runtime | Init JSON `runtime` field | Workflows never detect runtime directly |
| gsd-tools.cjs → Config | `config.json` `runtime` field | User override path |
| Installer → Runtime | `installCodexCLI()` function | New for Codex; existing for Claude/OpenCode |
| Workflow spawn site → Agent | Inline prose when no Task | Agent file read by orchestrator |
| Model resolution → Runtime | Default model table per runtime | In `MODEL_PROFILES` or separate table |

---

## Open Gaps (Require Phase-Specific Research)

### Gap 1: Codex CLI Command Format — UNKNOWN, CRITICAL

The installer needs to know the exact command file format for Codex CLI:
- File location (config directory path)
- Frontmatter schema (tool names, allowed-tools equivalent)
- Command namespace (does `/gsd:plan-phase` work or does it need conversion?)
- Whether agent files are supported at all

**Action:** Codex CLI capability audit must precede any Codex installer work.

### Gap 2: OpenCode Subagent Mechanism — UNKNOWN

OpenCode may have added a subagent spawning mechanism after GSD's existing support was written. The installer code predates any such feature. Verify:
- Does OpenCode have a `skill` tool that enables sub-workflow invocation?
- Does it have any parallel execution primitive?
- Does it support model selection per invocation?

**Action:** Read current OpenCode documentation before deciding on inline fallback vs. native mechanism.

### Gap 3: Codex CLI Env Vars for Detection — UNKNOWN

Without knowing what env vars Codex CLI sets, the detection heuristic for "this is Codex CLI" cannot be implemented.

**Action:** Find Codex CLI documentation on process environment.

### Gap 4: Tool Name Mapping for Codex CLI — UNKNOWN

Claude Code tools (Read, Write, Bash, Grep, Glob, AskUserQuestion) may have different names in Codex CLI. The installer's `convertClaudeToOpencodeFrontmatter()` pattern would need a Codex-equivalent function.

**Action:** Same Codex CLI capability audit as Gap 1.

---

## Confidence Assessment

| Area | Confidence | Reason |
|------|------------|--------|
| Task tool as gap | HIGH | Direct code inspection confirms Task calls throughout workflows |
| OpenCode install-level support | HIGH | Installer code directly inspected |
| OpenCode workflow gap | HIGH | No Task equivalent found in OpenCode docs or installer |
| Inline fallback pattern | HIGH | Pattern is straightforward and runtime-independent |
| Runtime detection via gsd-tools | HIGH | Detection centralization is the right pattern regardless of runtime specifics |
| Codex CLI specifics | LOW | Based on training data only; Codex CLI evolves rapidly |
| OpenCode subagent mechanism | LOW | May exist in current version; not confirmed from inspection |

---

## Sources

- GSD codebase: `/Users/thelorax/.claude/get-shit-done/` (direct inspection, HIGH confidence)
- GSD installer: `bin/install.js` — multi-runtime support referenced in `INTEGRATIONS.md` (HIGH confidence)
- GSD codebase map: `.planning/codebase/INTEGRATIONS.md`, `.planning/codebase/ARCHITECTURE.md` (HIGH confidence)
- Codex CLI specifics: Training data only (LOW confidence — verify before implementation)
- OpenCode subagent capability: Not confirmed — needs verification before Phase B implementation

---

*Architecture research for: GSD v1.1 — Codex CLI + OpenCode multi-runtime support*
*Researched: 2026-02-16*

<token_report>
dimensions: architecture
model: claude-sonnet-4-5-20250929
input_tokens: 92000
output_tokens: 4800
estimated_cost_usd: 0.3480
note: estimated from session context; actual may vary +/-20%
</token_report>
