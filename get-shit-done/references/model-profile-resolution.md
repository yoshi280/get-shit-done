# Model Profile Resolution

Resolve model profile and apply intelligent routing at the start of orchestration, then use for all Task spawns.

## Enhanced Resolution Pattern

```bash
MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | grep -o '"model_profile"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "balanced")
ROUTING_ENABLED=$(cat .planning/config.json 2>/dev/null | grep -o '"routing_enabled"[[:space:]]*:[[:space:]]*[^,}]*' | grep -o '[^:]*$' | tr -d ' "' || echo "false")
ROUTING_AGGRESSIVENESS=$(cat .planning/config.json 2>/dev/null | grep -o '"routing_aggressiveness"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "balanced")
```

Defaults: `balanced` profile, `false` routing, `balanced` aggressiveness if not set or config missing.

## Enhanced Lookup Process

@/Users/thelorax/.config/opencode/get-shit-done/references/model-profiles.md  
@/Users/thelorax/.config/opencode/get-shit-done/references/model-routing-logic.md

### Step 1: Task Context Analysis
Extract task characteristics from prompt and context:

```bash
# Extract task type indicators
TASK_TYPE=""
if echo "$PROMPT" | grep -E '\.(js|ts|py|go|rs|java|cpp|c|php|rb|swift|kt)' >/dev/null; then
  TASK_TYPE="code_generation"
elif echo "$PROMPT" | grep -E '(analyze|review|examine|assess|evaluate)' >/dev/null; then
  TASK_TYPE="analysis"
elif echo "$PROMPT" | grep -E '(plan|design|architecture|strategy|roadmap)' >/dev/null; then
  TASK_TYPE="planning"
elif echo "$PROMPT" | grep -E '(fix|debug|error|issue|problem|bug)' >/dev/null; then
  TASK_TYPE="debugging"
fi

# Estimate context size
CONTEXT_SIZE=$(echo "$PROMPT" | wc -c)
```

### Step 2: Intelligent Routing Decision
If routing enabled, apply routing logic:

```bash
if [ "$ROUTING_ENABLED" = "true" ]; then
  case "$TASK_TYPE" in
    "code_generation"|"debugging")
      if [ "$MODEL_PROFILE" != "quality" ]; then
        SELECTED_MODEL="codex"
      fi
      ;;
    "analysis")
      if [ "$CONTEXT_SIZE" -gt 20000 ] || [ "$MODEL_PROFILE" = "budget" ]; then
        SELECTED_MODEL="gemini-pro"
      fi
      ;;
    "planning")
      # Keep profile-based selection for planning (Opus preferred)
      ;;
  esac
  
  # Apply aggressiveness factor
  if [ "$ROUTING_AGGRESSIVENESS" = "aggressive" ] && [ -z "$SELECTED_MODEL" ]; then
    case "$TASK_TYPE" in
      "code_generation"|"debugging") SELECTED_MODEL="codex" ;;
      "analysis") SELECTED_MODEL="gemini-pro" ;;
    esac
  fi
fi
```

### Step 3: Profile-Based Fallback
If routing didn't select a model, use profile table:

```bash
if [ -z "$SELECTED_MODEL" ]; then
  # Look up agent in profile table for resolved profile
  SELECTED_MODEL=$(lookup_profile_table "$AGENT_TYPE" "$MODEL_PROFILE")
fi
```

### Step 4: Task Spawn
Pass the resolved model to Task calls:

```
Task(
  prompt="...",
  subagent_type="gsd-planner",
  model="{SELECTED_MODEL}",  # e.g., "codex", "gemini-pro", or "opus"
  routing_context={
    "task_type": "{TASK_TYPE}",
    "context_size": {CONTEXT_SIZE},
    "routing_enabled": {ROUTING_ENABLED}
  }
)
```

## Configuration Options

### Basic Configuration (.planning/config.json)
```json
{
  "model_profile": "balanced",
  "routing_enabled": true,
  "routing_aggressiveness": "balanced",
  "fallback_model": "sonnet"
}
```

### Per-Agent Overrides
```json
{
  "agent_overrides": {
    "gsd-executor": {
      "routing_enabled": true,
      "prefer_model": "codex",
      "force_routing": true
    },
    "gsd-codebase-mapper": {
      "routing_enabled": true,
      "prefer_model": "gemini-pro"
    },
    "gsd-planner": {
      "routing_enabled": false,
      "force_model": "opus"
    }
  }
}
```

## Resolution Flow Examples

### Example 1: Code Generation Task
- **Input:** Prompt contains "implement API endpoint", agent="gsd-executor"
- **Analysis:** task_type="code_generation", context_size=5000
- **Routing:** routing_enabled=true → prefer codex
- **Result:** model="codex" (instead of profile's "sonnet")

### Example 2: Large Document Analysis
- **Input:** Prompt contains "analyze requirements", agent="gsd-research-synthesizer", context_size=60000
- **Analysis:** task_type="analysis", large_context=true
- **Routing:** routing_enabled=true → prefer gemini-pro for large context
- **Result:** model="gemini-pro" (instead of profile's "sonnet")

### Example 3: Planning Task (Quality Profile)
- **Input:** Prompt contains "design architecture", agent="gsd-planner"
- **Analysis:** task_type="planning"
- **Routing:** Planning tasks prefer to keep profile-based selection
- **Result:** model="opus" (from quality profile, routing doesn't override)

### Example 4: Budget Profile with Aggressive Routing
- **Input:** Prompt contains "review code", agent="gsd-verifier", profile="budget"
- **Analysis:** task_type="analysis", aggressiveness="aggressive"
- **Routing:** Aggressive routing + budget profile → route to cheaper alternative
- **Result:** model="gemini-pro" (instead of profile's "haiku")

## Backward Compatibility

- **Default Behavior:** routing_enabled=false maintains current profile-only behavior
- **Gradual Migration:** Projects can enable routing per-agent or globally
- **Profile Preservation:** All existing profiles remain functional
- **No Breaking Changes:** Existing workflows continue without modification

## Usage

1. **Resolve configuration** once at orchestration start
2. **Extract task context** from prompt and metadata  
3. **Apply routing logic** if enabled and applicable
4. **Fallback to profile** lookup if routing doesn't apply
5. **Pass resolved model** to Task spawn with routing context
6. **Log routing decisions** for debugging and optimization
