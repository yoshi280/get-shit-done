# Project Research Summary

**Project:** GSD v1.1 — Codex CLI + OpenCode Multi-Runtime Support
**Domain:** Multi-runtime AI agent orchestration — porting a Claude Code-specific meta-prompting framework to additional runtimes
**Researched:** 2026-02-16
**Confidence:** MEDIUM (OpenCode: HIGH from SDK inspection; Codex CLI: LOW from training data only)

## Executive Summary

GSD is a layered agent-orchestration system built entirely on Claude Code's `Task()` tool, which spawns isolated subagents with fresh 200k context windows in parallel. The v1.1 milestone adds support for OpenCode and Codex CLI, but these runtimes have fundamentally different execution models. OpenCode has a `task` tool with a `SubtaskPart` mechanism (confirmed in SDK v1.1.53 types), but it requires agents to be registered in `opencode.json` rather than as markdown files — meaning GSD's named agent types (`gsd-executor`, `gsd-planner`, etc.) currently pass through the installer unresolved and silently fail on OpenCode. Codex CLI has no subagent spawning, no slash command system, and no named agent concept at all, making it structurally incompatible with GSD's wave-based parallel execution model.

The recommended approach is a Runtime Adapter Pattern: shared workflow files with runtime-conditional prose blocks that branch between `Task()` calls (Claude Code) and inline execution (OpenCode/Codex). Runtime detection is centralized in `gsd-tools.cjs` and surfaced via the `runtime` field in all init JSON payloads — workflows never detect runtime themselves. For OpenCode, the installer must parse agent markdown files and inject system prompts into `opencode.json` as `AgentConfig` entries. For Codex CLI, the v1.1 target is a degraded-but-functional sequential mode with explicit user notification of capability limitations; full parity is architecturally deferred.

