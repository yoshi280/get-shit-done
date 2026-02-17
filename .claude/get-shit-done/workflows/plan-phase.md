<purpose>
Create executable phase prompts (PLAN.md files) for a roadmap phase with integrated research and verification. Default flow: Research (if needed) -> Plan -> Verify -> Done. Orchestrates gsd-phase-researcher, gsd-planner, and gsd-plan-checker agents with a revision loop (max 3 iterations).
</purpose>

<required_reading>
Read all files referenced by the invoking prompt's execution_context before starting.

@/Users/thelorax/.claude/get-shit-done/references/ui-brand.md
</required_reading>

<process>

## 1. Initialize

Load all context in one call (include file contents to avoid redundant reads):

```bash
INIT_RAW=$(node /Users/thelorax/.claude/get-shit-done/bin/gsd-tools.cjs init plan-phase "$PHASE" --include state,roadmap,requirements,context,research,verification,uat)
# Large payloads are written to a tmpfile — output starts with @file:/path
if [[ "$INIT_RAW" == @file:* ]]; then
  INIT_FILE="${INIT_RAW#@file:}"
  INIT=$(cat "$INIT_FILE")
  rm -f "$INIT_FILE"
else
  INIT="$INIT_RAW"
fi
```

Parse JSON for: `researcher_model`, `planner_model`, `checker_model`, `research_enabled`, `plan_checker_enabled`, `commit_docs`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_research`, `has_context`, `has_plans`, `plan_count`, `planning_exists`, `roadmap_exists`.

**File contents (from --include):** `state_content`, `roadmap_content`, `requirements_content`, `context_content`, `research_content`, `verification_content`, `uat_content`. These are null if files don't exist.

**If `planning_exists` is false:** Error — run `/gsd:new-project` first.

## 2. Parse and Normalize Arguments

Extract from $ARGUMENTS: phase number (integer or decimal like `2.1`), flags (`--research`, `--skip-research`, `--gaps`, `--skip-verify`).

**If no phase number:** Detect next unplanned phase from roadmap.

**If `phase_found` is false:** Validate phase exists in ROADMAP.md. If valid, create the directory using `phase_slug` and `padded_phase` from init:
```bash
mkdir -p ".planning/phases/${padded_phase}-${phase_slug}"
```

**Existing artifacts from init:** `has_research`, `has_plans`, `plan_count`.

## 3. Validate Phase

```bash
PHASE_INFO=$(node /Users/thelorax/.claude/get-shit-done/bin/gsd-tools.cjs roadmap get-phase "${PHASE}")
```

**If `found` is false:** Error with available phases. **If `found` is true:** Extract `phase_number`, `phase_name`, `goal` from JSON.

## 4. Load CONTEXT.md

Use `context_content` from init JSON (already loaded via `--include context`).

**CRITICAL:** Use `context_content` from INIT — pass to researcher, planner, checker, and revision agents.

If `context_content` is not null, display: `Using phase context from: ${PHASE_DIR}/*-CONTEXT.md`

**If `context_content` is null (no CONTEXT.md exists):**

Use AskUserQuestion:
- header: "No context"
- question: "No CONTEXT.md found for Phase {X}. Plans will use research and requirements only — your design preferences won't be included. Continue or capture context first?"
- options:
  - "Continue without context" — Plan using research + requirements only
  - "Run discuss-phase first" — Capture design decisions before planning

If "Continue without context": Proceed to step 5.
If "Run discuss-phase first": Display `/gsd:discuss-phase {X}` and exit workflow.

## 5. Handle Research

**Skip if:** `--gaps` flag, `--skip-research` flag, or `research_enabled` is false (from init) without `--research` override.

**If `has_research` is true (from init) AND no `--research` flag:** Use existing, skip to step 6.

**If RESEARCH.md missing OR `--research` flag:**

### Step 5.1: Load Dimension Catalog

```bash
# Load global dims
GLOBAL_DIMS=$(ls ~/.claude/get-shit-done/dimensions/*.md 2>/dev/null)
# Load project dims (if any)
PROJECT_DIMS=$(ls .planning/dimensions/*.md 2>/dev/null)
```

For each .md file, extract frontmatter fields `name` and `short_description`. Parse each file's YAML frontmatter between the `---` delimiters using a bash loop or inline awk. Project dims override global dims on name collision (compare by `name` field value). Build a combined list: `AVAILABLE_DIMS` (array of objects with `name`, `short_description`, `source` (global|project), `prompt_body`).

Example parse (repeat for each file):
```bash
NAME=$(grep '^name:' "$DIM_FILE" | head -1 | sed 's/^name: *//')
SHORT_DESC=$(grep '^short_description:' "$DIM_FILE" | head -1 | sed 's/^short_description: *//')
```

### Step 5.2: Infer Relevant Dimensions

Claude reads the `project_type` from PROJECT.md and the current phase goal from ROADMAP.md. Pre-select dimensions based on project type:

| Project Type | Pre-selected Dimensions |
|---|---|
| web-app | stack, features, architecture, pitfalls, security-compliance |
| cli | stack, architecture, pitfalls, best-practices |
| library | stack, architecture, best-practices, testing-strategies |
| api/service | stack, architecture, pitfalls, security-compliance, data-structures |
| mobile | stack, features, architecture, pitfalls, security-compliance |
| fallback (unknown) | stack, features, architecture, pitfalls |

Store pre-selected names in `PRESELECTED_DIMS` (array of name strings).

### Step 5.3: Present Checklist

Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► RESEARCHING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Use AskUserQuestion:
- **header:** "Dimensions"
- **question:** "Which dimensions should the researcher cover for Phase {N}?\nClaude pre-selected based on project type ({type}) and phase goal."
- **options:** One option per available dimension, formatted as `{name} -- {short_description}`. Mark pre-selected ones as the default selection. Add `Other -- add a custom dimension` as the last option.
- Allow multi-select (user toggles on/off).

Store the user's selections in `SELECTED_DIMS`.

### Step 5.4: Handle "Other"

If "Other" is in `SELECTED_DIMS`:

1. Use AskUserQuestion with header "New Dim" and question "Name for this custom dimension? (used as slug, e.g. performance-optimization)"
2. Use AskUserQuestion with header "Describe It" and question "What should the researcher investigate for this dimension? (brief description)"
3. Claude generates a kebab-case slug from the name.
4. Claude expands the brief description into a full research prompt body (10-15 lines), written in the same imperative style as the existing dimension files.
5. **Name collision check:** If slug matches an existing global dim `name`:
   - Use AskUserQuestion with header "Collision" and question "'{name}' matches an existing global dimension. Override it (your version replaces the global for this project) or use a different name?"
   - options: `Override -- use my custom version`, `Rename -- enter a different name`
   - If "Rename": repeat slug/name collection (step 1 above).
6. Write to `.planning/dimensions/{slug}.md` with full YAML frontmatter and prompt body:

```markdown
---
name: {slug}
short_description: {brief description, <=10 words}
tags: [custom]
suggested_project_types: []
---

