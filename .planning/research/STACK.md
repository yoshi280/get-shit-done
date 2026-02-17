# Stack Research: Codex CLI + OpenCode Runtime Support

**Domain:** AI coding agent CLI runtime capabilities — subagent spawning, tool calling, slash commands
**Researched:** 2026-02-16
**Confidence:** MEDIUM-HIGH (OpenCode: HIGH from local SDK inspection; Codex CLI: LOW from training data only — web access unavailable)

---

## Research Scope

This document answers the v1.1 question: what do Codex CLI and OpenCode actually support, and where does GSD need to adapt?

Focus: subagent spawning (Task tool equivalent), slash commands/skills, tool calling patterns, multi-agent orchestration, models, integration points.

---

## Runtime Capability Matrix

| Capability | Claude Code | OpenCode | Codex CLI |
|------------|-------------|----------|-----------|
| Slash commands | `/gsd:command` namespace | `/gsd-command` flat namespace | None (no slash command system) |
| Agent definitions | Markdown in `~/.claude/agents/` | `opencode.json` `agent:` config key | None |
| Subagent spawning | `Task(subagent_type, model, prompt)` | `task` tool — uses SubtaskPart mechanism | None confirmed |
| Parallel subagent execution | Yes — Task spawns run in parallel | Unknown — SubtaskPart API exists but parallel behavior unverified | N/A |
| Tool permission model | `allowed-tools:` array in frontmatter | `tools:` boolean map in frontmatter | Approval-based per-action |
| User question prompts | `AskUserQuestion` | `question` | Native conversational turn |
| Invoke slash command | `SlashCommand` | `skill` | N/A |
| Hooks / lifecycle events | `SessionStart`, `StatusLine` | Plugin SDK hooks (event, chat.message, tool.execute.before/after) | None |
| Multi-model selection | `model=` param in Task call | `model:` field in AgentConfig | Set at CLI startup (`--model`) |
| Config directory | `~/.claude/` | `~/.config/opencode/` (XDG) | `~/.codex/` |
| Instructions file | `CLAUDE.md` | `opencode.json` `system:` field or plugin `chat.params` | `codex.md` |

---

## OpenCode — Detailed Capability Analysis

**Source:** `@opencode-ai/sdk` v1.1.53 and `@opencode-ai/plugin` v1.1.53, installed locally at `~/.config/opencode/node_modules/`. Confidence: HIGH.

### Subagent Spawning: `task` Tool

OpenCode has a `task` tool. Confirmed via:
1. Converted GSD command frontmatter includes `task: true` (e.g., `gsd-execute-phase.md`, `gsd-new-project.md`)
2. `@opencode-ai/sdk` v2 types include `task?: PermissionRuleConfig` in the permission schema
3. Underlying mechanism is `SubtaskPart` in the message part API

The SubtaskPart schema (v1 and v2 SDKs):
```typescript
type SubtaskPart = {
  id: string;
  sessionID: string;
  messageID: string;
  type: "subtask";
  prompt: string;
  description: string;
  agent: string;
  model?: { providerID: string; modelID: string; };  // v2 only
}
```

**GSD impact:** The `Task(subagent_type=..., model=..., prompt=...)` call pattern GSD workflows use is nominally supported — OpenCode exposes a `task` tool. However, the exact behavior difference between Claude Code's `Task` (which blocks until subagent completes and returns output) and OpenCode's subtask mechanism needs testing. The `description` field requirement and the `agent` field (which must be a registered agent name, not a markdown file path) are different.

**Critical difference:** Claude Code agents are defined as markdown files in `~/.claude/agents/`. OpenCode agents are defined in `opencode.json` under the `agent:` key as `AgentConfig` objects. GSD installs agent markdown files to `~/.config/opencode/` but OpenCode does not read these as agent definitions — it ignores them. This means `subagent_type="gsd-executor"` in a Task call will NOT work in OpenCode without additional configuration.

