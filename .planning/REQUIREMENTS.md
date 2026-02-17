# Requirements: GSD Framework Improvements

**Defined:** 2026-02-16
**Core Value:** GSD agents should accumulate and apply context intelligently — the right research for the right project, and knowledge that persists and compounds across phases.

## v1.0 Requirements

### Research Dimensions

- [ ] **DIM-01**: User can add custom research dimensions before researchers spawn
- [ ] **DIM-02**: User can remove default dimensions that aren't relevant to their project
- [ ] **DIM-03**: User can edit dimension prompts/questions to tailor research focus
- [ ] **DIM-04**: New dimension templates exist beyond current 4 (best-practices, data-structures already in fork)

### Phase Iteration

- [ ] **ITER-01**: User can re-execute a single phase with a modified plan
- [ ] **ITER-02**: Re-execution preserves work from tasks that don't need changes
- [ ] **ITER-03**: Git checkpoint created before each phase state write
- [ ] **ITER-04**: User can restore to a previous checkpoint to retry a phase

### Cross-Phase Awareness

- [ ] **CTX-01**: Phase planning agents receive summaries from prior completed phases
- [ ] **CTX-02**: Phase execution agents can reference prior phase artifacts via @-references
- [ ] **CTX-03**: Progressive summarization compresses old phase context to avoid token bloat
- [ ] **CTX-04**: STATE.md tracks a running context digest updated after each phase

### Custom Workflows

- [ ] **WF-01**: Workflow templates define pre-built phase structures for common project types
- [ ] **WF-02**: User can select a workflow template during project initialization
- [ ] **WF-03**: User can reorder phases in an active roadmap
- [ ] **WF-04**: User can skip a phase without breaking downstream dependencies

### Idea Capture

- [ ] **IDEA-01**: IDEAS.md backlog file captures bigger ideas beyond actionable todos
- [ ] **IDEA-02**: Ideas surfaced during phase execution are auto-logged to IDEAS.md
- [ ] **IDEA-03**: /gsd:add-todo can route to IDEAS.md vs todos based on item type

### Observability

- [ ] **OBS-01**: Token usage tracked per research dimension spawn
- [ ] **OBS-02**: Token usage tracked per phase execution
- [ ] **OBS-03**: Cost summary displayed in progress reports

## v1.1 Requirements

### Runtime Detection

- [ ] **RTE-01**: GSD detects the active runtime via multi-signal heuristics (env vars + filesystem) and exposes a `runtime` field in all gsd-tools.cjs init JSON payloads
- [ ] **RTE-02**: Parallelization is forced to `false` in init JSON when runtime is not Claude Code, regardless of config.json settings

### OpenCode Agent Registration

- [ ] **OCA-01**: GSD installer injects named agent system prompts from `gsd-*.md` files into `opencode.json` as `AgentConfig` entries so GSD's named agent types resolve on OpenCode
- [ ] **OCA-02**: gsd-tools.cjs resolves GSD tier names (`opus`, `sonnet`, `haiku`) to `provider/model` strings that OpenCode's `AgentConfig.model` field accepts

### OpenCode Workflow Parity

- [ ] **OWP-01**: The 5 key GSD workflows (`new-project`, `plan-phase`, `execute-phase`, `new-milestone`, `research-phase`) contain runtime-conditional spawn blocks that execute inline when the Task tool is unavailable
- [ ] **OWP-02**: Every fallback execution path displays an explicit degradation notice: what capability is absent, what the fallback does instead, and expected performance impact

### Validation

- [ ] **VAL-01**: Integration tests verify happy-path artifact output for `/gsd:new-project`, `/gsd:plan-phase`, and `/gsd:execute-phase` on OpenCode
- [ ] **VAL-02**: A runtime capability matrix is produced and committed, replacing LOW/MEDIUM-confidence entries from the v1.1 research

## v2 Requirements

### Research Dimensions

- **DIM-05**: Claude infers relevant dimensions automatically based on project type and context
- **DIM-06**: Dimension registry as data structure — new types pluggable without code changes
- **DIM-07**: Domain-specific dimension presets (e.g., "web app" preset, "CLI tool" preset)

