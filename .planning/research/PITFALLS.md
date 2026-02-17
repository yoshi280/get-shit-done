# Pitfalls Research

**Domain:** Multi-Runtime AI Agent Tool Support — Adding Codex CLI and OpenCode to a Claude-Specific Meta-Prompting Framework
**Researched:** 2026-02-16
**Confidence:** MEDIUM (no web access; findings anchored to GSD codebase analysis + training-data knowledge of target runtimes)

---

## Critical Pitfalls

### Pitfall 1: The Task Tool Has No Universal Equivalent

**What goes wrong:**
GSD's entire parallel execution model depends on Claude Code's `Task()` call — spawn N agents simultaneously, each with a fresh 200k context, block until all complete, collect results. This is not a generic capability. Codex CLI and OpenCode have no direct equivalent. Attempting to port GSD's wave-based parallel orchestration (see `execute-phase.md`) without addressing this gap produces a system that either serializes all execution (severe performance regression) or crashes silently when the orchestrator tries to spawn subagents and nothing happens.

**Why it happens:**
The `Task` tool is so central to GSD that it appears unremarkable — it's just "how execution works." The gap only becomes visible when the runtime doesn't provide it. Developers assume "parallel subagents" is a general pattern, when it's actually a Claude Code-specific capability.

**How to avoid:**
Define a runtime capability abstraction layer before writing any Codex/OpenCode support. Map each GSD operation that calls `Task()` to one of three outcomes per runtime: (a) native equivalent exists and is used, (b) falls back to sequential execution with explicit user warning, (c) feature is unavailable and the workflow gracefully skips it. This mapping must be explicit in a capability registry file — not inferred at runtime from try/catch.

**Warning signs:**
- Any code that calls `Task()` without a runtime-aware wrapper
- Parallel wave execution working on Claude Code but producing no output or hanging on Codex CLI
- Tests that pass on Claude Code and are never run on Codex CLI
- Documentation that says "parallel execution" without noting it is Claude Code-only

**Phase to address:**
Phase 1 (Runtime Abstraction Layer) — Must exist before any workflow porting begins. Every subsequent phase builds on this.

---

### Pitfall 2: Silent Feature Degradation Without User Notification

**What goes wrong:**
A fallback that works but delivers a materially worse experience without telling the user is worse than an outright error. Example: GSD on Codex CLI silently serializes all wave execution, tripling wall-clock time. The user sees "execution complete" and has no idea the parallel wave system was bypassed. They blame slow hardware, not the runtime limitation. Worse: if the user explicitly chose Codex CLI expecting full parity, they've been misled.

**Why it happens:**
Fallback implementations feel like good engineering: "at least it doesn't crash." The problem is that degraded-but-silent is actually a reliability failure — the user cannot make informed decisions about which runtime to use for which workload.

**How to avoid:**
Every fallback must emit a visible, specific warning at the moment it activates — not buried in logs, not a one-time install notice. The warning must state: what capability is missing, what the degraded behavior is, and what the user can do (switch runtimes, adjust expectations). Example: `[GSD] Parallel wave execution not available on Codex CLI. Running 3 plans sequentially — estimated 3x longer.` This is non-optional; treat it as a contract.

**Warning signs:**
- Any fallback code path that does not include a user-visible warning
- Feature flags that disable capabilities without surfacing the disable to the user
- "Works on all runtimes" claims in documentation without listing per-runtime capability differences
- Capability detection that logs to a debug file rather than the active terminal session

**Phase to address:**
Phase 1 (Runtime Abstraction Layer) — The degradation warning system is part of the abstraction layer contract, not a later polish concern.

---

### Pitfall 3: Markdown Instruction Compliance Varies Dramatically Across Models

