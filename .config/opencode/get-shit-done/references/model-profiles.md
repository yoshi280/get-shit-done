# Model Profiles

Model profiles control which Claude model each GSD agent uses. This allows balancing quality vs token spend.

## Profile Definitions

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

## Profile Philosophy

**quality** - Maximum reasoning power
- Opus for all decision-making agents
- Sonnet for read-only verification
- Use when: quota available, critical architecture work

**balanced** (default) - Smart allocation
- Opus only for planning (where architecture decisions happen)
- Sonnet for execution and research (follows explicit instructions)
- Sonnet for verification (needs reasoning, not just pattern matching)
- Use when: normal development, good balance of quality and cost

**budget** - Minimal Opus usage
- Sonnet for anything that writes code
- Haiku for research and verification
- Use when: conserving quota, high-volume work, less critical phases

**openai-codex** - Code-focused optimization
- Codex for code generation, debugging, and implementation tasks
- Claude (sonnet/opus) for complex reasoning and planning
- Haiku for simple read-only tasks
- Use when: code-heavy phases, cost optimization for development tasks

**gemini-pro** - Analysis and reasoning optimization
- Gemini for document analysis, research, and large context tasks
- Sonnet for code implementation and verification
- Opus only for critical planning decisions
- Use when: research-heavy phases, document processing, cost-sensitive operations

## Resolution Logic

Orchestrators resolve model before spawning:

```
1. Read .planning/config.json
2. Get model_profile (default: "balanced")
3. Look up agent in table above
4. Pass model parameter to Task call
```

## Switching Profiles

Runtime: `/gsd-set-profile <profile>`

Per-project default: Set in `.planning/config.json`:
```json
{
  "model_profile": "balanced"
}
```

## Design Rationale

**Why Opus for gsd-planner?**
Planning involves architecture decisions, goal decomposition, and task design. This is where model quality has the highest impact.

**Why Sonnet for gsd-executor?**
Executors follow explicit PLAN.md instructions. The plan already contains the reasoning; execution is implementation.

**Why Sonnet (not Haiku) for verifiers in balanced?**
Verification requires goal-backward reasoning - checking if code *delivers* what the phase promised, not just pattern matching. Sonnet handles this well; Haiku may miss subtle gaps.

**Why Haiku for gsd-codebase-mapper?**
Read-only exploration and pattern extraction. No reasoning required, just structured output from file contents.
