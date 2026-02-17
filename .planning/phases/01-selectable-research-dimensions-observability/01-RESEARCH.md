# Phase 1: Selectable Research Dimensions + Observability - Research

**Researched:** 2026-02-16
**Domain:** GSD framework internals — markdown workflow extension, dimension catalog, token observability
**Confidence:** HIGH

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

#### Dimension selection flow
- Selection happens during `/gsd:plan-phase`, not during `/gsd:new-project`
- Claude infers relevant dimensions based on project type AND phase goal, presents as multi-select checklist via AskUserQuestion
- User toggles dimensions on/off, can add custom ones via "Other"
- After selection, always offer to customize individual dimension prompts
- Custom dimensions require name + brief description; Claude expands into full research prompt

#### Token & cost reporting
- Token usage displayed in three places: live output after each researcher finishes, per-dimension detail in SUMMARY.md, per-phase aggregates in /gsd:progress
- Both per-dimension and per-phase granularity: detailed breakdown in summaries, aggregates in progress reports
- Report tokens AND estimated dollar cost, based on configurable price-per-token in config.json with current model defaults
- Cumulative project-wide cost tracker maintained in STATE.md, updated after each phase

#### New dimension catalog
- Expand beyond current 4 (stack, features, architecture, pitfalls) with: best-practices, data-structures, security & compliance, testing strategies, DevOps & deployment
- Ship curated defaults AND allow user-saved custom templates
- Claude infers dimensions by analyzing both project type (web app, CLI, library) and specific phase goal
- No hard limit on dimension count; cost reporting provides the natural feedback loop

#### Dimension prompt structure
- Rich dimension definitions: name + short description (for checklist display) + research prompt + tags/categories + suggested project types
- Storage: global defaults (`~/.claude/get-shit-done/dimensions/`) + project overrides (`.planning/dimensions/`); project-level takes precedence on name collision
- Storage format and prompt editing UX are Claude's discretion

### Claude's Discretion
- Dimension definition storage format (standalone files vs single catalog)
- Dimension prompt editing experience (free text override vs guided append)
- Exact implementation of token tracking mechanism

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| DIM-01 | User can add custom research dimensions before researchers spawn | Dimension catalog architecture pattern; `AskUserQuestion` checklist flow; storage in `.planning/dimensions/` |
| DIM-02 | User can remove default dimensions that aren't relevant to their project | Same checklist flow — unchecking removes dimension from the spawn list |
| DIM-03 | User can edit dimension prompts/questions to tailor research focus | Post-selection prompt-editing pattern; standalone file format enables direct Write tool edits |
| DIM-04 | New dimension templates exist beyond current 4 (best-practices, data-structures already in fork) | 5 new dimension files shipped in `~/.claude/get-shit-done/dimensions/` |
| OBS-01 | Token usage tracked per research dimension spawn | Self-reporting pattern: each researcher subagent writes its own token estimate into its RESEARCH.md header |
| OBS-02 | Token usage tracked per phase execution | Orchestrator aggregates per-dimension estimates after all researchers complete |
| OBS-03 | Cost summary displayed in progress reports | STATE.md cumulative cost tracker + progress.md reads it |
</phase_requirements>

<research_summary>
## Summary

This phase modifies the GSD framework internals — specifically the `plan-phase.md` orchestrator, the `gsd-phase-researcher` agent, the `progress.md` workflow, and adds a new dimension catalog system. There are no third-party libraries involved. The entire implementation is file operations, markdown edits, and workflow logic changes.

The dimension catalog uses standalone Markdown files — one file per dimension — stored in `~/.claude/get-shit-done/dimensions/` (global defaults) and `.planning/dimensions/` (project overrides). This aligns with how GSD currently structures agents, templates, and references: one concern per file, human-readable and editable. The format gives Claude maximum flexibility to expand into full research prompts, while keeping the checklist display compact via a `short_description` frontmatter field.

