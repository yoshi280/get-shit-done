# Model Profiles

Model profiles control which AI model each GSD agent uses across all configured providers (Anthropic, OpenAI, Google). This allows balancing quality, cost, and provider strengths.

## Profile Definitions (Tier Names)

The default profile table returns tier shorthand names. Projects with `model_allocation` in config.json override these with full `provider/model` IDs.

| Agent | `quality` | `balanced` | `budget` | `openai-codex` | `gemini-pro` |
|-------|-----------|------------|----------|---------------|--------------|
| gsd-planner | opus | opus | sonnet | opus | gemini-pro |
| gsd-roadmapper | opus | sonnet | sonnet | sonnet | gemini-pro |
| gsd-executor | opus | sonnet | sonnet | codex | sonnet |
| gsd-phase-researcher | opus | sonnet | haiku | sonnet | gemini-pro |
| gsd-project-researcher | opus | sonnet | haiku | sonnet | gemini-pro |
| gsd-research-synthesizer | sonnet | sonnet | haiku | sonnet | gemini-pro |
| gsd-debugger | opus | sonnet | sonnet | codex | sonnet |
| gsd-codebase-mapper | sonnet | haiku | haiku | haiku | gemini-pro |
| gsd-verifier | sonnet | sonnet | haiku | sonnet | sonnet |
| gsd-plan-checker | sonnet | sonnet | haiku | sonnet | gemini-pro |
| gsd-integration-checker | sonnet | sonnet | haiku | sonnet | sonnet |

## Per-Project Model Allocation (Full Provider/Model IDs)

Projects can override the tier table with exact model IDs via `model_allocation` in `.planning/config.json`. This is the recommended approach for multi-provider setups.

```json
{
  "model_profile": "balanced",
  "model_allocation": {
    "balanced": {
      "gsd-planner":  { "primary": "anthropic/claude-opus-4-6", "fallback": ["openai/gpt-5.2", "google/gemini-2.5-pro"] },
      "gsd-executor": { "primary": "openai/gpt-5.2-codex",     "fallback": ["openai/gpt-5.1-codex", "anthropic/claude-sonnet-4-5"] }
    }
  }
}
```

**Resolution priority:** `model_allocation[profile][agent].primary` → `MODEL_PROFILES[agent][profile]` → `balanced` fallback → `sonnet`

## Available Models by Provider

### Anthropic (Claude)
| Model | Tier | Strengths | Best For |
|-------|------|-----------|----------|
| claude-opus-4-6 | premium | Strongest reasoning, architecture | Planning, debugging, system design |
| claude-opus-4-5 | premium | Deep reasoning | Complex architecture decisions |
| claude-sonnet-4-5 | mid | Strong all-rounder | Execution, verification, synthesis |
| claude-sonnet-4-0 | mid | Reliable general purpose | Standard execution, fallback |
| claude-haiku-4-5 | budget | Fast, cheap | Codebase mapping, simple checks |

### OpenAI
| Model | Tier | Strengths | Best For |
|-------|------|-----------|----------|
| gpt-5.3-codex | premium | Latest code generation | Quality-tier code execution |
| gpt-5.2-codex | mid-high | Strong code generation | Balanced-tier code execution |
| gpt-5.2 | mid-high | General flagship reasoning | Plan checking, verification, reasoning |
| gpt-5.1-codex-max | mid-high | Max-capability code | Complex code tasks |
| gpt-5.1-codex | mid | Standard code generation | Budget-tier code execution |
| gpt-5.1-codex-mini | budget | Fast, cheap code | Codebase mapping, simple code checks |

### Google (Gemini)
| Model | Tier | Strengths | Best For |
|-------|------|-----------|----------|
| gemini-3-pro-preview | premium | Next-gen reasoning | Research, analysis (preview) |
| gemini-2.5-pro | mid-high | Large context, strong analysis | Research, synthesis, document processing |
| gemini-2.5-flash | mid | Fast, large context | Codebase mapping, plan checking, verification |
| gemini-2.5-flash-lite | budget | Cheapest, fast | Bulk exploration, simple research |

## Profile Philosophy

**quality** — Maximum reasoning power, best models everywhere
- Opus for all decision-making agents
- gpt-5.3-codex for code execution
- Gemini Pro for research
- Use when: critical architecture work, quota available

**balanced** (default) — Smart allocation across providers
- Opus for planning only (architecture decisions)
- Codex for execution and debugging (code-optimized)
- Gemini Pro for research (large context)
- Gemini Flash for exploration and checking (cost-effective)
- Use when: normal development, leveraging all 3 providers

**budget** — Minimize cost, maximize throughput
- Sonnet for planning (no Opus)
- gpt-5.1-codex / codex-mini for code tasks
- Gemini Flash / Flash Lite for everything else
- Use when: high-volume work, less critical phases

**openai-codex** — Route code-heavy work to OpenAI
- Codex for code generation, debugging, and implementation
- Claude (sonnet/opus) for complex reasoning and planning
- Haiku for simple read-only tasks
- Use when: code-heavy phases, OpenAI cost advantage

**gemini-pro** — Route analysis-heavy work to Google
- Gemini for document analysis, research, and large context tasks
- Sonnet for code implementation and verification
- Use when: research-heavy phases, document processing

## Resolution Logic

Orchestrators resolve model before spawning:

```
1. Read .planning/config.json
2. Get model_profile (default: "balanced")
3. Check model_allocation[profile][agent].primary (full provider/model ID)
4. If not found, fall back to MODEL_PROFILES[agent][profile] (tier name)
5. Pass resolved model to Task call
```

## Switching Profiles

Runtime: `/gsd-set-profile <profile>`

Valid profiles: `quality`, `balanced`, `budget`, `openai-codex`, `gemini-pro`

Per-project default in `.planning/config.json`:
```json
{
  "model_profile": "balanced"
}
```

## Design Rationale

**Why Opus for gsd-planner?**
Planning involves architecture decisions, goal decomposition, and task design. This is where model quality has the highest impact.

**Why Codex for gsd-executor?**
Executors follow explicit PLAN.md instructions writing code. Code-specialized models outperform general models for implementation tasks.

**Why Gemini for research agents?**
Research involves processing large documents and synthesizing information. Gemini's large context window and cost-effective analysis make it ideal.

**Why Gemini Flash for gsd-codebase-mapper?**
Read-only exploration and pattern extraction. Needs large context window, not deep reasoning. Flash is fast and cheap.

**Why Sonnet (not Haiku) for verifiers in balanced?**
Verification requires goal-backward reasoning — checking if code *delivers* what the phase promised, not just pattern matching. Sonnet handles this well; Haiku may miss subtle gaps.

**Why fallback chains?**
Provider outages and rate limits happen. Each agent has 2-3 fallbacks across providers ensuring workflows never stall on a single provider's availability.