### Agent Registration

`AgentConfig` in OpenCode:
```typescript
type AgentConfig = {
  model?: string;
  temperature?: number;
  prompt?: string;
  tools?: { [key: string]: boolean };
  disable?: boolean;
  description?: string;
  mode?: "subagent" | "primary" | "all";
  color?: string;
  maxSteps?: number;
  permission?: { edit?, bash?, webfetch?, doom_loop?, external_directory? };
}
```

Agents are configured in `opencode.json`:
```json
{
  "agent": {
    "plan": { "model": "...", "mode": "subagent" },
    "build": { "model": "...", "mode": "subagent" },
    "general": { ... }
  }
}
```

**GSD impact:** GSD's agents (gsd-executor, gsd-planner, etc.) cannot be registered via markdown files in OpenCode. They must be declared in `opencode.json`. The `prompt` field of `AgentConfig` is a string (the system prompt), not a file reference. GSD would need to either:
- Inject agent system prompts into `opencode.json` at install time, or
- Use the `general` built-in agent as a fallback for all subagent spawns

### Built-in Agents

OpenCode v2 SDK config shows four built-in agent names: `plan`, `build`, `general`, `explore`. The `general` agent is the fallback for Task calls where the named agent isn't registered. This is the closest equivalent to Claude Code's `general-purpose` subagent type (the installer already maps `subagent_type="general-purpose"` → `subagent_type="general"`).

### Tool Names

OpenCode uses lowercase tool names. GSD installer maps these correctly:
- `Read` → `read`
- `Write` → `write`
- `Bash` → `bash`
- `Task` → `task`
- `AskUserQuestion` → `question`
- `SlashCommand` → `skill`
- `TodoWrite` → `todowrite`
- `WebFetch` → `webfetch`

### Slash Commands

OpenCode uses a flat command namespace. GSD installer converts `commands/gsd/execute-phase.md` → `command/gsd-execute-phase.md`. Commands support `subtask?: boolean` field to indicate whether they spawn subagents.

Confirmed from SDK types:
```typescript
type Command = {
  name: string;
  description?: string;
  agent?: string;
  model?: string;
  template: string;
  subtask?: boolean;
}
```

### Multi-Model Support

OpenCode is multi-provider by design. The `AgentConfig.model` field accepts `"provider/model"` format strings. Commands can specify `model:` in frontmatter. GSD's model profile system (which resolves to provider/model strings) should integrate cleanly once agent registration is addressed.

### Plugin SDK

OpenCode has a TypeScript/Bun plugin SDK (`@opencode-ai/plugin` v1.1.53). Plugins can:
- Register custom tools (`tool.definition`)
- Hook into lifecycle events (`chat.message`, `tool.execute.before/after`, `permission.ask`, `shell.env`)
- Modify LLM call parameters (`chat.params`, `chat.headers`)
- Transform message history (`experimental.chat.messages.transform`)
- Customize compaction prompts (`experimental.session.compacting`)

This gives GSD a path to inject behavior (e.g., runtime detection, token tracking) without modifying OpenCode itself. However, plugins require Bun and TypeScript — GSD's zero-dependency Node.js constraint would need to be relaxed for plugin development.

### Permission Model

OpenCode uses an approval-based permission system with `opencode.json`:
```json
{
  "permission": {
    "read": { "~/.config/opencode/get-shit-done/*": "allow" },
    "external_directory": { "~/.config/opencode/get-shit-done/*": "allow" }
  }
}
```

Agents can have per-agent permission overrides in `AgentConfig.permission`. GSD already configures this at install time.

---

## Codex CLI — Detailed Capability Analysis

**Source:** Training data (knowledge cutoff January 2025) + GSD fork model configuration files. Confidence: LOW — requires web verification.