**What goes wrong:**
GSD workflows are written in a dense, structured markdown format with XML-tagged sections (`<step>`, `<task>`, `<objective>`), numbered steps, conditional logic ("If X: do Y, else: do Z"), and multi-level nesting. Claude follows this format reliably because it was trained extensively on instruction-following with this style. GPT-4o (used by Codex CLI) follows similar patterns but with meaningful differences: it tends to skip nested conditionals, compress multi-step procedures into fewer steps, and honor XML tags less consistently. Smaller or less capable models used by OpenCode may not follow this format at all.

**Why it happens:**
The GSD workflow files were written for Claude specifically. They use patterns that Claude's RLHF process has optimized for — not patterns that are universally portable. The failure mode is subtle: the model appears to be following instructions (output looks plausible) but is actually missing steps, skipping conditionals, or merging tasks that should be separate.

**How to avoid:**
Do not assume workflow markdown is model-agnostic. Treat GSD workflow files as Claude dialects and write separate, tested variants for each runtime. Start with the highest-stakes workflows (`execute-phase.md`, `execute-plan.md`) and validate step-by-step compliance on Codex CLI and OpenCode before considering any workflow "ported." Use explicit output format enforcement (JSON schemas for intermediate results where possible) rather than relying on model prose compliance.

**Warning signs:**
- Workflow ported to Codex CLI by simply passing the same .md file
- No per-runtime compliance tests for critical workflow steps
- GPT-4o output that looks reasonable but skips the self-check step in `gsd-executor`
- Checkpoint handling that works on Claude but produces no structured output on GPT-4o
- Any claim that "the model will follow the same instructions" without empirical verification

**Phase to address:**
Phase 2 (Workflow Translation) — Each workflow variant must be tested in isolation before integration.

---

### Pitfall 4: Runtime Detection That Returns Wrong Answers

**What goes wrong:**
"Detect runtime, adapt behavior" is the right architecture but the detection itself is fragile. Common failure modes: (1) Detection reads an environment variable that isn't set in all shell contexts (works in iTerm, fails in VS Code terminal). (2) Detection based on which binary is in PATH is fooled by wrappers or aliases. (3) Detection runs once at startup and is cached, but the user changes runtimes mid-session. (4) Detection succeeds but returns the wrong capability profile because the capability registry is out of date. The result: capability detection passes, the wrong code path runs, and the error looks like a GSD bug rather than a detection failure.

**Why it happens:**
Runtime detection feels like a solved problem — check which tool invoked the session and branch accordingly. In practice, Claude Code, Codex CLI, and OpenCode each have different invocation models, environment variable patterns, and extension/slash-command integration points. None of them document a stable, machine-readable "I am runtime X" signal that is guaranteed to be present.

**How to avoid:**
Use a multi-signal detection approach: check at least two independent signals (env var AND binary presence AND known tool API availability), require all signals to agree before committing to a capability profile, and fail loudly if signals conflict rather than guessing. Provide a manual override: `RUNTIME=codex-cli` or equivalent in `.planning/config.json` so users can correct misdetection without code changes. Log detection decisions with the signals that drove them.

**Warning signs:**
- Single-signal runtime detection (one env var or one `which` check)
- No manual override mechanism for runtime
- Detection result is never shown to the user — they cannot see what was detected
- Tests that mock the detection function rather than testing the detection logic
- Detection that silently falls back to "assume Claude Code"

**Phase to address:**
Phase 1 (Runtime Abstraction Layer) — Detection is the foundation; errors here corrupt everything downstream.

---

### Pitfall 5: The classifyHandoffIfNeeded Pattern Is Claude Code-Specific

**What goes wrong:**
GSD's `execute-phase.md` explicitly documents a known Claude Code bug: `classifyHandoffIfNeeded is not defined` fires after successful tool calls and causes agents to report failure despite completing all work. GSD handles this with post-hoc spot-checks (SUMMARY.md exists? Commits present? No Self-Check: FAILED marker?). This error handling pattern is Claude Code-specific. On Codex CLI, task completion signals work differently. On OpenCode, there may be no structured return value from sub-executions at all. Porting the spot-check logic without understanding the completion signal differences will produce false positives (treating genuine failures as the known bug) or miss failures entirely.