Token tracking is the most architecturally constrained problem in this phase. Claude Code's `Task()` subagent calls do **not** expose per-subagent token counts to the markdown workflow orchestrator. The Claude Agent SDK's `result.modelUsage` field is only accessible in programmatic SDK usage, not in markdown-based orchestrators. The practical approach is **self-reported token estimation**: each researcher subagent uses `/cost` command output or a structured footer in its RESEARCH.md to report its own usage. The orchestrator then reads these reported figures and aggregates them. This is an approximation, but it is implementable within GSD's markdown workflow paradigm and is accurate enough for the "informed, not anxious" UX goal.

**Primary recommendation:** Implement dimensions as standalone YAML-frontmatter Markdown files; integrate selection into the `plan-phase.md` Step 5 (Handle Research) block using `AskUserQuestion`; implement token tracking via researcher-reported estimates written into a structured RESEARCH.md footer that the orchestrator reads after Task() completes.
</research_summary>

<standard_stack>
## Standard Stack

This phase has no external library dependencies. All implementation is within the GSD framework's own file system.

### Core (existing GSD infrastructure being extended)

| Component | Location | Purpose | What Changes |
|-----------|----------|---------|--------------|
| `plan-phase.md` | `~/.claude/get-shit-done/workflows/` | Orchestrates research, planning, checking | Add dimension selection block before researcher spawn |
| `gsd-phase-researcher.md` | `~/.claude/agents/` | Researcher agent prompt | Add dimension-awareness and token reporting footer |
| `progress.md` | `~/.claude/get-shit-done/workflows/` | Progress reporting | Add cost summary section reading STATE.md |
| `gsd-tools.cjs` | `~/.claude/get-shit-done/bin/` | CLI utility | Add `state update-cost` subcommand |
| `config.json` template | `~/.claude/get-shit-done/templates/` | Default project config | Add `token_prices` section |
| `state.md` template | `~/.claude/get-shit-done/templates/` | STATE.md template | Add `## Cost Tracker` section |

### New Artifacts

| Artifact | Location | Purpose |
|----------|----------|---------|
| `{dimension-name}.md` (×9) | `~/.claude/get-shit-done/dimensions/` | Dimension catalog files |
| `.planning/dimensions/` directory | Per-project | Project-level dimension overrides |

### No External Dependencies

This phase does not add npm packages, Python dependencies, or external APIs.

**Installation:** None required.
</standard_stack>

<architecture_patterns>
## Architecture Patterns

### Recommended File Structure (new)

```
~/.claude/get-shit-done/
├── dimensions/                         # Global dimension catalog (new)
│   ├── stack.md                        # Existing 4 dims migrated here
│   ├── features.md
│   ├── architecture.md
│   ├── pitfalls.md
│   ├── best-practices.md              # New in this phase
│   ├── data-structures.md             # New in this phase
│   ├── security-compliance.md         # New in this phase
│   ├── testing-strategies.md          # New in this phase
│   └── devops-deployment.md           # New in this phase

.planning/                              # Per-project
├── config.json                         # Add token_prices section
├── STATE.md                            # Add ## Cost Tracker section
└── dimensions/                         # Project overrides (new, optional)
    └── {custom-dim}.md
```

### Pattern 1: Dimension File Format

**What:** Standalone Markdown file with YAML frontmatter defining a research dimension.
**When to use:** All dimensions — both global defaults and project overrides.

```markdown
---
name: best-practices
short_description: Industry patterns and expert conventions for this domain
tags: [patterns, conventions, quality]
suggested_project_types: [web-app, cli, library, api, mobile]
---

Research best practices and industry conventions for this project's domain.

Investigate:
- Patterns expert practitioners use (and why)
- Standard conventions for code organization, naming, and structure
- Performance and maintainability trade-offs at scale
- Common "right way vs wrong way" debates with resolution
- Style guides and linting conventions adopted by major projects

Produce findings with confidence levels. Cite sources from official docs or
widely-adopted frameworks. Flag any findings where community consensus is weak.
```

**Frontmatter fields:**
- `name`: kebab-case identifier, matches filename
- `short_description`: ≤10 words shown in the checklist
- `tags`: for category grouping and future filtering (DIM-06 v2 prep)
- `suggested_project_types`: for dimension inference (DIM-05 v2 prep)

