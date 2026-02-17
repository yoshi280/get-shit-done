# Phase 1: Selectable Research Dimensions + Observability - Context

**Gathered:** 2026-02-16
**Status:** Ready for planning

<domain>
## Phase Boundary

Users can customize which research dimensions run for their project, and see token/cost impact of research choices. Dimension selection happens per-phase during `/gsd:plan-phase`. The fixed 4-dimension pipeline is replaced with a flexible, user-editable set drawn from a curated catalog plus user-saved templates.

</domain>

<decisions>
## Implementation Decisions

### Dimension selection flow
- Selection happens during `/gsd:plan-phase`, not during `/gsd:new-project`
- Claude infers relevant dimensions based on project type AND phase goal, presents as multi-select checklist via AskUserQuestion
- User toggles dimensions on/off, can add custom ones via "Other"
- After selection, always offer to customize individual dimension prompts
- Custom dimensions require name + brief description; Claude expands into full research prompt

### Token & cost reporting
- Token usage displayed in three places: live output after each researcher finishes, per-dimension detail in SUMMARY.md, per-phase aggregates in /gsd:progress
- Both per-dimension and per-phase granularity: detailed breakdown in summaries, aggregates in progress reports
- Report tokens AND estimated dollar cost, based on configurable price-per-token in config.json with current model defaults
- Cumulative project-wide cost tracker maintained in STATE.md, updated after each phase

### New dimension catalog
- Expand beyond current 4 (stack, features, architecture, pitfalls) with: best-practices, data-structures, security & compliance, testing strategies, DevOps & deployment
- Ship curated defaults AND allow user-saved custom templates
- Claude infers dimensions by analyzing both project type (web app, CLI, library) and specific phase goal
- No hard limit on dimension count; cost reporting provides the natural feedback loop

### Dimension prompt structure
- Rich dimension definitions: name + short description (for checklist display) + research prompt + tags/categories + suggested project types
- Storage: global defaults (~/.claude/get-shit-done/dimensions/) + project overrides (.planning/dimensions/); project-level takes precedence on name collision
- Storage format and prompt editing UX are Claude's discretion

### Claude's Discretion
- Dimension definition storage format (standalone files vs single catalog)
- Dimension prompt editing experience (free text override vs guided append)
- Exact implementation of token tracking mechanism

</decisions>

<specifics>
## Specific Ideas

- The multi-select checklist should feel native to the existing AskUserQuestion pattern already used in GSD workflows
- "Claude infers + user edits" was an early project decision (see PROJECT.md Key Decisions) — this phase implements that vision
- Cost reporting should make the user feel informed, not anxious — present data without judgment

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-selectable-research-dimensions-observability*
*Context gathered: 2026-02-16*
