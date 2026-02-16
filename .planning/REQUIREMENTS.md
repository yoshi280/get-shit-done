# Requirements: GSD Framework Improvements

**Defined:** 2026-02-16
**Core Value:** GSD agents should accumulate and apply context intelligently — the right research for the right project, and knowledge that persists and compounds across phases.

## v1 Requirements

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

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| DIM-01 | — | Pending |
| DIM-02 | — | Pending |
| DIM-03 | — | Pending |
| DIM-04 | — | Pending |
| ITER-01 | — | Pending |
| ITER-02 | — | Pending |
| ITER-03 | — | Pending |
| ITER-04 | — | Pending |
| CTX-01 | — | Pending |
| CTX-02 | — | Pending |
| CTX-03 | — | Pending |
| CTX-04 | — | Pending |
| WF-01 | — | Pending |
| WF-02 | — | Pending |
| WF-03 | — | Pending |
| WF-04 | — | Pending |
| IDEA-01 | — | Pending |
| IDEA-02 | — | Pending |
| IDEA-03 | — | Pending |
| OBS-01 | — | Pending |
| OBS-02 | — | Pending |
| OBS-03 | — | Pending |

**Coverage:**
- v1 requirements: 22 total
- Mapped to phases: 0
- Unmapped: 22

---
*Requirements defined: 2026-02-16*
*Last updated: 2026-02-16 after initial definition*