**Body:** The actual research prompt that the researcher subagent receives verbatim.

### Pattern 2: Dimension Selection Flow in plan-phase.md

**What:** A new block inserted into Step 5 (Handle Research) of `plan-phase.md`, before the researcher is spawned.
**When to use:** Every time a researcher is about to be spawned (unless `--skip-research`).

Pseudocode for the new block:

```
1. Load all dimension files:
   a. Read ~/.claude/get-shit-done/dimensions/*.md
   b. Read .planning/dimensions/*.md (if exists)
   c. Project dims override global dims on name collision

2. Claude infers relevant dimensions:
   - Analyze: project type from PROJECT.md, phase goal from ROADMAP.md
   - Pre-select dimensions appropriate to context

3. Present AskUserQuestion checklist:
   - header: "Research Dimensions"
   - question: "Which dimensions should the researcher cover?
                Estimated cost shown after selection."
   - options: one per dimension (name + short_description)
   - Include "Other — add a custom dimension" option

4. If user selects "Other":
   - Ask: "Name for this dimension?"
   - Ask: "Brief description of what to research?"
   - Claude expands into full research prompt
   - Save to .planning/dimensions/{slug}.md

5. Offer prompt editing:
   - "Customize any dimension prompt before researching?"
   - If yes: present dimensions one at a time with current prompt
   - User edits inline or Claude patches specific sections

6. Spawn researcher with selected dimensions injected into prompt
```

### Pattern 3: Token Tracking via Self-Reported Footer

**What:** Researcher subagent writes a structured token/cost footer into the RESEARCH.md file it produces. Orchestrator reads the footer after Task() completes.
**When to use:** Every researcher subagent invocation.

**Why self-reporting:** Claude Code's `Task()` in markdown workflows does not expose per-subagent token counts to the orchestrator. The Agent SDK's `result.modelUsage` is only available in programmatic SDK usage. Self-reporting is the only mechanism available in the markdown workflow paradigm.

**Researcher writes this footer into RESEARCH.md:**
```markdown
<token_report>
dimension: stack
model: claude-sonnet-4-5
input_tokens: 8420
output_tokens: 2150
estimated_cost_usd: 0.058
</token_report>
```

**Orchestrator reads after Task() completes:**
```bash
# After researcher Task() returns, read the token_report from RESEARCH.md
RESEARCH_PATH="${PHASE_DIR}/${PADDED_PHASE}-RESEARCH.md"
TOKEN_DATA=$(grep -A 6 "<token_report>" "$RESEARCH_PATH" | grep -E "dimension:|model:|input_tokens:|output_tokens:|estimated_cost_usd:")
```

**Orchestrator displays live output after each dimension:**
```
Stack research complete
  Input: 8,420 tokens  Output: 2,150 tokens
  Cost: ~$0.058 (claude-sonnet-4-5)
```

### Pattern 4: Cost Accumulation in STATE.md

**What:** A `## Cost Tracker` section in STATE.md that accumulates per-phase and total project costs.
**When to use:** After each phase's research completes; read by `/gsd:progress`.

**STATE.md new section template:**
```markdown
## Cost Tracker

**Total project cost:** $0.00
**Last updated:** 2026-02-16

| Phase | Dimensions | Input Tokens | Output Tokens | Cost |
|-------|-----------|-------------|--------------|------|
| - | - | - | - | - |
```

**gsd-tools.cjs new subcommand:**
```
state update-cost --phase N --dimensions "stack,architecture" \
  --input-tokens 42000 --output-tokens 8500 --cost-usd 0.25
```

This updates the Cost Tracker table in STATE.md and recalculates the total.

### Pattern 5: Price Configuration in config.json

**What:** A `token_prices` section in `.planning/config.json` with model-specific rates, used by the researcher to calculate estimated cost.
**When to use:** Researcher reads this at start; calculates cost from reported token counts.

