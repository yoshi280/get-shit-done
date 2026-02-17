# Feature Research: GSD Multi-Runtime Parity (v1.1)

**Domain:** Multi-runtime AI developer tool — bringing GSD workflow parity to Codex CLI and OpenCode
**Researched:** 2026-02-16
**Confidence:** HIGH for OpenCode gaps (sourced from codebase); LOW-MEDIUM for Codex CLI (training data + architecture inference)

## Context: What "Parity" Means for GSD

GSD's value comes from agent orchestration: parallel researchers, planning/checking revision loops, wave-based execution, post-execution verification. All of this runs through Claude Code's `Task` tool, which spawns fresh agents with independent 200k context windows.

On OpenCode: install-level support exists (frontmatter conversion, path remapping, tool name mapping). The Task tool is available with `subagent_type="general"` for generic spawns. Named subagent types (`gsd-executor`, `gsd-planner`, `gsd-verifier`, etc.) are passed through unchanged — behavior on OpenCode is unverified.

On Codex CLI: zero GSD support. Codex CLI uses OpenAI's model and has a different execution model (sandbox-based code execution, no Task-tool-equivalent for spawning parallel agents in fresh contexts).

Full parity = the same agent orchestration quality, the same questioning flows, the same token observability, and explicit notification when something must degrade.

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume work when GSD is installed on a runtime. Missing any of these makes the runtime experience feel broken, not just degraded.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| **Named subagent type routing on OpenCode** | GSD workflows spawn `gsd-executor`, `gsd-planner`, `gsd-verifier`, etc. by name. OpenCode only converts `general-purpose` → `general`; named types pass through unverified. If OpenCode doesn't resolve named agents, every `execute-phase`, `plan-phase`, and `verify-work` silently breaks. | MEDIUM | Audit which named types work, fix installer conversion or add `general` fallback with agent body inlined. Sourced from installer code analysis (HIGH confidence). |
| **AskUserQuestion / interactive questioning flows** | Users expect `/gsd:new-project` questioning to work — the step-by-step discovery flow is GSD's first impression. OpenCode maps `AskUserQuestion` → `question` at install time. Codex CLI has no equivalent — the tool doesn't exist. | LOW (OpenCode), HIGH (Codex) | OpenCode conversion exists but needs validation. Codex CLI needs full fallback strategy — inline multi-choice markdown OR sequential text-based questions. |
| **SlashCommand / workflow invocation across runtimes** | GSD auto-advances with `SlashCommand("/gsd:discuss-phase 1 --auto")`. OpenCode maps this to `skill`. Codex CLI: unknown. Without this, auto-advance chains break silently. | LOW (OpenCode), MEDIUM (Codex) | For Codex CLI, inline the target workflow content rather than invoking via SlashCommand. |
| **Config path resolution for all runtimes** | Workflows hardcode `~/.claude/get-shit-done/...` paths. OpenCode installer replaces `~/.claude` → `~/.config/opencode`. Codex CLI has no install-time replacement — all path references are wrong. | MEDIUM | Codex CLI needs path substitution pass at install time. Add `~/.codex/` path target. Or switch workflows to use a runtime-resolved path variable. |
| **gsd-tools.cjs availability on all runtimes** | Every workflow calls `node ~/.claude/get-shit-done/bin/gsd-tools.cjs`. Path is hardcoded. If not replaced at install time on Codex CLI, every `init` call fails silently. | LOW | Same problem as config path resolution. Install-time substitution or runtime detection via env var. |
| **Runtime detection** | GSD needs to know which runtime it's in so it can route to correct paths, tool names, and fallback behaviors. Currently no detection exists — all workflows assume Claude Code conventions. | LOW | Detect via environment variable or config flag set at install time. Write to `.planning/config.json` as `runtime: "claude"|"opencode"|"codex"`. |
| **Parallel research spawning on all runtimes** | `/gsd:new-project` spawns N researchers in parallel via `Task(..., run_in_background=true)`. If `run_in_background` isn't supported on a runtime, parallel spawning silently becomes sequential or fails entirely. | HIGH | Verify `run_in_background` behavior on OpenCode. For Codex CLI: sequential fallback with explicit notice. Don't silently drop parallel work. |
| **Plan-checker revision loop on all runtimes** | `plan-phase` spawns planner → checker → planner (max 3 iterations). This requires Task tool to block until agent returns. If async/await semantics differ, loops either skip or loop forever. | MEDIUM | Verify Task blocking behavior on OpenCode with named agent types. Codex CLI needs inline revision logic as fallback. |

