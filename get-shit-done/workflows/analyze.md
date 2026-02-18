<purpose>
Run specialist analysis on a phase. Specialists analyze phase context through domain-specific lenses and produce actionable recommendations that feed into the planner.

Specialists are pluggable: drop a `.md` file into `get-shit-done/specialists/` (global) or `.planning/specialists/` (project) to add a new specialist.
</purpose>

<required_reading>
Read all files referenced by the invoking prompt's execution_context before starting.

@~/.claude/get-shit-done/references/ui-brand.md
</required_reading>

<process>

## 1. Initialize

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.cjs init phase-op "${PHASE}")
```

Parse JSON for: `commit_docs`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_research`, `has_context`, `has_plans`, `planning_exists`.

**If `planning_exists` is false:** Error — run `/gsd:new-project` first.
**If `phase_found` is false:** Error — phase not found in roadmap. Use `/gsd:progress` to see available phases.

Resolve specialist model:
```bash
SPECIALIST_MODEL=$(node ~/.claude/get-shit-done/bin/gsd-tools.cjs config-get model_profile 2>/dev/null || echo "balanced")
```

Map profile to model: `quality` → `sonnet`, `balanced` → `sonnet`, `budget` → `haiku`.

## 2. Load Specialist Catalog

Load global specialists:
```bash
GLOBAL_SPECS=$(ls ~/.claude/get-shit-done/specialists/*.md 2>/dev/null)
```

Load project specialists (override global on name collision):
```bash
PROJECT_SPECS=$(ls .planning/specialists/*.md 2>/dev/null)
```

For each `.md` file, extract frontmatter fields: `name`, `short_description`, `tags`, `triggers`, `authority`. Parse YAML frontmatter between `---` delimiters.

Build combined list: `AVAILABLE_SPECS` (array of objects with `name`, `short_description`, `triggers`, `authority`, `source` (global|project), `prompt_body`).

Project specialists override global specialists on name collision (compare by `name` field value).

**If no specialists found:** Display message and exit:
```
No specialists found.

Add specialist definitions to:
  Global: ~/.claude/get-shit-done/specialists/
  Project: .planning/specialists/
```

## 3. Select Specialists

**If specific specialist name provided in arguments:**
- Find matching specialist in `AVAILABLE_SPECS`
- If not found: error with available names
- If found: `SELECTED_SPECS = [matching specialist]`

**If `--all` flag:**
- `SELECTED_SPECS = AVAILABLE_SPECS`

**Otherwise (interactive selection):**

Use AskUserQuestion (multiSelect: true):
- header: "Specialists"
- question: "Which specialists should analyze Phase {phase_number}: {phase_name}?"
- options: One option per available specialist, formatted as `{name} — {short_description}`. Mark specialists with `triggers` containing the current context (e.g., `plan-phase`) as pre-selected.

Store selections in `SELECTED_SPECS`.

**If no specialists selected:** Display "No specialists selected." and exit.

## 4. Gather Context

Read phase context for injection into specialist agents:

```bash
PHASE_DESC=$(node ~/.claude/get-shit-done/bin/gsd-tools.cjs roadmap get-phase "${PHASE}" | jq -r '.section')
REQUIREMENTS=$(cat .planning/REQUIREMENTS.md 2>/dev/null)
ROADMAP=$(cat .planning/ROADMAP.md 2>/dev/null)
STATE=$(cat .planning/STATE.md 2>/dev/null)
CONTEXT_MD=$(cat "$phase_dir"/*-CONTEXT.md 2>/dev/null)
RESEARCH_MD=$(cat "$phase_dir"/*-RESEARCH.md 2>/dev/null)
```

Extract phase requirement IDs from roadmap:
```bash
PHASE_REQ_IDS=$(echo "$ROADMAP" | grep -A5 "Phase ${PHASE}" | grep -i "Requirements:" | head -1 | sed 's/.*Requirements:\*\*\s*//' | sed 's/[\[\]]//g')
```

## 5. Spawn Specialist Agents

Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► SPECIALIST ANALYSIS — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning {N} specialist(s) in parallel...
  → {spec_name} analysis
  [repeat for each selected specialist]
