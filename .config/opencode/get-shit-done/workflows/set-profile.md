<purpose>
Switch the model profile used by GSD agents. Controls which AI model each agent uses across all configured providers (Anthropic, OpenAI, Google), balancing quality vs token spend.
</purpose>

<required_reading>
Read all files referenced by the invoking prompt's execution_context before starting.
</required_reading>

<process>

<step name="validate">
Validate argument:

```
if $ARGUMENTS.profile not in ["quality", "balanced", "budget", "openai-codex", "gemini-pro"]:
  Error: Invalid profile "$ARGUMENTS.profile"
  Valid profiles: quality, balanced, budget, openai-codex, gemini-pro
  EXIT
```
</step>

<step name="ensure_and_load_config">
Ensure config exists and load current state:

```bash
node /Users/thelorax/.config/opencode/get-shit-done/bin/gsd-tools.js config-ensure-section
INIT=$(node /Users/thelorax/.config/opencode/get-shit-done/bin/gsd-tools.js state load)
```

This creates `.planning/config.json` with defaults if missing and loads current config.
</step>

<step name="update_config">
Read current config from state load or directly:

Update `model_profile` field:
```json
{
  "model_profile": "$ARGUMENTS.profile"
}
```

Write updated config back to `.planning/config.json`.
</step>

<step name="confirm">
Display confirmation with model table for selected profile:

```
✓ Model profile set to: $ARGUMENTS.profile

Agents will now use:

[Show table from MODEL_PROFILES in gsd-tools.js for selected profile]

Example:
| Agent | Model |
|-------|-------|
| gsd-planner | opus |
| gsd-executor | sonnet |
| gsd-verifier | haiku |
| ... | ... |

Next spawned agents will use the new profile.
```

Map profile names:
- quality: use "quality" column from MODEL_PROFILES (Opus-heavy, best reasoning)
- balanced: use "balanced" column from MODEL_PROFILES (Opus for planning, Sonnet for execution)
- budget: use "budget" column from MODEL_PROFILES (Sonnet/Haiku, minimal Opus)
- openai-codex: use "openai-codex" column from MODEL_PROFILES (Codex for code gen/debug, Claude for planning)
- gemini-pro: use "gemini-pro" column from MODEL_PROFILES (Gemini for research/analysis, Claude for code)

Note: If `model_allocation` is configured in `.planning/config.json`, it overrides
the profile table with full provider/model IDs (e.g., "openai/gpt-5.2-codex").
</step>

</process>

<success_criteria>
- [ ] Argument validated
- [ ] Config file ensured
- [ ] Config updated with new model_profile
- [ ] Confirmation displayed with model table
</success_criteria>