### Differentiators (Competitive Advantage)

Features that make the multi-runtime experience excellent, not just functional.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| **Runtime capability manifest** | A machine-readable file (`.planning/runtime-capabilities.json`) documenting what the current runtime supports: parallel spawning, named agents, interactive questions, background tasks. Every workflow reads this and adjusts behavior accordingly. Other multi-runtime tools (Aider, Continue) hard-code runtime branches — a capability manifest is more maintainable. | MEDIUM | Write once at install time or first run. Workflows check capabilities before using advanced features. |
| **Explicit degradation notices** | When a workflow falls back from parallel → sequential, or from named agent → inline logic, it prints a visible notice: `[Codex] Parallel research unavailable — running 4 dimensions sequentially (~3x slower)`. Users understand the tradeoff instead of wondering why something seems slower. | LOW | Small addition to each fallback branch. High UX value. |
| **Runtime-aware token budgeting** | Different runtimes use different models with different context windows and cost profiles. Codex CLI uses GPT-4o (128k context). OpenCode supports multiple providers. A runtime-aware token budget prevents context overflow on smaller-context models. | MEDIUM | Extend existing token observability (v1.0 feature) to be runtime-aware. Read context window size from runtime config. |
| **Inline agent fallback** | When `subagent_type="gsd-executor"` can't be resolved by a runtime, automatically inline the agent's body into the Task prompt. This preserves agent behavior without requiring named agent registration. | HIGH | Requires the installer (or a runtime wrapper) to embed agent file content into workflow prompts at install time. Significant installer change. |
| **Sequential execution mode as first-class config** | Make `parallelization: false` in `config.json` fully supported and well-tested for all runtimes, not just a fallback. Codex CLI users would set this and get a clean sequential experience. | LOW | Config already exists. Ensure all workflows respect it consistently. Add to Codex install wizard as default. |
| **Cross-runtime test suite** | Automated validation that core GSD workflows produce the same artifacts on all supported runtimes. Missing artifacts or wrong file contents flag regressions. | HIGH | Would require test environment for each runtime. Likely out of scope for v1.1 but worth flagging as future work. |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem helpful but make the multi-runtime experience worse.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| **Silent fallback without notice** | "Don't break the workflow, just do the best you can" | Users can't tell if they're getting full GSD quality or a degraded version. Builds false confidence. They file bugs for behavior that is working as intended. | Always print a one-line notice when falling back. Make degradation visible. |
| **Full feature parity on Codex CLI via workarounds** | "Support everything on every runtime" | Codex CLI's execution model is fundamentally different. Trying to emulate parallel agent spawning via sequential loops with fake parallelism adds complexity and breaks GSD's lean-orchestrator design principle. | Define an honest Codex CLI feature set. Implement it cleanly. Don't pretend it has Task-tool parity it doesn't have. |
| **Runtime auto-detection via heuristics** | "Detect runtime at prompt-time without explicit config" | Heuristics break. Tool availability changes between runtime versions. A heuristic that worked in Codex v1.0 fails in v1.1. Results in hard-to-diagnose behavior. | Set `runtime` in `config.json` at install time. Explicit is reliable. |
| **One unified workflow file for all runtimes** | "Fewer files to maintain" | A single file with `if runtime == claude` conditionals in markdown is unmaintainable. Markdown conditionals aren't a real language. | Install-time conversion (installer already does this for OpenCode). Maintain runtime-specific variants generated from a shared source. |
| **Gemini CLI parity in v1.1** | "Do all three runtimes at once" | PROJECT.md explicitly defers Gemini to v1.2+. Gemini has a fundamentally different agent model (TOML files, auto-registered tools). Adding Gemini to v1.1 scope inflates complexity by 50% without proportional value. | Defer. The installer already handles Gemini frontmatter conversion. Validate separately after Codex + OpenCode are stable. |

## Feature Dependencies

```
Runtime Detection
    └──required by──> Named Subagent Routing
    └──required by──> Config Path Resolution
    └──required by──> Runtime Capability Manifest
    └──required by──> Explicit Degradation Notices

Named Subagent Routing
    └──required by──> Plan-Checker Revision Loop
    └──required by──> Execute-Phase Wave Execution
    └──required by──> Verify-Phase Agent

Runtime Capability Manifest
    └──enables──> Runtime-Aware Token Budgeting
    └──enables──> Inline Agent Fallback
    └──enhances──> Explicit Degradation Notices

Parallel Research Spawning
    └──requires──> run_in_background support (runtime-dependent)
    └──falls back to──> Sequential Execution Mode

AskUserQuestion Fallback
    └──required by──> /gsd:new-project questioning flow
    └──required by──> Dimension selection (v1.0 feature)
    └──required by──> All interactive workflow gates
```