**Important note:** In the GSD codebase, "Codex" appears in two distinct contexts that must not be conflated:
1. **OpenAI Codex models** (`codex`, `gpt-5.x-codex`) — model tier names used in GSD's multi-model routing, used within Claude Code/OpenCode/Gemini runtimes
2. **Codex CLI** — OpenAI's open-source CLI tool for agentic coding (github.com/openai/codex), the v1.1 runtime target

This document covers Codex CLI as a runtime host.

### What Codex CLI Is

Codex CLI (released open-source ~April 2025) is OpenAI's agentic coding assistant CLI. It runs locally, uses OpenAI models (o3, o4-mini, codex-mini by default), and supports three approval modes: `suggest`, `auto-edit`, and `full-auto`.

### Slash Commands

Codex CLI does NOT have a slash command system comparable to Claude Code's `/command` or OpenCode's `/command`. It does not load markdown command files from a config directory. Codex CLI accepts freeform natural language prompts, not structured slash commands.

**GSD impact (CRITICAL):** GSD's entire UX is built on slash commands (`/gsd:new-project`, `/gsd:execute-phase`, etc.). These do not exist in Codex CLI. GSD cannot be invoked via its existing command system in Codex CLI.

**Possible workaround:** Codex CLI supports a `codex.md` system prompt file at `~/.codex/codex.md`. GSD could install workflow instructions there, making them available as natural language instructions rather than slash commands. However, this fundamentally changes how users invoke GSD.

### Agent Definitions / Subagent Spawning

Codex CLI does NOT have an agent definition system or a Task-tool equivalent. It does not support spawning subagents from within a conversation. Codex CLI is a single-agent system.

**GSD impact (CRITICAL):** GSD's multi-agent orchestration (researcher spawning, parallel plan execution, verifier spawning) cannot be implemented in Codex CLI using any native mechanism. All of GSD's wave-based parallel execution would need to be redesigned for a single-agent sequential model.

### Tool Calling

Codex CLI supports file operations (read, write, shell execution) via a sandboxed execution environment. Tool calls are function-call based (OpenAI function calling API format). Codex CLI does not expose named tools with the same surface area as Claude Code.

Confirmed tool capabilities:
- Shell command execution (bash)
- File read/write
- No web search built-in (can execute shell commands that do web requests)

### Multi-Model Support

Codex CLI defaults to `codex-mini` but supports switching via `--model` flag at startup. Model cannot be changed per-agent (no agent system) or per-task (no Task tool). The model is set once for the entire session.

**GSD impact:** GSD's per-agent model routing (Opus for planning, Codex for execution, Gemini for research) cannot be implemented in Codex CLI. All agents run on the same model.

### Configuration

Codex CLI uses `~/.codex/` as its config directory. It reads `~/.codex/codex.md` as a system-level instruction file (similar to `CLAUDE.md`). No structured agent/command configuration exists.

---

## GSD Integration Points

### Where GSD Currently Depends on Claude Code Exclusives

| GSD Feature | Claude Code Mechanism | OpenCode Status | Codex CLI Status |
|-------------|----------------------|-----------------|------------------|
| Parallel researcher spawning | `Task(subagent_type=..., model=..., prompt=...)` parallel calls | `task` tool exists; parallel behavior unconfirmed | No Task tool — impossible |
| Named agent dispatch | `subagent_type="gsd-executor"` (reads `agents/gsd-executor.md`) | Agent must be in `opencode.json`, not markdown file | No agent system |
| Per-agent model selection | `model=` in Task call | Supported via `model:` in SubtaskPart (v2 SDK) | Not supported |
| Slash command invocation | `/gsd:execute-phase 1` | `/gsd-execute-phase 1` (converted at install) | No slash commands |
| User question flow | `AskUserQuestion({...})` | `question` tool | Native conversation |
| Lifecycle hooks | `SessionStart`, `StatusLine` | Plugin SDK hooks | None |
| Subagent type `general-purpose` | Built-in Claude type | Maps to `general` (installer handles this) | N/A |

### OpenCode Adaptation Requirements