{expanded research prompt body}
```

7. Add the new dimension to `SELECTED_DIMS` (replace "Other" entry).

### Step 5.5: Offer Prompt Editing

Use AskUserQuestion:
- **header:** "Customize?"
- **question:** "Customize any dimension prompt before researching? (optional -- select dimension to edit or skip)"
- **options:** One option per name in `SELECTED_DIMS` + `No -- research as configured`

If user selects a dimension name:
- Display the current prompt body for that dimension.
- Use AskUserQuestion with header "Edit Prompt" and question "Paste your replacement prompt (or describe targeted edits):"
- Apply the edits and save to `.planning/dimensions/{slug}.md` (project-level override — does not modify global dims).
- Return to the edit loop (offer again in case they want to edit another dimension).

When user selects "No -- research as configured": proceed to researcher spawn.

### Spawn gsd-phase-researcher

```bash
PHASE_DESC=$(node /Users/thelorax/.claude/get-shit-done/bin/gsd-tools.cjs roadmap get-phase "${PHASE}" | jq -r '.section')
# Use requirements_content from INIT (already loaded via --include requirements)
REQUIREMENTS=$(echo "$INIT" | jq -r '.requirements_content // empty' | grep -A100 "## Requirements" | head -50)
PHASE_REQ_IDS=$(echo "$INIT" | jq -r '.roadmap_content // empty' | grep -i "Requirements:" | head -1 | sed 's/.*Requirements:\*\*\s*//' | sed 's/[\[\]]//g' | tr ',' '\n' | sed 's/^ *//;s/ *$//' | grep -v '^$' | tr '\n' ',' | sed 's/,$//')
STATE_SNAP=$(node /Users/thelorax/.claude/get-shit-done/bin/gsd-tools.cjs state-snapshot)
# Extract decisions from state-snapshot JSON: jq '.decisions[] | "\(.phase): \(.summary) - \(.rationale)"'