The primary risks are silent degradation (fallbacks that run without notifying the user), model-compliance variability (GPT-4o through OpenCode follows GSD's dense XML-tagged workflow markdown less reliably than Claude), and Codex CLI's sandboxed execution environment blocking access to `gsd-tools.cjs` at its hardcoded absolute path. These risks must be addressed in the foundation phase before any workflow porting begins — they affect every subsequent phase.

---

## Key Findings

### Recommended Stack

See full details: `.planning/research/STACK.md`

The implementation requires no new runtime dependencies. All changes live in the existing three artifacts: `bin/install.js` (new Codex installer section + OpenCode agent registration), `gsd-tools.cjs` (runtime detection + model ID resolver), and the ~5 workflow markdown files that contain `Task()` calls (conditional spawn blocks added).

**Core components to build:**

- **Runtime detector** (`gsd-tools.cjs` `loadConfig()`): Multi-signal heuristic (env vars + filesystem), outputs `runtime: "claude-code"|"opencode"|"codex-cli"` into all init JSON payloads. Manual override via `config.json`.
- **OpenCode agent installer** (`bin/install.js` extension): Parse `agents/gsd-*.md` frontmatter and system prompt, inject as `AgentConfig` entries into `opencode.json`. This is the critical gap for OpenCode parity.
- **Model ID resolver** (`gsd-tools.cjs` new command): Map GSD tier names (`opus`, `sonnet`, `codex`) to provider/model strings for OpenCode (e.g., `anthropic/claude-opus-4-6`).
- **Codex CLI installer** (`bin/install.js` new section): Path substitution (`~/.claude` to project-relative or `~/.codex`), tool name mapping, command format conversion.

**What NOT to build in v1.1:**
- OpenCode plugin SDK integration (requires Bun/TypeScript — violates zero-dependency constraint)
- Full Codex CLI parity (single-agent architecture is fundamentally incompatible with wave-based orchestration)
- Gemini CLI support (explicitly deferred to v1.2+ per PROJECT.md)

**Critical version note:** OpenCode SDK is v1.1.53 (confirmed locally). `AgentConfig.model` accepts `"provider/model"` format strings, not GSD tier names.

### Expected Features

See full details: `.planning/research/FEATURES.md`

**Must have (table stakes for v1.1):**

- **Runtime detection + `config.json` runtime field** — foundational dependency for all other multi-runtime features; must be item 1
- **Named subagent type audit on OpenCode** — determine whether `gsd-executor`, `gsd-planner`, etc. resolve; required before writing any fixes
- **Named subagent type fix for OpenCode** — installer must inject agent system prompts into `opencode.json`; highest-priority OpenCode gap
- **Codex CLI install support** — `--codex` flag in installer, path substitution, tool name mapping
- **AskUserQuestion fallback for Codex CLI** — print numbered options, parse selection; without this, `/gsd:new-project` breaks at first interactive gate
- **Explicit degradation notices** — every fallback branch prints a visible one-liner before running; non-negotiable for user trust
- **Sequential execution mode validation** — verify `parallelization: false` in `config.json` works end-to-end on Codex CLI

**Should have (v1.x after validation):**
- Runtime capability manifest (`.planning/runtime-capabilities.json`) — formalize ad-hoc branches into declarative manifest
- `run_in_background` verification on OpenCode — parallel research may or may not work; test before claiming it does
- Runtime-aware token budgeting — context window differences between Claude (200k) and GPT-4o (128k) affect plan depth

**Defer (v2+):**
- Cross-runtime automated test suite (high infrastructure investment, out of scope until multi-runtime is stable)
- Gemini CLI parity (PROJECT.md explicit deferral)
- Runtime-specific model profiles (useful once per-runtime cost data exists)

**Anti-features to reject:**
- Silent fallback without user notification (destroys user trust in runtime capability claims)
- Runtime-specific workflow file variants (90+ files for 3 runtimes; unmaintainable)
- Runtime detection via single env var with no manual override (breaks across shell environments)

### Architecture Approach

See full details: `.planning/research/ARCHITECTURE.md`

The architecture adds a thin runtime detection + spawn adapter layer on top of GSD's existing five-layer stack (Command → Orchestrator → Agent → State Tool → State). This does NOT introduce runtime-specific workflow files — it adds conditional prose blocks to the ~5 existing workflow files that contain `Task()` calls. All artifact outputs (PLAN.md, SUMMARY.md, STATE.md) remain identical across runtimes; only the execution mechanism differs.

**Major components and responsibilities:**

1. **Runtime Detector** (`gsd-tools.cjs loadConfig()`) — Centralized detection via layered heuristics; exposes `runtime` field in all init JSON. Workflows never detect runtime themselves.
2. **Spawn Adapter** (conditional prose in workflow markdown) — Claude Code path: `Task(subagent_type=..., model=..., prompt=...)`; non-Claude path: read agent markdown file and execute inline in orchestrator context.
3. **Effective Parallelization Override** (`gsd-tools.cjs init *`) — Forces `parallelization: false` in init JSON for non-Claude runtimes. Existing sequential execution code in workflows already handles this correctly.
4. **OpenCode Agent Installer** (`bin/install.js`) — Parses `agents/gsd-*.md`, embeds system prompts as `AgentConfig.prompt` in `opencode.json`.
5. **Codex CLI Installer** (`bin/install.js`) — New `installCodexCLI()` function; tool name mapping; project-relative path substitution.
6. **Model Routing Table** (`gsd-tools.cjs MODEL_PROFILES`) — Runtime-aware defaults mapping GSD tier names to provider/model strings.

**Files requiring changes:** `gsd-tools.cjs` (~20-30 lines in `loadConfig()` and init handlers), `bin/install.js` (new Codex section + OpenCode agent injection), and 5 workflow files: `new-project.md`, `plan-phase.md`, `execute-phase.md`, `new-milestone.md`, `research-phase.md`.

**Patterns to follow:**
- Capability-conditioned prose in workflow markdown (not runtime-specific files)
- Runtime field from init JSON (not inline detection in workflows)
- Effective parallelization override in init layer (not per-workflow flag checks)

**Patterns to reject:**
- Runtime-specific workflow file variants (`execute-phase-opencode.md` style)
- Runtime detection scattered across workflow prose
- Assuming Task tool availability without explicit runtime check

### Critical Pitfalls

See full details: `.planning/research/PITFALLS.md`

1. **Task tool has no universal equivalent** — Every `Task()` call site must have a runtime-aware wrapper with an inline fallback before any workflow porting begins. Build the abstraction layer first. Warning sign: any workflow that calls `Task()` without a runtime check.

2. **Silent degradation without user notification** — Every fallback path must emit a specific terminal warning stating what capability is missing, what the fallback does, and how much slower it will be. Treat this as a contract, not polish.

3. **Runtime detection returning wrong answers** — Single-signal detection breaks across shell environments. Use multi-signal detection, require signals to agree, fail loudly on conflict, and always provide a `config.json` manual override.

4. **Codex CLI sandboxing blocks gsd-tools.cjs paths** — GSD workflows hardcode `node /Users/thelorax/.claude/get-shit-done/bin/gsd-tools.cjs`. Codex CLI's sandbox may block access to user-home absolute paths. Use project-relative paths for Codex workflows, or require and document `--full-auto` mode explicitly.

5. **OpenCode's model-agnostic design creates variable compliance** — OpenCode proxies Anthropic, OpenAI, and Google models. GPT-4o skips nested conditionals, compresses multi-step procedures, and produces non-compliant structured returns. GSD on OpenCode must check the active model at session start and warn when non-Claude is active.

6. **Markdown compliance varies dramatically across models** — GSD's workflow markdown is a Claude dialect, not generic markdown. Workflows ported to Codex CLI must be tested end-to-end on each target model. GPT-4o output looks plausible but may silently skip steps.

7. **`classifyHandoffIfNeeded` handling is Claude Code-specific** — GSD's post-hoc spot-check pattern for detecting false failures must NOT be copied verbatim to Codex/OpenCode adapters. Map each runtime's completion signal model before writing failure handling.

---

## Implications for Roadmap

### Phase 1: Runtime Abstraction Foundation

**Rationale:** Every subsequent phase depends on knowing which runtime GSD is running in. This is a pure infrastructure phase with no workflow changes — get detection right before touching anything else. Both architecture and pitfalls research are unambiguous that this comes first.

**Delivers:**
- `runtime` field in `gsd-tools.cjs loadConfig()` with layered heuristic detection
- `runtime` field surfaced in all `init *` JSON payloads
- `parallelization` forced to `false` in init JSON for non-Claude runtimes
- Manual override via `config.json` `runtime` field
- Runtime-aware model routing table in `MODEL_PROFILES`
- `references/runtime-compatibility.md` documenting per-runtime capabilities

**Addresses:** Runtime detection (P1 feature), effective parallelization override
**Avoids:** Pitfall 3 (wrong runtime detection), foundation for Pitfall 1 (Task tool gap)

**Research flag:** Standard implementation patterns. No phase-specific research needed.

---

### Phase 2: OpenCode Workflow Parity

**Rationale:** OpenCode already has install-level support. The gap is workflow-level: named agent types fail silently because OpenCode ignores agent markdown files. This is the highest-impact gap for existing OpenCode users. Fix OpenCode before Codex CLI — it's a tractable gap (shared `task` tool infrastructure), whereas Codex CLI is an architectural redesign.

**Delivers:**
- OpenCode agent registration: installer parses `agents/gsd-*.md` and injects `AgentConfig` entries into `opencode.json`
- Inline fallback blocks added to 5 workflow files (`new-project.md`, `plan-phase.md`, `execute-phase.md`, `new-milestone.md`, `research-phase.md`)
- Capability audit: verify which named agent types work on OpenCode after installer fix
- Explicit degradation notices in every fallback branch
- Model check at session start: warn if non-Claude model is active in OpenCode
- Task tool behavior verification: confirm OpenCode's `task` tool blocks like Claude Code's `Task`

**Addresses:** Named subagent routing fix (P1), AskUserQuestion flow (OpenCode path), explicit degradation notices (P1)
**Avoids:** Pitfall 1 (inline fallback), Pitfall 2 (degradation notices required), Pitfall 5 (active model check)

**Research flag:** NEEDS RESEARCH. OpenCode's `task` tool parallel behavior is unverified. Run a `/gsd:research-phase` on OpenCode task tool semantics before finalizing the inline fallback design. Specifically: does it block until completion? Does it support `run_in_background`? Does it return structured output?

---

### Phase 3: Codex CLI Install Support

**Rationale:** Codex CLI has zero current GSD support. The install foundation (path substitution, tool name mapping, command format) must exist before any workflow work can be validated on Codex CLI.

**Delivers:**
- `--codex` installer flag in `bin/install.js`
- New `installCodexCLI()` function with tool name mapping and command format conversion
- Path substitution strategy for Codex CLI sandbox constraints
- `gsd-tools.cjs` accessibility strategy (project-relative paths or `--full-auto` requirement documented)
- AskUserQuestion fallback: numbered list output, numeric response parsing
- SlashCommand fallback: inline target workflow content

**Addresses:** Codex CLI install support (P1), AskUserQuestion fallback for Codex (P1)
**Avoids:** Pitfall 4 (Codex sandboxing — project-relative paths)

**Research flag:** NEEDS RESEARCH before implementation. Critical unknowns: exact Codex CLI command file format, env vars set at startup (needed for Phase 1 detection), tool name surface area, whether any subagent mechanism exists in current Codex CLI. STACK.md confidence on Codex CLI is LOW (training data, cutoff January 2025).

---

### Phase 4: Codex CLI Workflow Integration

**Rationale:** With the installer in place, validate end-to-end workflow execution on Codex CLI. This phase adds conditional prose blocks to workflow files for the Codex path and validates that sequential inline execution produces the same artifacts as Claude Code.

**Delivers:**
- Conditional spawn blocks in 5 workflow files for Codex CLI path (inline execution fallback)
- Sequential execution mode validated end-to-end on Codex CLI
- State resumption validated from cold start (no conversational history)
- Runtime-specific completion signal handling for Codex CLI (not copied from Claude Code handler)
- Explicit degradation notices in all Codex CLI fallback paths

**Addresses:** Sequential execution mode validation (P1), state resumption parity
**Avoids:** Pitfall 6 (markdown compliance — end-to-end testing required per model), Pitfall 7 (`classifyHandoffIfNeeded` not copied verbatim), Pitfall 8 (state management — cold start resumption tested)

**Research flag:** Prompt compliance testing required before this phase is closed. Must run `new-project`, `plan-phase`, and `execute-phase` end-to-end on Codex CLI and verify structured output at each step. Do not mark complete based on a single test run.

---

### Phase 5: Testing Infrastructure and Release Validation

**Rationale:** Multi-runtime support cannot be declared production-ready without per-runtime validation. The pitfalls research is explicit: testing is the most commonly skipped step and the most common source of post-release bugs. This phase builds minimal viable test coverage runnable in under 30 minutes per runtime.

**Delivers:**
- Integration test suite: happy path for `/gsd:new-project`, `/gsd:plan-phase`, `/gsd:execute-phase` on all three runtimes
- Pass/fail criteria based on artifact existence and structure (not subjective LLM output quality)
- Runtime capability matrix validated against actual behavior (replaces LOW/MEDIUM confidence entries)
- `run_in_background` behavior on OpenCode verified and documented
- "Looks Done But Isn't" checklist from PITFALLS.md verified for all runtimes

**Avoids:** Pitfall 9 (multi-runtime testing skipped — this phase exists specifically to prevent that)

**Research flag:** Standard testing patterns. No research needed. Do not skip or defer this phase.

---

### Phase Ordering Rationale

- Phase 1 must come first: runtime detection is a foundational dependency for every other feature. Nothing else can be built correctly without it.
- Phase 2 (OpenCode) before Phase 3-4 (Codex): OpenCode has an existing install base and a tractable gap. Codex CLI has an architectural gap and critical unknowns requiring verification. Ship OpenCode parity first.
- Phase 3 (Codex install) before Phase 4 (Codex workflows): The installer resolves path/sandbox questions that affect workflow design. Cannot validate workflows without an install.
- Phase 5 last: validates all prior phases. Must not be deferred to post-release.

### Research Flags

**Phases needing `/gsd:research-phase` during planning:**

- **Phase 2 (OpenCode parity):** OpenCode `task` tool parallel behavior is unverified. STACK.md confirms the tool exists but does not confirm blocking semantics or `run_in_background` support. This directly affects the inline fallback design decision.
- **Phase 3 (Codex CLI install):** ALL Codex CLI specifics are LOW confidence (training data, cutoff January 2025). Command format, env vars, tool names, and any subagent mechanism must be verified from current official Codex CLI documentation before the installer can be written.

**Phases with standard patterns (skip research-phase):**

- **Phase 1 (Runtime detection):** Detection heuristics and init JSON extension patterns are well-understood. Implementation scope is clear from architecture research.
- **Phase 4 (Codex workflows):** Extends the inline fallback pattern established in Phase 2. No new architectural concepts required.
- **Phase 5 (Testing):** Standard integration testing patterns. Scope defined by pitfalls research.

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack (OpenCode) | HIGH | Sourced from local SDK inspection (`@opencode-ai/sdk` v1.1.53, `@opencode-ai/plugin` v1.1.53) and GSD installer code |
| Stack (Codex CLI) | LOW | Training data only, knowledge cutoff January 2025. Codex CLI was open-sourced April 2025 and has likely evolved. Web verification required. |
| Features | HIGH (OpenCode), LOW-MEDIUM (Codex) | OpenCode gaps sourced from codebase analysis. Codex CLI feature set inferred from architecture + training data. |
| Architecture | HIGH | Pattern recommendations derived from direct GSD codebase inspection. Runtime-specific details for Codex CLI are LOW confidence. |
| Pitfalls | MEDIUM | OpenCode pitfalls grounded in SDK + codebase analysis. Codex CLI pitfalls are training-data inferences, not empirically verified. |

**Overall confidence:** MEDIUM — High confidence on OpenCode path (tractable, well-sourced). Low confidence on Codex CLI specifics (unverified, training data only). Phase 3 research gate is required before Codex CLI implementation begins.

### Gaps to Address

- **Codex CLI command format:** Unknown. Exact file format, location, and frontmatter schema needed for installer. Blocks Phase 3 implementation.
- **Codex CLI env vars for runtime detection:** Unknown. What environment variables does Codex CLI set at startup? Required for Phase 1 detection heuristics to correctly identify Codex CLI.
- **OpenCode `task` tool parallel behavior:** Unconfirmed. Does `run_in_background` work? Does the `task` tool block until subagent completion? Required before Phase 2 inline fallback design is finalized.
- **OpenCode markdown agent file handling:** SDK shows no markdown agent loading path, but this is a negative claim that needs official documentation to confirm before the `opencode.json` injection approach is committed.
- **GPT-4o compliance with GSD workflow markdown:** Assumed poor based on training-data knowledge of GPT-4o instruction following, but not empirically tested against GSD's specific format. Must be validated during Phase 4 testing.

---

## Sources

### Primary (HIGH confidence)

- `~/.config/opencode/node_modules/@opencode-ai/sdk/` v1.1.53 — `AgentConfig`, `SubtaskPart`, `Command` types
- `~/.config/opencode/node_modules/@opencode-ai/plugin/` v1.1.53 — Plugin API, tool definitions, lifecycle hooks
- `~/get-shit-done/bin/install.js` — Multi-runtime installer, tool name mappings, frontmatter conversion logic
- `~/.claude/get-shit-done/workflows/` — Task tool usage patterns across all GSD workflows
- `~/.planning/codebase/ARCHITECTURE.md` and `INTEGRATIONS.md` — GSD architecture and runtime integration map

### Secondary (MEDIUM confidence)

- GSD CHANGELOG.md — Runtime support history; OpenCode support added v1.19
- GSD INTEGRATIONS.md — Config directory locations per runtime
- Training data: GPT-4o instruction-following behavior with XML-structured prompts
- Training data: OpenCode architecture (multi-model support, sst.dev terminal tool)
- Aider and Continue.dev architecture: multi-provider without subagent spawning (MEDIUM confidence, pre-Jan 2025)

### Tertiary (LOW confidence — verify before implementing)

- Training data: Codex CLI capabilities — slash command support, tool API surface, subagent capability, sandbox model, env vars. Knowledge cutoff January 2025; Codex CLI was released April 2025 and has evolved. All Codex CLI specifics need web verification before Phase 3 implementation.

---

*Research completed: 2026-02-16*
*Dimensions researched: 4 of 6 (STACK, FEATURES, ARCHITECTURE, PITFALLS)*
*Dimensions skipped: BEST-PRACTICES, DATA-STRUCTURES*
*Ready for roadmap: yes*