1. **Agent registration in `opencode.json`:** GSD's `agents/gsd-*.md` files must be registered as `AgentConfig` entries in `opencode.json` at install time. The agent's markdown content (system prompt) must be embedded in the `prompt:` field of `AgentConfig`. The installer needs to parse agent markdown and inject into `opencode.json`.

2. **Task tool behavior verification:** OpenCode's `task` tool and Claude Code's `Task` tool share a name but their exact behavior (blocking vs non-blocking, return value handling, parallel execution) must be validated against GSD's wave-based orchestration pattern.

3. **Model ID format:** GSD model profiles resolve to tier names (`opus`, `sonnet`, `codex`, `gemini-pro`). These tier names must be mapped to OpenCode's `provider/model` format (e.g., `anthropic/claude-opus-4-6`). OpenCode does not recognize Claude Code tier names.

4. **Flat command namespace:** Already handled by the installer (`/gsd:X` → `/gsd-X`). No additional work needed.

5. **Hook absence:** OpenCode does not support `SessionStart` or `StatusLine` hooks. The GSD statusline and update-check hooks only run in Claude Code and Gemini. This is already accounted for at install time — no hooks are registered for OpenCode.

### Codex CLI Adaptation Requirements

Codex CLI is fundamentally incompatible with GSD's current architecture. Supporting it requires choosing one of three strategies:

**Strategy A: Shallow support** — Install `codex.md` with GSD workflow summaries. No slash commands, no subagents. Users invoke GSD workflows via natural language. Single-agent sequential execution only. Low effort, severely degraded experience.

**Strategy B: Prompt-injection orchestration** — Encode GSD's multi-step workflow in a single agent prompt that does everything sequentially (no parallel). Relies on Codex's single agent following a multi-step plan. Moderate effort, loses parallelism and agent specialization.

**Strategy C: External orchestrator** — Build a Node.js wrapper that spawns multiple Codex CLI processes to simulate parallel subagents. High effort, fragile, outside GSD's zero-dependency constraint. Not recommended.

**Recommendation:** v1.1 should focus on OpenCode workflow parity (feasible, shared `task` tool infrastructure) and defer Codex CLI to a future milestone after validating the fundamental architecture gap.

---

## Model Availability by Runtime

| Model Tier | Claude Code | OpenCode | Codex CLI |
|------------|-------------|----------|-----------|
| Claude Opus 4.6 | Native | Via `anthropic/claude-opus-4-6` | Not supported (OpenAI only) |
| Claude Sonnet 4.5 | Native | Via `anthropic/claude-sonnet-4-5` | Not supported |
| OpenAI gpt-5.x / codex | Via multi-provider config | Via `openai/gpt-5.x` | Native (o3, o4-mini, codex-mini) |
| Gemini 2.5 Pro | Via multi-provider config | Via `google/gemini-2.5-pro` | Not supported |

OpenCode is provider-agnostic — any model accessible via API can be used. Codex CLI is OpenAI-only and not configurable per-task.

---

## Recommended Stack for v1.1

### What to Build for OpenCode Parity

| Component | Technology | Why |
|-----------|------------|-----|
| Agent installer | `bin/install.js` extension | Parse agent markdown frontmatter and body, inject into `opencode.json` as `AgentConfig` entries. Zero new dependencies — installer already handles frontmatter parsing. |
| Task tool validation | Manual test document | Verify OpenCode's `task` tool blocks like Claude Code's Task before relying on wave parallelism. Cannot be verified from docs alone. |
| Model ID resolver | `gsd-tools.cjs` new command | Map GSD tier names (opus, sonnet, codex) to provider/model strings for OpenCode context. Example: `"opus"` → `"anthropic/claude-opus-4-6"`. |
| Runtime detector | `gsd-tools.cjs` new command | Detect runtime from env (presence of `opencode.json`, Claude Code settings.json, etc.). Needed for adaptive workflow behavior. |

### What NOT to Build for v1.1