# Build selected_dimensions block from SELECTED_DIMS
# For each dim name in SELECTED_DIMS: read its prompt body from the resolved file path
# (project override if exists, otherwise global), then format as XML entries
SELECTED_DIMS_BLOCK=""
for DIM_NAME in "${SELECTED_DIMS[@]}"; do
  DIM_FILE=".planning/dimensions/${DIM_NAME}.md"
  if [ ! -f "$DIM_FILE" ]; then
    DIM_FILE="${HOME}/.claude/get-shit-done/dimensions/${DIM_NAME}.md"
  fi
  if [ -f "$DIM_FILE" ]; then
    # Strip YAML frontmatter (between first pair of --- delimiters), keep body
    DIM_BODY=$(awk '/^---/{c++; if(c==2){found=1; next}} found{print}' "$DIM_FILE")
    SELECTED_DIMS_BLOCK+="<dimension>\n<name>${DIM_NAME}</name>\n<prompt>${DIM_BODY}</prompt>\n</dimension>\n"
  fi
done
```

Research prompt:

```markdown
<objective>
Research how to implement Phase {phase_number}: {phase_name}
Answer: "What do I need to know to PLAN this phase well?"
</objective>

<phase_context>
IMPORTANT: If CONTEXT.md exists below, it contains user decisions from /gsd:discuss-phase.
- **Decisions** = Locked — research THESE deeply, no alternatives
- **Claude's Discretion** = Freedom areas — research options, recommend
- **Deferred Ideas** = Out of scope — ignore

{context_content}
</phase_context>

<additional_context>
**Phase description:** {phase_description}
**Phase requirement IDs (MUST address):** {phase_req_ids}
**Requirements:** {requirements}
**Prior decisions:** {decisions}
</additional_context>

<selected_dimensions>
Research ONLY the following dimensions. Do NOT fall back to hardcoded defaults if this block is present.
{selected_dims_block}
</selected_dimensions>

<output>
Write to: {phase_dir}/{phase_num}-RESEARCH.md
</output>
```

```
Task(
  prompt="First, read /Users/thelorax/.claude/agents/gsd-phase-researcher.md for your role and instructions.\n\n" + research_prompt,
  subagent_type="general-purpose",
  model="{researcher_model}",
  description="Research Phase {phase}"
)
```

### Handle Researcher Return

- **`## RESEARCH COMPLETE`:** Parse token report and display confirmation (see below), continue to step 6
- **`## RESEARCH BLOCKED`:** Display blocker, offer: 1) Provide context, 2) Skip research, 3) Abort

### Display Live Token Usage

After the researcher Task() returns, parse the `<token_report>` from the resulting RESEARCH.md:

```bash
RESEARCH_FILE="${PHASE_DIR}/${PADDED_PHASE}-RESEARCH.md"
TOKEN_REPORT=$(awk '/<token_report>/{found=1; next} /<\/token_report>/{found=0} found{print}' "$RESEARCH_FILE" 2>/dev/null)

# Parse individual fields from token_report
TR_DIMENSIONS=$(echo "$TOKEN_REPORT" | grep '^dimensions:' | sed 's/^dimensions: *//')
TR_MODEL=$(echo "$TOKEN_REPORT" | grep '^model:' | sed 's/^model: *//')
TR_INPUT=$(echo "$TOKEN_REPORT" | grep '^input_tokens:' | sed 's/^input_tokens: *//')
TR_OUTPUT=$(echo "$TOKEN_REPORT" | grep '^output_tokens:' | sed 's/^output_tokens: *//')
TR_COST=$(echo "$TOKEN_REPORT" | grep '^estimated_cost_usd:' | sed 's/^estimated_cost_usd: *//')
```

Display formatted output (only if token_report was found in RESEARCH.md):

```
Research complete
  Dimensions: {TR_DIMENSIONS}
  Input: {TR_INPUT} tokens  Output: {TR_OUTPUT} tokens
  Estimated cost: ~${TR_COST} ({TR_MODEL})
```

### Persist Research Cost to STATE.md

After displaying the token report, call gsd-tools.cjs state update-cost to persist the per-phase cost row:

```bash
if [ -n "$TR_DIMENSIONS" ]; then
  node ~/.claude/get-shit-done/bin/gsd-tools.cjs state update-cost \
    --phase "${PHASE_NUMBER}" \
    --dimensions "${TR_DIMENSIONS}" \
    --input-tokens "${TR_INPUT}" \
    --output-tokens "${TR_OUTPUT}" \
    --cost "${TR_COST}" \
    2>/dev/null || echo "Warning: state update-cost unavailable (Plan 03 not yet executed). Cost data displayed above."
fi
```