```json
{
  "token_prices": {
    "claude-opus-4-6":    { "input_per_mtok": 5.00, "output_per_mtok": 25.00 },
    "claude-sonnet-4-5":  { "input_per_mtok": 3.00, "output_per_mtok": 15.00 },
    "claude-haiku-4-5":   { "input_per_mtok": 1.00, "output_per_mtok": 5.00 }
  }
}
```

The researcher reads its own model from the Task() invocation context, looks up the price, and writes the cost estimate into the token_report footer.

### Anti-Patterns to Avoid

- **Trying to intercept Task() return metadata:** The markdown workflow Task() call only returns text output. There is no mechanism to intercept `modelUsage` from within a markdown orchestrator. The self-reporting pattern is the correct approach.
- **Single catalog file:** A single `dimensions.json` or `dimensions.md` is harder to edit, harder to extend, and harder to override at project level. Standalone files per dimension follow GSD's existing pattern (one agent per file, one template per file).
- **Blocking on dimension count:** No hard limit on dimensions. The cost display is the feedback loop. Imposing a limit adds friction without value.
- **Modifying existing 4 dimension research flow:** The current researcher agent handles 4 hardcoded dimensions in its prompt. This phase should parameterize the dimension list, not fork the agent logic. The existing RESEARCH.md format stays intact; the agent just runs each selected dimension and populates the appropriate sections.
</architecture_patterns>

<dont_hand_roll>
## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| File loading with precedence | Custom merge logic for global vs project dims | Simple: load global, load project, overwrite by name match | Two-level override is the only requirement; a full config merge system is over-engineered |
| Token counting | Client-side tokenizer or external API call | `/cost` output or Claude's self-knowledge of turn token counts | Claude Code already tracks session tokens; the researcher can access its own usage via `/cost` in Bash or report its approximation |
| Dimension selection UI | Custom rendered checklist | `AskUserQuestion` with checkbox-style options | Already used throughout GSD; native to the workflow pattern; headers must stay ≤12 chars (existing validation) |
| Cost display formatting | Custom currency formatter | Simple inline string interpolation `$0.058` | No edge cases warrant a library at this scale |
| Dimension prompt templating | Mustache/Handlebars | Plain string replacement in researcher prompt | Dimensions are plain markdown; no template language needed |

**Key insight:** This phase is entirely within the GSD framework's markdown-and-bash paradigm. The correct tool for every problem here is already present in the codebase.
</dont_hand_roll>

<common_pitfalls>
## Common Pitfalls

### Pitfall 1: Attempting Programmatic Token Capture from Task()

**What goes wrong:** Orchestrator tries to parse token usage from Task() metadata, or tries to call `gsd-tools.cjs websearch` / `/cost` from within an orchestrator that doesn't have bash access at that moment.
**Why it happens:** The Claude Agent SDK's `result.modelUsage` is documented and seems applicable, but it requires programmatic SDK usage — not available in markdown workflow Task() calls.
**How to avoid:** Use the self-reporting pattern. The researcher subagent has full tool access including Bash. It can run bash to get its context window usage or use Claude's knowledge of its own token consumption to write the `<token_report>` footer.
**Warning signs:** If the plan proposes reading `result.usage` from a Task() call, that's the wrong path.

### Pitfall 2: AskUserQuestion Header Length Violation

**What goes wrong:** Workflow validation rejects checklist headers > 12 characters.
**Why it happens:** GSD's `AskUserQuestion` has a validated constraint: headers must be ≤ 12 characters. "Research Dimensions" is 20 characters.
**How to avoid:** Use short headers: "Dimensions" (10 chars), "Customize?" (10 chars), "Add custom?" (11 chars). The question field carries the full explanatory text.
**Warning signs:** Plan proposes headers like "Research Dimensions" or "Select Dimensions."

### Pitfall 3: Dimension Name Collision on Slug Generation