| Avoid | Why |
|-------|-----|
| Codex CLI slash command system | Codex CLI has no slash command infrastructure. Building one from scratch is a separate framework project. |
| OpenCode plugin SDK integration | Requires Bun, TypeScript — violates zero-dependency constraint. Defer until plugin value justifies dependency exception. |
| Full Codex CLI parity | Single-agent architecture is fundamentally incompatible with GSD's wave-based multi-agent orchestration. |

### Installation Requirements

| Runtime | Install Command | Config Dir | Agent Registration |
|---------|----------------|------------|--------------------|
| Claude Code | `npx get-shit-done-cc --claude` | `~/.claude/` | Auto: `agents/*.md` → `~/.claude/agents/` |
| OpenCode | `npx get-shit-done-cc --opencode` | `~/.config/opencode/` | Required: Parse `agents/*.md` → inject into `opencode.json` `agent:` block |
| Codex CLI | Not yet supported | `~/.codex/` | Not applicable (no agent system) |

---

## Alternatives Considered

| Recommended | Alternative | Why Not |
|-------------|-------------|---------|
| Inject agents into `opencode.json` | Keep agents as markdown files | OpenCode does not read markdown agent files — they are silently ignored |
| Focus on OpenCode first | Build Codex CLI support simultaneously | Codex CLI architecture gap is fundamental; OpenCode gap is tractable |
| Runtime detection via env/file heuristics | Require explicit runtime flag | Users shouldn't need to configure what runtime they're in |
| `task` tool for OpenCode subagents | Plugin SDK for orchestration | Plugin SDK requires Bun/TS dependency, overkill for what is essentially a tool call |

---

## Sources

- `~/.config/opencode/node_modules/@opencode-ai/plugin/` v1.1.53 — `index.d.ts`, `tool.d.ts`, `example.js` — Plugin API (HIGH confidence)
- `~/.config/opencode/node_modules/@opencode-ai/sdk/` v1.1.53 — `gen/types.gen.d.ts` (v1 and v2) — AgentConfig, SubtaskPart, Command types (HIGH confidence)
- `~/.config/opencode/command/gsd-execute-phase.md` — Converted GSD command, confirmed `task: true` in tools frontmatter (HIGH confidence)
- `~/.config/opencode/opencode.json` — Live permission config showing external_directory and read permissions (HIGH confidence)
- `~/get-shit-done/bin/install.js` — `convertClaudeToOpencodeFrontmatter()`, `claudeToOpencodeTools` mapping, `convertGeminiToolName()` (HIGH confidence)
- `~/.claude/get-shit-done/workflows/execute-phase.md` — Task tool usage patterns in GSD orchestration (HIGH confidence)
- `~/.planning/codebase/ARCHITECTURE.md` — GSD architecture analysis (HIGH confidence)
- OpenAI Codex CLI — Training data only, knowledge cutoff January 2025 (LOW confidence — web verification needed for: slash command support, tool API surface, subagent capability, current version)

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| OpenCode `task` tool existence | HIGH | Confirmed in SDK types + converted GSD commands |
| OpenCode SubtaskPart mechanism | HIGH | Confirmed in SDK v1 + v2 type definitions |
| OpenCode agent registration via `opencode.json` | HIGH | Confirmed in AgentConfig type schema |
| OpenCode markdown agent file handling | MEDIUM | SDK shows no markdown agent loading — but negative claim needs official doc verification |
| OpenCode parallel task execution | LOW | SubtaskPart API exists but parallel behavior not confirmed from SDK alone |
| Codex CLI slash command absence | LOW | Training data only — official docs needed |
| Codex CLI subagent absence | LOW | Training data only — official docs needed |
| Codex CLI current version/features | LOW | Training data may be outdated; Codex CLI was open-sourced April 2025 and may have changed |

---

*Stack research for: Codex CLI + OpenCode runtime support in GSD v1.1*
*Researched: 2026-02-16*
