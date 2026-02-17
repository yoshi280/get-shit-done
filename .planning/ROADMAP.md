# Roadmap: GSD Framework Improvements

## Overview

Transform GSD's fixed research/execution pipeline into a flexible, context-aware system that learns across phases. Starting with selectable research dimensions to prove the core value proposition, we layer on observability, persistent memory, iteration capabilities, and finally custom workflow controls. Each phase builds on proven patterns while preserving GSD's zero-dependency philosophy and upstream compatibility.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Selectable Research Dimensions + Observability** - Flexible, user-editable dimensions with usage tracking (completed 2026-02-17)
- [ ] **Phase 2: Idea Capture System** - Backlog file for bigger ideas beyond actionable todos
- [ ] **Phase 3: Cross-Phase Memory and Context Awareness** - Persistent state and learning across phase boundaries
- [ ] **Phase 4: Phase State Machine with Iteration** - Rollback capability and plan re-execution
- [ ] **Phase 5: Custom Workflow Controls** - User-defined phase templates and reordering

## Phase Details

### Phase 1: Selectable Research Dimensions + Observability
**Goal**: Users can customize which research dimensions run for their project, and see token/cost impact of research choices
**Depends on**: Nothing (first phase)
**Requirements**: DIM-01, DIM-02, DIM-03, DIM-04, OBS-01, OBS-02, OBS-03
**Success Criteria** (what must be TRUE):
  1. User can add custom research dimensions before researchers spawn
  2. User can remove default dimensions that aren't relevant to their project
  3. User can edit dimension prompts to tailor research focus
  4. New dimension templates exist beyond current 4 (best-practices, data-structures available)
  5. Token usage is tracked and displayed per research dimension
  6. Token usage is tracked and displayed per phase execution
  7. Cost summary appears in progress reports
**Plans**: 3 plans

Plans:
- [ ] 01-01-PLAN.md -- Dimension catalog system (9 dimension files: 4 migrated + 5 new)
- [ ] 01-02-PLAN.md -- Dimension selection flow in plan-phase.md + researcher parameterization
- [ ] 01-03-PLAN.md -- Observability backend (cost tracking tools, STATE.md tracker, progress display)

### Phase 2: Idea Capture System
**Goal**: Users have a structured place to capture emerging ideas during execution without cluttering actionable todos
**Depends on**: Phase 1
**Requirements**: IDEA-01, IDEA-02, IDEA-03
**Success Criteria** (what must be TRUE):
  1. IDEAS.md backlog file exists for bigger ideas beyond actionable todos
  2. Ideas surfaced during phase execution are auto-logged to IDEAS.md
  3. /gsd:add-todo can route to IDEAS.md vs todos based on item type
**Plans**: TBD

Plans:
- [ ] 02-01: TBD
- [ ] 02-02: TBD

### Phase 3: Cross-Phase Memory and Context Awareness
**Goal**: Later phases receive and apply insights from earlier phases instead of starting fresh each time
**Depends on**: Phase 1 (needs observability for tracking context)
**Requirements**: CTX-01, CTX-02, CTX-03, CTX-04
**Success Criteria** (what must be TRUE):
  1. Phase planning agents receive summaries from prior completed phases
  2. Phase execution agents can reference prior phase artifacts via @-references
  3. Progressive summarization compresses old phase context to avoid token bloat
  4. STATE.md tracks a running context digest updated after each phase
**Plans**: TBD

Plans:
- [ ] 03-01: TBD
- [ ] 03-02: TBD
- [ ] 03-03: TBD

### Phase 4: Phase State Machine with Iteration
**Goal**: Users can retry phases with different approaches without losing all prior work
**Depends on**: Phase 3 (requires state persistence infrastructure)
**Requirements**: ITER-01, ITER-02, ITER-03, ITER-04
**Success Criteria** (what must be TRUE):
  1. User can re-execute a single phase with a modified plan
  2. Re-execution preserves work from tasks that don't need changes
  3. Git checkpoint created before each phase state write
  4. User can restore to a previous checkpoint to retry a phase
**Plans**: TBD

Plans:
- [ ] 04-01: TBD
- [ ] 04-02: TBD
- [ ] 04-03: TBD

### Phase 5: Custom Workflow Controls
**Goal**: Users can define, select, and modify phase structures for their domain-specific needs
**Depends on**: Phase 4 (can't reorder/skip without iteration mechanics)
**Requirements**: WF-01, WF-02, WF-03, WF-04
**Success Criteria** (what must be TRUE):
  1. Workflow templates define pre-built phase structures for common project types
  2. User can select a workflow template during project initialization
  3. User can reorder phases in an active roadmap
  4. User can skip a phase without breaking downstream dependencies
**Plans**: TBD

Plans:
- [ ] 05-01: TBD
- [ ] 05-02: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Selectable Research Dimensions + Observability | 0/3 | Complete    | 2026-02-17 |
| 2. Idea Capture System | 0/2 | Not started | - |
| 3. Cross-Phase Memory and Context Awareness | 0/3 | Not started | - |
| 4. Phase State Machine with Iteration | 0/3 | Not started | - |
| 5. Custom Workflow Controls | 0/2 | Not started | - |
