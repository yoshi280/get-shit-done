# GSD Framework Improvements

## What This Is

A fork-based development effort to make get-shit-done's meta-prompting smarter and more flexible. The fixed research/execution pipeline works for simple projects but breaks down for complex or unusual ones — rigid dimensions, no phase memory, no iteration. This project addresses those gaps while staying compatible with upstream patterns.

## Core Value

GSD agents should accumulate and apply context intelligently — the right research for the right project, and knowledge that persists and compounds across phases instead of being lost at each context boundary.

## Requirements

### Validated

- Multi-agent orchestrator system (commands → workflows → agents → tools) — existing
- 4-dimension research pipeline (stack/features/architecture/pitfalls) — existing
- Linear phase execution (plan → execute → verify) — existing
- State management via STATE.md + config.json — existing
- Git-integrated atomic commits per task — existing
- Multi-runtime support (Claude Code, OpenCode, Gemini) — existing
- Model profile routing (quality/balanced/budget) — existing
- Zero external dependencies (Node.js built-ins only) — existing
- Checkpoint system for user decisions during execution — existing
- Deviation tracking in plan summaries — existing
- Local patch persistence via SHA256 manifest — existing

### Active

- [ ] Flexible research dimensions — Claude infers relevant dimensions per project, user can add/remove
- [ ] New research dimension types beyond the current fixed 4
- [ ] Structural changes to how dimensions are defined and dispatched
- [ ] Phase iteration — re-execute phases with modified plans, keeping what worked
- [ ] Revision cycles — phases produce output, get feedback, revise like drafts
- [ ] Phase rollback — undo a phase and try a different approach
- [ ] Cross-phase context — later phases know what earlier phases built
- [ ] Cross-phase learning — patterns and mistakes from prior phases inform future ones
- [ ] Custom workflows — user-defined workflow steps beyond the fixed GSD pipeline
- [ ] Idea capture — todos for actionable items + backlog file for bigger ideas
- [ ] Codex CLI runtime support — GSD workflows function correctly inside Codex CLI
- [ ] OpenCode runtime parity — real workflow parity beyond install-level support
- [ ] Runtime detection — GSD adapts behavior based on which CLI it's running in
- [ ] Cross-runtime agent orchestration — subagent spawning patterns for non-Claude runtimes

### Out of Scope

- Multi-provider model routing (switching GSD to call OpenAI/Gemini APIs) — separate effort on `fork/multi-provider-routing` branch
- Changes to upstream installer/distribution — focus on workflow and agent improvements
- Mobile or web interfaces — CLI-only tool
- Gemini CLI runtime parity — deferred to v1.2+

## Context

- **Fork:** `yoshi280/get-shit-done`, synced with upstream through 1.20.3
- **PR #610:** Open for initial selectable research dimensions, awaiting review from glittercowboy
- **Development flow:** Prototype in local install (`~/.claude/get-shit-done/`), formalize in fork (`~/get-shit-done/`), submit PRs upstream when ready
- **Living project:** No fixed end state. Milestone per feature set, ongoing improvement.
- **Codebase map:** `.planning/codebase/` has 7 documents from current analysis

## Constraints

- **Upstream compatibility**: Changes must follow existing GSD patterns (markdown agents, gsd-tools.cjs CLI, state management via tools) so PRs are viable
- **Zero dependencies**: No new npm packages — Node.js built-ins only, matching upstream philosophy
- **Multi-runtime**: Agent/workflow changes must work across Claude Code, OpenCode, and Gemini CLI
- **Fork fragility**: Local install is upstream + manually copied fork files. Any `/gsd:update` overwrites fork additions until PRs merge or install switches to fork

## Current Milestone: v1.0 GSD Framework Improvements

**Goal:** Transform GSD's fixed pipeline into a flexible, context-aware system with selectable research dimensions, observability, idea capture, cross-phase memory, phase iteration, and custom workflow controls.

**Target features:**
- Selectable research dimensions (user-editable, parameterized researchers)
- Observability (token/cost tracking per dimension and phase)
- Idea capture system (IDEAS.md backlog)
- Cross-phase memory and context awareness
- Phase state machine with iteration/rollback
- Custom workflow controls

## Upcoming Milestone: v1.1 Codex + OpenCode Support

**Goal:** GSD works on Codex CLI and OpenCode with the same quality it has on Claude Code — full parity in agent orchestration, questioning flows, subagent spawning, and token efficiency.

**Target features:**
- Capability audit: document what each runtime actually supports vs. Claude Code
- OpenCode: real workflow parity (currently install-only)
- Codex CLI: net-new support (no GSD support today)
- Runtime detection and adaptive behavior
- Agent orchestration / subagent patterns for non-Claude runtimes
- Token efficiency for models with different behavior characteristics

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Prototype locally, port to fork | Faster iteration without push/pull cycle | — Pending |
| Claude infers dimensions + user edits | Pure user-pick is tedious, pure auto misses domain knowledge | — Pending |
| Todos + backlog file for idea capture | Actionable items need different tracking than big ideas | — Pending |
| Multi-provider routing out of scope | Already on separate branch, different concern | — Pending |

---
*Last updated: 2026-02-16 after v1.1 milestone definition*