If the command fails or is unavailable, log a warning but do not abort — the cost data is already shown in the live display above.

## 6. Check Existing Plans

```bash
ls "${PHASE_DIR}"/*-PLAN.md 2>/dev/null
```

**If exists:** Offer: 1) Add more plans, 2) View existing, 3) Replan from scratch.

## 7. Use Context Files from INIT

All file contents are already loaded via `--include` in step 1 (`@` syntax doesn't work across Task() boundaries):

```bash
# Extract from INIT JSON (no need to re-read files)
STATE_CONTENT=$(echo "$INIT" | jq -r '.state_content // empty')
ROADMAP_CONTENT=$(echo "$INIT" | jq -r '.roadmap_content // empty')
REQUIREMENTS_CONTENT=$(echo "$INIT" | jq -r '.requirements_content // empty')
RESEARCH_CONTENT=$(echo "$INIT" | jq -r '.research_content // empty')
VERIFICATION_CONTENT=$(echo "$INIT" | jq -r '.verification_content // empty')
UAT_CONTENT=$(echo "$INIT" | jq -r '.uat_content // empty')
CONTEXT_CONTENT=$(echo "$INIT" | jq -r '.context_content // empty')
```

## 8. Spawn gsd-planner Agent

Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PLANNING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning planner...
```

Planner prompt:

```markdown
<planning_context>
**Phase:** {phase_number}
**Mode:** {standard | gap_closure}

**Project State:** {state_content}
**Roadmap:** {roadmap_content}
**Phase requirement IDs (every ID MUST appear in a plan's `requirements` field):** {phase_req_ids}
**Requirements:** {requirements_content}

**Phase Context:**
IMPORTANT: If context exists below, it contains USER DECISIONS from /gsd:discuss-phase.
- **Decisions** = LOCKED — honor exactly, do not revisit
- **Claude's Discretion** = Freedom — make implementation choices
- **Deferred Ideas** = Out of scope — do NOT include

{context_content}

**Research:** {research_content}
**Gap Closure (if --gaps):** {verification_content} {uat_content}
</planning_context>

<downstream_consumer>
Output consumed by /gsd:execute-phase. Plans need:
- Frontmatter (wave, depends_on, files_modified, autonomous)
- Tasks in XML format
- Verification criteria
- must_haves for goal-backward verification
</downstream_consumer>

<quality_gate>
- [ ] PLAN.md files created in phase directory
- [ ] Each plan has valid frontmatter
- [ ] Tasks are specific and actionable
- [ ] Dependencies correctly identified
- [ ] Waves assigned for parallel execution
- [ ] must_haves derived from phase goal
</quality_gate>
```

```
Task(
  prompt="First, read /Users/thelorax/.claude/agents/gsd-planner.md for your role and instructions.\n\n" + filled_prompt,
  subagent_type="general-purpose",
  model="{planner_model}",
  description="Plan Phase {phase}"
)
```

## 9. Handle Planner Return

- **`## PLANNING COMPLETE`:** Display plan count. If `--skip-verify` or `plan_checker_enabled` is false (from init): skip to step 13. Otherwise: step 10.
- **`## CHECKPOINT REACHED`:** Present to user, get response, spawn continuation (step 12)
- **`## PLANNING INCONCLUSIVE`:** Show attempts, offer: Add context / Retry / Manual

## 10. Spawn gsd-plan-checker Agent

Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFYING PLANS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning plan checker...
```

```bash
PLANS_CONTENT=$(cat "${PHASE_DIR}"/*-PLAN.md 2>/dev/null)
```

Checker prompt:

```markdown
<verification_context>
**Phase:** {phase_number}
**Phase Goal:** {goal from ROADMAP}

**Plans to verify:** {plans_content}
**Phase requirement IDs (MUST ALL be covered):** {phase_req_ids}
**Requirements:** {requirements_content}

**Phase Context:**
IMPORTANT: Plans MUST honor user decisions. Flag as issue if plans contradict.
- **Decisions** = LOCKED — plans must implement exactly
- **Claude's Discretion** = Freedom areas — plans can choose approach
- **Deferred Ideas** = Out of scope — plans must NOT include

{context_content}
</verification_context>

<expected_output>
- ## VERIFICATION PASSED — all checks pass
- ## ISSUES FOUND — structured issue list
</expected_output>
```

```
Task(
  prompt=checker_prompt,
  subagent_type="gsd-plan-checker",
  model="{checker_model}",
  description="Verify Phase {phase} plans"
)
```

## 11. Handle Checker Return

- **`## VERIFICATION PASSED`:** Display confirmation, proceed to step 13.
- **`## ISSUES FOUND`:** Display issues, check iteration count, proceed to step 12.

## 12. Revision Loop (Max 3 Iterations)

Track `iteration_count` (starts at 1 after initial plan + check).

**If iteration_count < 3:**

Display: `Sending back to planner for revision... (iteration {N}/3)`

```bash
PLANS_CONTENT=$(cat "${PHASE_DIR}"/*-PLAN.md 2>/dev/null)
```

Revision prompt:

```markdown
<revision_context>
**Phase:** {phase_number}
**Mode:** revision

**Existing plans:** {plans_content}
**Checker issues:** {structured_issues_from_checker}

**Phase Context:**
Revisions MUST still honor user decisions.
{context_content}
</revision_context>

<instructions>
Make targeted updates to address checker issues.
Do NOT replan from scratch unless issues are fundamental.
Return what changed.
</instructions>
```

```
Task(
  prompt="First, read /Users/thelorax/.claude/agents/gsd-planner.md for your role and instructions.\n\n" + revision_prompt,
  subagent_type="general-purpose",
  model="{planner_model}",
  description="Revise Phase {phase} plans"
)
```

After planner returns -> spawn checker again (step 10), increment iteration_count.

**If iteration_count >= 3:**

Display: `Max iterations reached. {N} issues remain:` + issue list

Offer: 1) Force proceed, 2) Provide guidance and retry, 3) Abandon

## 13. Present Final Status

Route to `<offer_next>` OR `auto_advance` depending on flags/config.

## 14. Auto-Advance Check

Check for auto-advance trigger:

1. Parse `--auto` flag from $ARGUMENTS
2. Read `workflow.auto_advance` from config:
   ```bash
   AUTO_CFG=$(node /Users/thelorax/.claude/get-shit-done/bin/gsd-tools.cjs config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**If `--auto` flag present OR `AUTO_CFG` is true:**

Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTO-ADVANCING TO EXECUTE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Plans ready. Spawning execute-phase...
```

Spawn execute-phase as Task:
```
Task(
  prompt="Run /gsd:execute-phase ${PHASE} --auto",
  subagent_type="general-purpose",
  description="Execute Phase ${PHASE}"
)
```

**Handle execute-phase return:**
- **PHASE COMPLETE** → Display final summary:
  ```
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   GSD ► PHASE ${PHASE} COMPLETE ✓
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Auto-advance pipeline finished.

  Next: /gsd:discuss-phase ${NEXT_PHASE} --auto
  ```
- **GAPS FOUND / VERIFICATION FAILED** → Display result, stop chain:
  ```
  Auto-advance stopped: Execution needs review.

  Review the output above and continue manually:
  /gsd:execute-phase ${PHASE}
  ```

**If neither `--auto` nor config enabled:**
Route to `<offer_next>` (existing behavior).

</process>

<offer_next>
Output this markdown directly (not as a code block):

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PHASE {X} PLANNED ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {X}: {Name}** — {N} plan(s) in {M} wave(s)

| Wave | Plans | What it builds |
|------|-------|----------------|
| 1    | 01, 02 | [objectives] |
| 2    | 03     | [objective]  |

Research: {Completed | Used existing | Skipped}
Verification: {Passed | Passed with override | Skipped}

───────────────────────────────────────────────────────────────

## ▶ Next Up

**Execute Phase {X}** — run all {N} plans

/gsd:execute-phase {X}

<sub>/clear first → fresh context window</sub>

───────────────────────────────────────────────────────────────

**Also available:**
- cat .planning/phases/{phase-dir}/*-PLAN.md — review plans
- /gsd:plan-phase {X} --research — re-research first

───────────────────────────────────────────────────────────────
</offer_next>

<success_criteria>
- [ ] .planning/ directory validated
- [ ] Phase validated against roadmap
- [ ] Phase directory created if needed
- [ ] CONTEXT.md loaded early (step 4) and passed to ALL agents
- [ ] Research completed (unless --skip-research or --gaps or exists)
- [ ] gsd-phase-researcher spawned with CONTEXT.md
- [ ] Existing plans checked
- [ ] gsd-planner spawned with CONTEXT.md + RESEARCH.md
- [ ] Plans created (PLANNING COMPLETE or CHECKPOINT handled)
- [ ] gsd-plan-checker spawned with CONTEXT.md
- [ ] Verification passed OR user override OR max iterations with user decision
- [ ] User sees status between agent spawns
- [ ] User knows next steps
</success_criteria>