**Why it happens:**
Developers read the `classifyHandoffIfNeeded` special-case in the GSD code and assume it's a pattern they need to carry forward. They copy the spot-check logic into the Codex/OpenCode adapter without verifying whether the same bug exists, whether a different bug manifests, or whether the completion signaling model is fundamentally different.

**How to avoid:**
Map the completion signal model for each runtime before writing any failure-handling code. Document what "plan execution complete" looks like for each runtime — what data is returned, what the absence of data means, and what runtime-specific error strings indicate false failures vs. real failures. Write a runtime-specific failure classifier rather than a shared one, then test it against known failure scenarios on each platform.

**Warning signs:**
- `classifyHandoffIfNeeded` handling copied verbatim into a non-Claude Code adapter
- Failure handling that uses Claude-specific error string matching on other runtimes
- No runtime-specific integration tests for task completion detection
- Spot-check logic that runs on all runtimes but was only validated against Claude Code behavior

**Phase to address:**
Phase 2 (Workflow Translation) — Completion signaling must be addressed per-runtime before any plan execution is considered reliable.

---

### Pitfall 6: Codex CLI Sandboxing Breaks GSD's Bash Tool Usage

**What goes wrong:**
GSD workflows invoke `node /Users/thelorax/.claude/get-shit-done/bin/gsd-tools.cjs ...` extensively for state management, commit operations, progress tracking, and configuration. These are filesystem and subprocess operations. Codex CLI runs in a sandboxed execution environment — by default it restricts or requires approval for network calls and filesystem writes outside the project directory. The `gsd-tools.cjs` binary lives at an absolute path that may not be accessible within Codex CLI's sandbox. If it is accessible, the `--full-auto` mode approves all commands, but the default sandbox mode will prompt for approval on every gsd-tools call, destroying the autonomous execution model.

**Why it happens:**
Codex CLI's sandbox was designed for running code within a project, not for invoking external tool binaries at user-global paths. GSD's tool architecture assumes the executing agent can freely invoke `gsd-tools.cjs` without approval gates. These assumptions conflict at a fundamental level.

**How to avoid:**
Do not assume `gsd-tools.cjs` is accessible within Codex CLI's sandbox as-is. Either: (a) bundle a Codex-compatible version of the necessary tools within the project's `.planning/` directory, (b) use only project-relative paths in Codex CLI workflows, or (c) require `--full-auto` mode and document this requirement explicitly. Option (b) is safest long-term. Audit every `gsd-tools.cjs` call in every workflow file and identify which can be replaced with project-relative equivalents.

**Warning signs:**
- Codex CLI workflows that call `gsd-tools.cjs` with absolute `/Users/thelorax/` paths
- No documentation specifying required Codex CLI sandbox mode
- Testing Codex CLI integration only in `--full-auto` mode
- Any Codex CLI workflow that assumes filesystem access outside the project root

**Phase to address:**
Phase 1 (Runtime Abstraction Layer) and Phase 3 (Codex CLI Integration) — The path architecture decision must be made early because it affects every workflow.

---

### Pitfall 7: OpenCode's Model-Agnostic Design Creates Variable Compliance

**What goes wrong:**
OpenCode supports multiple model providers (Anthropic Claude, OpenAI GPT-4o, Google Gemini, and others). This means a user running GSD through OpenCode might be using any of these models at any time. GSD workflows that depend on Claude-specific instruction-following behavior will work when the OpenCode user has Claude selected, but fail or degrade silently when they switch to GPT-4o or Gemini. From GSD's perspective, it's running "on OpenCode" — but the actual compliance behavior varies based on which model OpenCode is proxying. This is a two-level capability problem: OpenCode support is necessary but not sufficient for reliable GSD behavior.