### Dependency Notes

- **Runtime Detection required by everything:** No other multi-runtime feature can work without knowing which runtime GSD is running in. This is Phase 1, item 1.
- **Named subagent routing blocks execute-phase and plan-phase:** If OpenCode doesn't resolve named agent types, the two most important GSD workflows are broken. This is the highest-priority OpenCode gap.
- **Inline agent fallback is a late-dependency feature:** It depends on runtime detection AND understanding which named agent types fail on which runtimes. Build after capability audit, not before.
- **Parallel research depends on runtime capability manifest:** The manifest tells workflows whether to attempt parallel spawning or go sequential. Without the manifest, workflows would need per-runtime conditional logic hardcoded into every file.

## MVP Definition

### Launch With (v1.1)

Minimum viable multi-runtime support — what's needed for OpenCode and Codex CLI to be usable, not just installable.

- [ ] **Runtime detection + config.json `runtime` field** — All other features depend on this. Write at install time. Read in every workflow that needs to branch. (LOW complexity, HIGH unlock value)
- [ ] **Named subagent type audit on OpenCode** — Verify whether `gsd-executor`, `gsd-planner`, `gsd-verifier`, etc. work on OpenCode. Document which ones fail. Required before any fixes. (LOW complexity, HIGH diagnostic value)
- [ ] **Named subagent type fix for OpenCode** — If audit shows failures, add installer conversion: `subagent_type="gsd-executor"` → `subagent_type="general"` with agent body inlined in prompt. (MEDIUM complexity)
- [ ] **Codex CLI install support** — Add `--codex` flag to installer. Path substitution (`~/.claude` → `~/.codex` or wherever Codex stores config). Frontmatter conversion for tool names. (MEDIUM complexity)
- [ ] **AskUserQuestion fallback for Codex CLI** — Codex CLI has no `question` tool. Fallback: print options as numbered list, expect numbered response, parse. Keeps questioning flow functional. (MEDIUM complexity)
- [ ] **Explicit degradation notices** — Every fallback branch prints a one-liner. (LOW complexity, essential for user trust)
- [ ] **Sequential execution mode validation** — Ensure `parallelization: false` in config.json works end-to-end on Codex CLI. This is the baseline execution model for a runtime without parallel Task spawning. (LOW complexity)

### Add After Validation (v1.x)

Features to add once v1.1 baseline is working and users are hitting the limits.

- [ ] **Runtime capability manifest** — Formalize the ad-hoc capability branches into a declarative manifest. Add when there are 3+ capability branches needing coordination. (MEDIUM complexity)
- [ ] **Runtime-aware token budgeting** — Extend v1.0 token observability with context window awareness. Add when Codex CLI users report context overflow. (MEDIUM complexity)
- [ ] **`run_in_background` validation on OpenCode** — Verify parallel research works on OpenCode. If it doesn't, implement sequential fallback with notice. (MEDIUM complexity, depends on having OpenCode test environment)
- [ ] **Inline agent fallback** — Embed agent body into Task prompts as fallback. Add when named agent type fix (above) proves insufficient. (HIGH complexity)

### Future Consideration (v2+)

- [ ] **Cross-runtime test suite** — Automated artifact validation across runtimes. Requires test infrastructure investment that's out of scope until multi-runtime stability is proven. (HIGH complexity)
- [ ] **Gemini CLI parity** — Explicitly deferred to v1.2+ per PROJECT.md. Gemini has unique TOML/auto-register model that needs its own design pass.
- [ ] **Runtime-specific model profiles** — Different default model selections per runtime based on what's available. Useful once usage data shows per-runtime cost patterns.

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority | v1.0 Dependency |
|---------|------------|---------------------|----------|-----------------|
| Runtime detection | HIGH | LOW | P1 | None |
| Named subagent audit (OpenCode) | HIGH | LOW | P1 | None |
| Named subagent fix (OpenCode) | HIGH | MEDIUM | P1 | Audit results |
| Codex CLI install support | HIGH | MEDIUM | P1 | None |
| AskUserQuestion fallback (Codex) | HIGH | MEDIUM | P1 | Codex install |
| Explicit degradation notices | HIGH | LOW | P1 | Runtime detection |
| Sequential mode validation | MEDIUM | LOW | P1 | Codex install |
| SlashCommand fallback (Codex) | MEDIUM | MEDIUM | P2 | Codex install |
| Runtime capability manifest | MEDIUM | MEDIUM | P2 | All P1 items |
| `run_in_background` audit (OpenCode) | MEDIUM | LOW | P2 | OpenCode environment |
| Runtime-aware token budgeting | LOW | MEDIUM | P2 | v1.0 observability |
| Inline agent fallback | MEDIUM | HIGH | P2 | Capability manifest |
| Cross-runtime test suite | HIGH | HIGH | P3 | Stable multi-runtime |
| Gemini CLI parity | MEDIUM | HIGH | P3 | None (deferred) |

