---
name: gsd:analyze
description: Run specialist analysis on a phase for domain-specific recommendations
argument-hint: "<phase> [specialist-name] [--all]"
allowed-tools:
  - Read
  - Bash
  - Task
  - AskUserQuestion
---
<objective>
Run specialist analysis on a phase. Specialists analyze phase context through domain-specific lenses (e.g., data structures, performance) and produce actionable recommendations.

**Specialists are ADVISORY** — their output feeds into the planner as input, not as binding constraints.

**Modes:**
- Interactive: select specialists from catalog
- Named: run a specific specialist by name
- `--all`: run all specialists in catalog
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/analyze.md
@~/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
Phase number: $ARGUMENTS (required — which phase to analyze)

**Flags:**
- `[specialist-name]` — Run a specific specialist by name
- `--all` — Run all available specialists
</context>

<process>
Execute the analyze workflow from @~/.claude/get-shit-done/workflows/analyze.md end-to-end.
Preserve all workflow gates (catalog loading, selection, agent spawning, urgent findings handling).
</process>