```

Create specialists output directory:
```bash
mkdir -p "${phase_dir}/specialists"
```

For each specialist in `SELECTED_SPECS`, spawn in parallel:

```
Task(prompt="First, read ~/.claude/agents/gsd-specialist.md for your role and instructions.

<specialist_body>
{prompt_body from specialist catalog entry}
</specialist_body>

<specialist_meta>
name: {specialist_name}
authority:
  can_create_tasks: {from frontmatter}
  can_create_phases: {from frontmatter}
</specialist_meta>

<phase_context>
**Phase:** {phase_number} - {phase_name}
**Phase Description:** {phase_desc}
**Phase Requirement IDs:** {phase_req_ids}

**Requirements:**
{requirements}

**Roadmap:**
{roadmap}

**State:**
{state}

**CONTEXT.md (user decisions — LOCKED):**
{context_md}

**RESEARCH.md:**
{research_md}
</phase_context>

<output>
Write to: {phase_dir}/specialists/{specialist_name}-ANALYSIS.md
</output>
", subagent_type="general-purpose", model="{specialist_model}", description="{specialist_name} analysis")
```

## 6. Handle Returns

For each specialist return:

- **`## ANALYSIS COMPLETE`:** Display confirmation:
  ```
  ✓ {specialist_name} — {recommendation_count} recommendations, {hotspot_count} hotspots
  ```
- **`## ANALYSIS BLOCKED`:** Display blocker:
  ```
  ✗ {specialist_name} — BLOCKED: {reason}
  ```

## 7. Handle Urgent Findings

Scan all completed analysis files for non-empty `## Urgent Findings` sections:

```bash
for f in "${phase_dir}/specialists/"*-ANALYSIS.md; do
  URGENT=$(sed -n '/^## Urgent Findings/,/^## /p' "$f" | grep -v '^## ' | grep -v '^$')
  if [ -n "$URGENT" ]; then
    echo "URGENT in $(basename $f): $URGENT"
  fi
done
```

**If urgent findings exist:**

Display each urgent finding, then use AskUserQuestion for each:
- header: "Urgent"
- question: "{specialist_name} flagged an urgent finding: {summary}. How should we handle this?"
- options:
  - "Insert phase" — Create a new phase to address this
  - "Note for planning" — Pass to planner as high-priority input
  - "Dismiss" — Not actionable right now

**If "Insert phase":** Note for user to run `/gsd:insert-phase` after analysis completes.
**If "Note for planning":** Mark finding as `priority: high` in analysis file.
**If "Dismiss":** No action.

## 8. Commit

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.cjs commit "docs(${padded_phase}): specialist analysis" --files "${phase_dir}/specialists/"
```

## 9. Present Results

Display summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► SPECIALIST ANALYSIS COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {X}: {Name}** — {N} specialist(s)

| Specialist | Recommendations | Hotspots | Risks | Status |
|------------|-----------------|----------|-------|--------|
| {name} | {N} | {N} | {N} | ✓ Complete |

Files: {phase_dir}/specialists/

───────────────────────────────────────────────────────────────

## ▶ Next Up

**Plan Phase {X}** — specialist analysis will feed into planning

/gsd:plan-phase {X}

<sub>/clear first → fresh context window</sub>

───────────────────────────────────────────────────────────────

**Also available:**
- cat {phase_dir}/specialists/*-ANALYSIS.md — review analysis
- /gsd:analyze {X} {another-specialist} — run additional specialist

───────────────────────────────────────────────────────────────
```

</process>

<success_criteria>
- [ ] Phase validated against roadmap
- [ ] Specialist catalog loaded (global + project, project overrides)
- [ ] Specialists selected (interactive, named, or --all)
- [ ] Phase context gathered and injected
- [ ] Specialist agents spawned in parallel
- [ ] Returns handled (complete or blocked)
- [ ] Urgent findings surfaced and handled
- [ ] Analysis files committed
- [ ] User knows next steps
</success_criteria>