**What goes wrong:** User creates a custom dimension with a name that collides with a global default (e.g., "stack"), silently overwriting the global.
**Why it happens:** Project-level precedence is by design, but a user creating a custom "stack" dimension while expecting the global stack dimension to still run is a UX surprise.
**How to avoid:** When a user adds a custom dimension, check for name collision with globals. If collision: "This matches an existing dimension. Overriding it means the global 'stack' template won't run. Is that what you want?" with options [Override | Use different name].
**Warning signs:** User sees fewer dimensions than expected after adding a custom one.

### Pitfall 4: Token Estimate Accuracy Drift

**What goes wrong:** Researcher's self-reported token count diverges significantly from actual API usage.
**Why it happens:** Claude's own introspective token estimates are imprecise. The actual API token count includes system prompt tokens, tool definitions, and cache reads/writes that may not be visible to the model's self-assessment.
**How to avoid:** Frame cost displays as "estimated" not "exact." Display as `~$0.058 (estimated)`. The "informed, not anxious" UX principle applies — approximate figures are fine as long as they're labeled as such.
**Warning signs:** If the plan proposes exact-to-the-cent cost reporting, that implies a precision that self-reporting cannot deliver.

### Pitfall 5: Breaking Existing researcher Agent Format

**What goes wrong:** The parameterized dimension list changes the structure of RESEARCH.md, breaking the planner's expectation of the file format.
**Why it happens:** Adding per-dimension token footers and changing section order can break downstream planner consumption.
**How to avoid:** Keep the existing RESEARCH.md main body structure intact. The `<token_report>` blocks go at the bottom, after all existing sections. The planner reads RESEARCH.md and can ignore `<token_report>` tags — it only reads the named sections it knows about.
**Warning signs:** Planner agent throws "research not found" or misreads dimension findings.

### Pitfall 6: STATE.md Bloat from Cost Tracker

**What goes wrong:** The Cost Tracker table in STATE.md grows without bound across many phases, violating the 100-line STATE.md size constraint.
**Why it happens:** Each phase adds a row. A 5-phase project adds 5 rows, which is fine. A 20-phase project might not be.
**How to avoid:** The Cost Tracker table is capped at the last 10 phases; older entries are pruned when the table exceeds that. The total project cost field always shows the running cumulative.
**Warning signs:** STATE.md exceeds 100 lines.
</common_pitfalls>

<code_examples>
## Code Examples

### Dimension File: best-practices.md

```markdown
---
name: best-practices
short_description: Expert patterns and industry conventions
tags: [patterns, conventions, quality, architecture]
suggested_project_types: [web-app, cli, library, api, mobile, service]
---

Research expert best practices and industry conventions for this project's domain.

Investigate:
- Patterns expert practitioners adopt for this type of project (not just what works,
  but what the community has converged on as "the right way")
- Standard naming, organization, and structure conventions from widely-adopted projects
- Performance and maintainability patterns proven at scale
- Key "always do / never do" rules for this domain with concrete rationale
- Relevant style guides, linting configs, and automated enforcement tools
- Anti-patterns that beginners adopt and experts avoid

Produce findings with confidence levels. Focus on patterns with broad community
adoption, not personal preference. Cite authoritative sources.
```

### Dimension Selection in plan-phase.md (pseudocode block)