**Priority key:**
- P1: Must have for v1.1 — makes runtime actually usable
- P2: Should have — makes it excellent, add when P1 is stable
- P3: Future consideration — deferred per PROJECT.md or out of scope

## How Other Multi-Runtime AI Tools Handle This

**Aider** (multi-provider, single execution model): Aider side-steps the problem by not having an agent orchestration layer. It runs the model directly and doesn't spawn subagents. This makes multi-runtime support trivial (just swap model API) but also means it can't do parallel work. GSD's approach is more powerful but harder to make portable.

**Continue.dev** (VS Code extension, multiple LLM backends): Uses provider abstraction at the API level, not at the agent orchestration level. Each provider has a standardized interface for chat + context. No subagent spawning. Same limitation as Aider — simpler to port but less capable for autonomous work.

**The lesson from both:** Tools that sidestep subagent spawning are easier to port but fundamentally less capable. GSD should not sacrifice its orchestration model to achieve portability. Instead, it should define a capability tier system: Tier 1 (full Task-tool support = Claude Code), Tier 2 (partial = OpenCode), Tier 3 (sequential only = Codex CLI) — and document what each tier provides.

## Competitor Runtime Capability Comparison

| Capability | Claude Code | OpenCode | Codex CLI |
|-----------|-------------|----------|-----------|
| Task tool (agent spawn) | Native | `general` subagent (verified), named types (unverified) | Not available (LOW confidence) |
| AskUserQuestion | Native | `question` (installer converts) | Not available (LOW confidence) |
| SlashCommand | Native | `skill` (installer converts) | Not available (LOW confidence) |
| run_in_background | Native | Unverified (MEDIUM confidence) | Not available (LOW confidence) |
| Named agent routing | Native | Unverified (MEDIUM confidence) | Not available (LOW confidence) |
| MCP tools | Native | Supported | Limited (LOW confidence) |
| Config directory | `~/.claude/` | `~/.config/opencode/` | `~/.codex/` (LOW confidence) |
| Model context window | 200k | Provider-dependent | 128k (GPT-4o, LOW confidence) |
| Current GSD support | Full | Install-level | None |

**Confidence notes:** All Codex CLI entries are LOW confidence from training data (cutoff January 2025). OpenCode named agent routing is MEDIUM confidence — installer converts `general-purpose` → `general` but named types like `gsd-executor` are passed through unchanged and OpenCode behavior is undocumented in the codebase. Capability audit (P1 feature above) is needed to replace these LOW/MEDIUM entries with verified data.

## Sources

- GSD installer source code: `/Users/thelorax/get-shit-done/bin/install.js` — lines 307-482 (tool name mappings, frontmatter conversion, subagent type conversion) — HIGH confidence
- GSD workflow files: `/Users/thelorax/.claude/get-shit-done/workflows/` — subagent_type usage audit — HIGH confidence
- GSD CHANGELOG.md: runtime support history (OpenCode support added v1.19, named subagent type conversion noted, `run_in_background` not mentioned) — HIGH confidence
- GSD INTEGRATIONS.md: config directory locations per runtime — HIGH confidence
- Aider architecture: training knowledge (multi-provider, no subagent spawning) — MEDIUM confidence (pre-Jan 2025 knowledge)
- Continue.dev architecture: training knowledge — MEDIUM confidence (pre-Jan 2025 knowledge)
- Codex CLI capabilities: training knowledge (OpenAI Codex CLI released 2025, sandbox-based model) — LOW confidence, needs verification against actual Codex CLI docs

---
*Feature research for: GSD v1.1 — Codex CLI and OpenCode runtime parity*
*Researched: 2026-02-16*
*Confidence: HIGH for OpenCode gaps (from codebase); LOW for Codex CLI specifics (training data)*