### Phase Iteration

- **ITER-05**: Multi-phase rollback with compensating actions across dependent phases
- **ITER-06**: Revision cycles — phase produces output, gets feedback, revises like drafts
- **ITER-07**: Selective rollback — undo phase N without losing independent phase N+1

### Cross-Phase Awareness

- **CTX-05**: Pattern and mistake learning — agents learn from prior phase outcomes
- **CTX-06**: Dual-tier memory architecture (private per-agent + shared cross-phase)

### Custom Workflows

- **WF-05**: Workflow DSL in markdown for fully user-defined workflow steps
- **WF-06**: Phase injection — insert urgent work mid-roadmap without disrupting flow

### Runtime Support

- **RTE-03**: User can override runtime detection via `config.json` manual `runtime` field
- **RTE-04**: GSD warns at session start when OpenCode is backed by a non-Claude model
- **RTE-05**: Codex CLI runtime support — full GSD support for OpenAI Codex CLI (deferred pending capability audit)

## Out of Scope

| Feature | Reason |
|---------|--------|
| Multi-provider model routing (OpenAI, Gemini) | Separate effort on `fork/multi-provider-routing` branch |
| Swarm coordination patterns | Defer until hierarchical patterns proven inadequate |
| Multimodal memory | Research-level problem, not needed for markdown workflows |
| Temporal/enterprise orchestration | Massive scope increase, no enterprise demand yet |
| Real-time agent coordination | Unnecessary complexity; checkpoint-based is sufficient |
| Agents for deterministic tasks | Rules-based systems are simpler and more reliable |
| Graph-based workflow visualization | Nice-to-have UI, not core to workflow improvement |
| Codex CLI support in v1.1 | Requires capability audit; training data confidence is LOW (released Apr 2025, post-cutoff); deferred to v1.2 |
| config.json runtime override (v1.1) | Deferred to v2 — auto-detection sufficient for initial release |
| Gemini CLI workflow parity | Explicitly deferred to v1.2+ per PROJECT.md |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| DIM-01 | Phase 1 | Pending |
| DIM-02 | Phase 1 | Pending |
| DIM-03 | Phase 1 | Pending |
| DIM-04 | Phase 1 | Pending |
| OBS-01 | Phase 1 | Pending |
| OBS-02 | Phase 1 | Pending |
| OBS-03 | Phase 1 | Pending |
| IDEA-01 | Phase 2 | Pending |
| IDEA-02 | Phase 2 | Pending |
| IDEA-03 | Phase 2 | Pending |
| CTX-01 | Phase 3 | Pending |
| CTX-02 | Phase 3 | Pending |
| CTX-03 | Phase 3 | Pending |
| CTX-04 | Phase 3 | Pending |
| ITER-01 | Phase 4 | Pending |
| ITER-02 | Phase 4 | Pending |
| ITER-03 | Phase 4 | Pending |
| ITER-04 | Phase 4 | Pending |
| WF-01 | Phase 5 | Pending |
| WF-02 | Phase 5 | Pending |
| WF-03 | Phase 5 | Pending |
| WF-04 | Phase 5 | Pending |

| RTE-01 | Phase TBD | Pending |
| RTE-02 | Phase TBD | Pending |
| OCA-01 | Phase TBD | Pending |
| OCA-02 | Phase TBD | Pending |
| OWP-01 | Phase TBD | Pending |
| OWP-02 | Phase TBD | Pending |
| VAL-01 | Phase TBD | Pending |
| VAL-02 | Phase TBD | Pending |

**Coverage:**
- v1.0 requirements: 22 total — mapped to phases 1-5 ✓
- v1.1 requirements: 8 total — phases TBD (roadmap creation pending)
- Unmapped: 8 ⚠️ (v1.1 — will be resolved by roadmapper)

---
*Requirements defined: 2026-02-16*
*Last updated: 2026-02-16 after v1.1 milestone definition*