**Why it happens:**
The natural framing is "GSD now supports OpenCode." But OpenCode is a runtime, not a model. The model is what actually executes instructions. Treating "OpenCode support" as complete without specifying model requirements buries a model-compliance dependency inside a runtime-compatibility story.

**How to avoid:**
GSD's OpenCode support must include a model requirement specification: "GSD on OpenCode requires Claude (claude-3-7-sonnet or later) as the active model for full functionality. Other models provide degraded-mode operation." This must be surfaced at session start when GSD detects OpenCode and Claude is not the active model. The workflow architecture for OpenCode must be tested against each supported model, not just Claude.

**Warning signs:**
- OpenCode integration tested only with Claude as the active model
- No model-checking logic in GSD's OpenCode initialization
- Documentation that says "OpenCode supported" without specifying required model
- Assuming OpenCode users always use Claude because "that's what we use"

**Phase to address:**
Phase 4 (OpenCode Integration) — Model requirement detection must be part of the integration, not a later addition.

---

### Pitfall 8: Prompt Pattern Translation Is Not Mechanical

**What goes wrong:**
Developers attempt to port GSD workflow files by systematically replacing Claude-specific patterns with model-agnostic equivalents — e.g., replacing Claude's instruction-following XML tags with JSON schemas or numbered lists. This feels like a mechanical transformation. It is not. Prompt patterns fail in ways that are model-specific, context-dependent, and non-obvious. A pattern that works for Claude in a 10-step workflow may fail for GPT-4o in step 7 specifically, because GPT-4o handles accumulated context differently than Claude. The failure is not visible until a full end-to-end run.

**Why it happens:**
Pattern translation is treated as a translation problem (mechanical substitution) rather than an empirical problem (test every combination). The assumption is that if individual patterns work in isolation, they will work in composition. This assumption is wrong for LLMs.

**How to avoid:**
Test prompt patterns in composition, not isolation. For each workflow ported to Codex CLI or OpenCode: run the entire workflow end-to-end, not just individual steps. Collect output at each step and verify compliance. Build a test corpus of known-good outputs from Claude executions and diff against Codex/OpenCode outputs. Flag any step where output structure diverges, even if the final result appears correct — divergence in intermediate steps predicts future failures.

**Warning signs:**
- Prompt translation tested via spot-checking individual prompts
- No end-to-end workflow test suite
- "It worked on my test case" without a systematic test corpus
- No diff of intermediate outputs between Claude and target runtime

**Phase to address:**
Phase 2 (Workflow Translation) and Phase 5 (Testing) — Translation and testing must be interleaved, not sequential.

---

### Pitfall 9: State Management Assumptions Break Across Runtime Restart Models

**What goes wrong:**
GSD's state model (STATE.md, per-plan SUMMARY.md files, git commits as checkpoints) was designed for Claude Code's session model where context persists within a conversation. The resumption logic (`Re-run /gsd:execute-phase {phase}` → discover completed SUMMARYs → skip them → resume) works because GSD can call `git log` and read files to reconstruct position. Codex CLI has a different session lifecycle — by default, each invocation starts fresh without access to conversation history. OpenCode may retain context within a session but not across sessions. If GSD's resumption logic assumes the agent has conversational context about what happened before (not just file artifacts), resumption will fail when that context is not available.

**Why it happens:**
GSD's resumption was designed to survive Claude Code context compaction (the system that prunes old conversation turns). It relies on structured file artifacts as the source of truth. This should be portable — but developers discover edge cases where agents refer to conversational context ("as I mentioned earlier") that doesn't exist in a fresh Codex CLI invocation, causing confusion in the execution flow.

**How to avoid:**
Audit every GSD workflow step that refers to prior context without reading it from a file. Any reference to "what was discussed" or "what was decided" must be grounded in a file read, not assumed to be in context. STATE.md and SUMMARY.md files must be the complete, sufficient resumption context — no conversational breadcrumbs allowed. Test resumption explicitly by starting fresh Codex CLI and OpenCode sessions mid-phase and verifying that execution continues correctly from file state alone.