```markdown
## 5.1 Load Dimension Catalog

```bash
# Load global dims
GLOBAL_DIMS=$(ls ~/.claude/get-shit-done/dimensions/*.md 2>/dev/null)
# Load project dims (if any)
PROJECT_DIMS=$(ls .planning/dimensions/*.md 2>/dev/null)
# Combined: project overrides global on name collision
```

For each .md file, extract frontmatter fields: name, short_description.

## 5.2 Infer Relevant Dimensions

Based on PROJECT.md project type and current phase goal, pre-select dimensions.
Default for web-app: stack, features, architecture, pitfalls, security-compliance.
Default for CLI: stack, architecture, pitfalls, best-practices.
Default for library: stack, architecture, best-practices, testing-strategies.

## 5.3 Present Checklist

Use AskUserQuestion:
- header: "Dimensions" (≤12 chars)
- question: "Which dimensions should the researcher cover for Phase {N}?
  Claude pre-selected based on project type ({type}) and phase goal."
- options: [
    "stack — {short_description}" (pre-selected),
    "features — {short_description}" (pre-selected),
    "architecture — {short_description}" (pre-selected),
    "pitfalls — {short_description}" (pre-selected),
    "best-practices — {short_description}" (pre-selected for web-app),
    "security-compliance — {short_description}",
    "testing-strategies — {short_description}",
    "devops-deployment — {short_description}",
    "data-structures — {short_description}",
    "Other — add a custom dimension"
  ]
- Allow multi-select; user toggles on/off

## 5.4 Handle "Other"

If "Other" selected:
  Ask: name? → generate slug
  Ask: "Brief description — what should the researcher investigate?"
  Claude expands into full research prompt
  Write to .planning/dimensions/{slug}.md
  Add to selected set

## 5.5 Offer Prompt Editing

AskUserQuestion:
- header: "Customize?" (≤12 chars)
- question: "Customize any dimension prompt before researching? (optional)"
- options: [selected dimension names..., "No — research as configured"]
```

### Token Report Footer (researcher writes this to RESEARCH.md)

```markdown
<token_report>
dimension: all-selected
model: claude-sonnet-4-5
input_tokens: 42180
output_tokens: 8350
estimated_cost_usd: 0.251
note: estimated from session context; actual may vary ±15%
</token_report>
```

### STATE.md Cost Tracker Section

```markdown
## Cost Tracker

**Total project cost:** $0.51 (estimated)
**Last updated:** 2026-02-16

| Phase | Dimensions Run | Input Tokens | Output Tokens | Cost (est.) |
|-------|---------------|-------------|--------------|-------------|
| 1 | stack, architecture, pitfalls, security | 42,180 | 8,350 | $0.251 |
| 2 | stack, features, pitfalls | 31,200 | 6,100 | $0.185 |
| 3 | architecture, best-practices, testing | 38,500 | 7,400 | $0.227 |
```

### config.json token_prices Section

```json
{
  "token_prices": {
    "claude-opus-4-6":   { "input_per_mtok": 5.00,  "output_per_mtok": 25.00 },
    "claude-opus-4-5":   { "input_per_mtok": 5.00,  "output_per_mtok": 25.00 },
    "claude-sonnet-4-5": { "input_per_mtok": 3.00,  "output_per_mtok": 15.00 },
    "claude-haiku-4-5":  { "input_per_mtok": 1.00,  "output_per_mtok": 5.00 }
  }
}
```

### progress.md Cost Summary Block

In the `/gsd:progress` report output, add after the progress bar:

```markdown
## Cost Summary

**Project cost to date:** $0.51 (estimated research tokens)

| Phase | Cost | Dimensions |
|-------|------|-----------|
| 1. Selectable Research Dimensions | $0.251 | stack, architecture, pitfalls, security |
| 2. Idea Capture System | $0.185 | stack, features, pitfalls |

*Costs are estimates based on self-reported token usage. Actual API billing may differ.*
```
</code_examples>

<sota_updates>
## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Fixed 4-dim research (STACK, FEATURES, ARCHITECTURE, PITFALLS) | Selectable catalog of 9+ dimensions, user-controlled | This phase | Users stop paying for irrelevant research dimensions |
| No cost visibility | Per-dimension and per-phase cost estimates in STATE.md and progress reports | This phase | Users can calibrate research depth to their budget |
| Global defaults only | Two-tier storage: `~/.claude/get-shit-done/dimensions/` + `.planning/dimensions/` | This phase | Per-project dimension customization without affecting global defaults |

**Current Anthropic pricing (verified 2026-02-16):**
- Claude Opus 4.6: $5 input / $25 output per MTok
- Claude Sonnet 4.5: $3 input / $15 output per MTok
- Claude Haiku 4.5: $1 input / $5 output per MTok

Source: [Anthropic Pricing Docs](https://platform.claude.com/docs/en/about-claude/pricing)

**Token tracking architecture note (verified 2026-02-16):**
Claude Code's markdown workflow `Task()` calls do not expose per-subagent token usage to the orchestrator. The Claude Agent SDK's `result.modelUsage` is only available in programmatic SDK usage. GSD's markdown orchestrators must use self-reporting from within the subagent.

Source: [Claude Code Costs Docs](https://code.claude.com/docs/en/costs), [Agent SDK Cost Tracking](https://platform.claude.com/docs/en/agent-sdk/cost-tracking)
</sota_updates>

<open_questions>
## Open Questions

1. **How does the researcher know its own token count?**
   - What we know: Claude has approximate awareness of its own context window usage. It can also run `bash: /cost` within its session if the tool is available. The self-reporting pattern is the only viable approach in the markdown workflow paradigm.
   - What's unclear: How accurate Claude's self-reported token count will be in practice. The researcher operates within a subagent Task() and may not have perfect visibility into system prompt tokens, caching tokens, etc.
   - Recommendation: Accept ±15-20% estimation error as documented in the token_report footer. Label all costs as "estimated." If accuracy becomes a problem in practice, a future phase could add a PostToolUse hook that captures actual usage from the API response headers.

2. **How does multi-dimension research map to RESEARCH.md structure?**
   - What we know: The existing RESEARCH.md has one section per topic (## Standard Stack, ## Architecture Patterns, etc.), corresponding to the current 4 fixed dimensions.
   - What's unclear: If a user selects 7 dimensions, does each get a new top-level section? Does the planner need updating to handle new section names?
   - Recommendation: Keep existing section names for the 4 legacy dimensions. New dimensions (best-practices, security-compliance, etc.) add their own `## {Dimension Name}` sections. The planner reads all `##` sections and uses what's relevant. The RESEARCH.md format is already consumed as raw markdown context, not by section name lookup.

3. **Should dimension selection be persisted per-phase or re-asked each run?**
   - What we know: Decisions says selection happens during `/gsd:plan-phase`. There's no explicit lock-in mechanism.
   - What's unclear: If a user re-runs `--research` on an existing phase, should it remember prior selection?
   - Recommendation: Save selected dimensions into the RESEARCH.md frontmatter (new field: `dimensions_used: [stack, architecture, ...]`). On re-research, pre-fill the checklist from this field. User can modify. This is natural and consistent with GSD's "existing artifacts" pattern.
</open_questions>

<sources>
## Sources

### Primary (HIGH confidence)
- Direct codebase inspection of `/Users/thelorax/.claude/get-shit-done/` — all workflow files, templates, and gsd-tools.cjs read line-by-line
- [Anthropic Pricing Docs](https://platform.claude.com/docs/en/about-claude/pricing) — model pricing table verified 2026-02-16
- [Claude Code Costs Docs](https://code.claude.com/docs/en/costs) — token usage mechanism, `/cost` command behavior
- [Claude Agent SDK Cost Tracking](https://platform.claude.com/docs/en/agent-sdk/cost-tracking) — `result.modelUsage` structure, confirms it's SDK-only (not markdown workflow)

### Secondary (MEDIUM confidence)
- [Claude Code Sub-agents Docs](https://code.claude.com/docs/en/sub-agents) — Task() return value behavior, SubagentStop hooks; confirms no per-task token metadata in markdown orchestrators

### Tertiary (LOW confidence)
- None — all critical claims verified with primary sources.
</sources>

<metadata>
## Metadata

**Confidence breakdown:**
- Dimension catalog design: HIGH — follows GSD's existing file-per-concern pattern; directly observed in templates/ and agents/
- AskUserQuestion integration: HIGH — pattern is used throughout existing workflows; header length constraint documented
- Token tracking mechanism: HIGH — verified against official Claude Code docs that Task() does not expose token metadata; self-reporting is the only viable path
- Current model pricing: HIGH — read directly from official Anthropic pricing page 2026-02-16

**Research date:** 2026-02-16
**Valid until:** 2026-03-16 (pricing may change; implementation patterns are stable)

---
*Phase: 01-selectable-research-dimensions-observability*
*Research completed: 2026-02-16*
*Ready for planning: yes*
</metadata>