**Warning signs:**
- Workflow steps that reference "earlier in this session" or "as established above"
- STATE.md files that record decisions as summaries rather than decisions
- Resumption logic that was never tested with a cold-start (no conversational history)
- Checkpoint continuation that relies on the orchestrator "remembering" what the checkpoint said

**Phase to address:**
Phase 3 (State Architecture) — Must validate file-only resumption before any runtime porting is considered complete.

---

### Pitfall 10: Testing Multi-Runtime Compatibility Is Expensive and Skipped

**What goes wrong:**
Testing GSD on three runtimes (Claude Code, Codex CLI, OpenCode) is approximately 3x the manual testing effort of testing on one. The temptation is to test only on Claude Code (it's the primary runtime) and assert "other runtimes should work similarly." This produces a system with known runtime support in documentation and unknown runtime support in practice. The failure is discovered by users, not by developers.

**Why it happens:**
Multi-runtime testing requires: accounts/installs for each runtime, test scenarios designed to exercise runtime-specific behavior, time for non-deterministic LLM outputs to stabilize, and a way to isolate runtime differences from random model variation. All of these create friction. The path of least resistance is to skip it.

**How to avoid:**
Build a minimal but real integration test suite before shipping runtime support. The test suite must: (1) be runnable in under 30 minutes per runtime, (2) cover the happy path for the top 3 workflows (`/gsd:new-project`, `/gsd:plan-phase`, `/gsd:execute-phase`), (3) have pass/fail criteria that don't require subjective evaluation of LLM output quality. Accept that 100% automated testing of LLM behavior is not achievable — but 80% automated + 20% structured manual is achievable and sufficient.

**Warning signs:**
- No integration tests exist for Codex CLI or OpenCode support
- Codex CLI support marked "complete" based on a single manual test run
- Test suite only tests GSD's tool infrastructure (gsd-tools.cjs) not the workflow execution
- No definition of what "works on Codex CLI" means in measurable terms

**Phase to address:**
Phase 5 (Testing Infrastructure) — Must be built before any runtime is declared production-ready.

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Copy-paste Claude workflow files to Codex CLI without testing | Fast "port" with minimal effort | Silent failures in critical steps, no compliance guarantee | Never — the workflow files are Claude dialects, not generic markdown |
| Single `RUNTIME` env var for detection with no fallback | Simple to implement | Misdetects in many shell environments, no recovery path | Never in production |
| "Parallel execution not supported" with no user warning | No warning code to write | Users see 3x slowdown with no explanation | Never — always warn |
| Hardcoding `/Users/thelorax/` paths in Codex CLI workflows | Works for the author | Fails for every other user and in Codex sandbox | Never — all paths must be runtime-resolved |
| Testing OpenCode support only with Claude as active model | Test passes | Real users with GPT-4o see degraded behavior with no warning | Only acceptable during initial development; must be addressed before release |
| One capability registry maintained in the orchestrator | Simple, one place to update | Capability logic scattered across workflows when they need to branch on runtime | Acceptable for first version if designed for extraction later |
| Treating "OpenCode support" and "model-agnostic support" as the same thing | Simpler mental model | OpenCode with GPT-4o fails while "OpenCode is supported" | Never — must distinguish runtime from model |

---

## Integration Gotchas

Common mistakes when connecting to external services.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Codex CLI | Passing `@file-reference` syntax expecting automatic file reads | Codex CLI does not process `@file` prefixes; must use explicit read-file tool calls or inline content |
| Codex CLI | Expecting `Task()` subagent spawning to work | Codex CLI has no subagent spawning API; sequential execution only, or use Codex CLI's own orchestration if available |
| Codex CLI sandbox | Calling `gsd-tools.cjs` at `/Users/thelorax/` path | Sandbox may block access; use project-relative paths or require `--full-auto` mode explicitly |
| OpenCode | Assuming Claude is the active model | OpenCode proxies multiple models; detect active model and warn if not Claude |
| OpenCode | Passing structured YAML frontmatter expecting model to parse it | Model compliance with frontmatter parsing varies; validate output format explicitly |
| GPT-4o (via Codex CLI) | Multi-level XML instruction nesting | GPT-4o handles single-level XML reasonably but loses compliance with deeply nested conditional XML structures |
| GPT-4o (via Codex CLI) | Expecting `## PLAN COMPLETE` structured return formats | GPT-4o generates plausible-looking but structurally non-compliant returns; must use output format enforcement |

---

## Performance Traps

Patterns that work at small scale but fail as usage grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Sequential fallback for wave execution without warning | Execution takes 3x longer, user doesn't know why | Explicit warning at wave start: "Parallel not available on this runtime" | First multi-plan phase on Codex CLI |
| Running all workflows on every runtime without caching capability profiles | Redundant capability detection on every invocation | Cache detected capability profile in `.planning/config.json` per session | After 10+ workflow invocations |
| No model-specific context budget tuning | Context overflow on smaller model windows | Tune plan context budgets per runtime's model context window | First time a non-Claude model is used with standard-depth planning |
| Porting prompt verbosity without trimming for smaller models | Token cost 2-3x higher for same task on smaller models | Audit prompt length per model and trim unnecessary instructions | When OpenCode users on GPT-4o see unexpectedly high costs |

---

## Security Mistakes

Domain-specific security issues beyond general web security.

| Mistake | Risk | Prevention |
|---------|------|------------|
| Runtime detection based on user-controlled env var without validation | User spoofs runtime identity to bypass capability checks | Validate runtime detection with multiple signals; don't trust single env var |
| OpenCode model detection trusting user-reported model name | User claims Claude is active to get full-capability mode, then runs restricted model | Detect model from actual API response headers or provider-specific signals, not user input |
| Codex CLI `--full-auto` mode required without documenting its scope | User enables full-auto without understanding it approves all shell commands | Require `--full-auto` with explicit user acknowledgment; document exactly what it approves |

---

## UX Pitfalls

Common user experience mistakes in this domain.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| No runtime indicator in GSD session header | User doesn't know which runtime or capability profile is active | Show runtime + capability summary at session start: "GSD on Codex CLI — parallel execution: no, subagents: no" |
| Degraded features look identical to full features | User can't tell if they're getting full GSD or fallback GSD | Visual distinction between full-capability and degraded-mode operations |
| Runtime-specific setup instructions buried in docs | User encounters Codex CLI-specific errors without guidance | Show runtime-specific setup instructions at first run per runtime |
| "Supported" listed for OpenCode without model requirement | User assumes any model works; switches to Gemini; GSD fails | State supported models explicitly in runtime support documentation |
| Parallel wave output silently becomes sequential output | User waiting for parallel result, gets sequential result 3x later | Announce sequential fallback before starting: "Running 3 plans sequentially (parallel not available)" |

---

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **Codex CLI support:** Often missing sandbox path resolution — verify `gsd-tools.cjs` is accessible in Codex CLI's execution environment
- [ ] **OpenCode support:** Often missing model requirement check — verify behavior when GPT-4o or Gemini is active, not just Claude
- [ ] **Runtime detection:** Often missing multi-signal fallback — verify detection works in VS Code terminal, iTerm, and non-interactive shells
- [ ] **Parallel execution fallback:** Often missing user notification — verify sequential fallback emits a visible warning before starting
- [ ] **Workflow porting:** Often tested only in isolation — verify each ported workflow runs end-to-end without step-skipping
- [ ] **State resumption:** Often tested only with conversational context present — verify resumption works from cold start with file state only
- [ ] **Capability registry:** Often incomplete for edge cases — verify every GSD capability has an explicit entry per runtime (not "unknown = assume works")
- [ ] **Completion signal handling:** Often copied from Claude Code — verify failure detection logic is validated against actual Codex CLI and OpenCode failure outputs

---

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Task spawning fails silently on Codex CLI | MEDIUM | Add explicit Task() call detection in abstraction layer, surface error, redirect to sequential path |
| gsd-tools.cjs inaccessible in Codex sandbox | HIGH | Audit and relocate all gsd-tools dependencies to project-relative paths; short-term: require --full-auto |
| Workflow file produces wrong output on GPT-4o | HIGH | Write GPT-4o-specific workflow variant from scratch; don't iterate on the Claude version |
| Runtime misdetected, wrong capability profile used | LOW | Add manual override to config.json; re-run with explicit runtime flag |
| OpenCode user on wrong model gets silent degradation | MEDIUM | Add model check to OpenCode initialization; surface model requirement on next session start |
| Resumption fails after cold start | MEDIUM | Audit workflow for conversational context dependencies; add file-read step to load all needed context at workflow start |
| Sequential fallback used without user awareness | LOW | Add warning emission to fallback path; no refactoring needed |

---

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Task tool has no universal equivalent | Phase 1: Runtime Abstraction | Every Task() call is wrapped; wrapper has runtime-specific behavior for all 3 runtimes |
| Silent feature degradation | Phase 1: Runtime Abstraction | Each degraded path emits a visible warning in terminal output |
| Runtime detection returning wrong answers | Phase 1: Runtime Abstraction | Detection tested in 3+ shell environments; manual override works |
| Codex CLI sandboxing breaks gsd-tools paths | Phase 1 + Phase 3: Codex Integration | gsd-tools.cjs called with no absolute user-home paths in Codex workflows |
| Markdown compliance varies across models | Phase 2: Workflow Translation | End-to-end workflow test on Codex CLI produces compliant structured output |
| classifyHandoffIfNeeded pattern is Claude-specific | Phase 2: Workflow Translation | Completion signal handling is runtime-specific, not copied from Claude handler |
| OpenCode model-agnostic design | Phase 4: OpenCode Integration | Model check at session start; non-Claude models show capability warning |
| Prompt pattern translation is not mechanical | Phase 2: Workflow Translation | Full end-to-end test suite covers all ported workflows |
| State management assumptions break | Phase 3: State Architecture | Resumption tested from cold start; no conversational context references in workflows |
| Multi-runtime testing is skipped | Phase 5: Testing Infrastructure | Automated integration tests run on Claude Code, Codex CLI, OpenCode before release |

---

## Sources

- GSD codebase analysis: `/Users/thelorax/.claude/get-shit-done/workflows/execute-phase.md` — Task() spawning model, wave execution, classifyHandoffIfNeeded handling
- GSD codebase analysis: `/Users/thelorax/.claude/agents/gsd-executor.md` — completion signaling, state update model
- GSD codebase analysis: `/Users/thelorax/.claude/get-shit-done/workflows/new-project.md` — research spawning, agent invocation patterns
- GSD codebase analysis: `/Users/thelorax/.claude/agents/gsd-planner.md` — subagent spawning patterns, Claude-specific instruction compliance assumptions
- Training data (MEDIUM confidence): Codex CLI architecture — OpenAI open-source terminal agent, GPT-4o/o3 models, Responses API, sandboxed execution, no Task subagent API
- Training data (MEDIUM confidence): OpenCode architecture — sst.dev terminal tool, multi-model support (Anthropic/OpenAI/Gemini), slash-command extensibility, no Task subagent API
- Training data (HIGH confidence): GPT-4o instruction following — handles XML tags but with lower compliance than Claude on deeply nested conditional structures, structured return formats require explicit enforcement

---
*Pitfalls research for: Multi-Runtime AI Agent Tool Support (Codex CLI + OpenCode on GSD)*
*Researched: 2026-02-16*
