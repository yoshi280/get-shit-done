---
phase: 2-enhance-model-balancing-to-include-opena
plan: 2
type: execute
wave: 1
depends_on: []
files_modified:
  - .config/opencode/get-shit-done/references/model-profiles.md
  - .config/opencode/get-shit-done/references/model-routing-logic.md
  - .config/opencode/get-shit-done/references/model-profile-resolution.md
autonomous: true

must_haves:
  truths:
    - "OpenAI Codex can be used for code generation tasks"
    - "Gemini can be used for analysis and reasoning tasks"
    - "Model routing intelligently selects optimal model based on task characteristics"
  artifacts:
    - path: ".config/opencode/get-shit-done/references/model-profiles.md"
      provides: "Extended model profiles including OpenAI and Gemini options"
      contains: "openai-codex|gemini"
    - path: ".config/opencode/get-shit-done/references/model-routing-logic.md"
      provides: "Intelligent routing logic based on task type"
      min_lines: 40
  key_links:
    - from: "model-profile-resolution.md"
      to: "model-routing-logic.md"
      via: "task type detection"
      pattern: "task_type.*routing"
---

<objective>
Enhance the existing model balancing system to include OpenAI Codex and Gemini with intelligent routing based on task type and performance characteristics.

Purpose: Enable more cost-effective and performance-optimized model selection by leveraging different models' strengths
Output: Extended model profiles and routing logic supporting multi-provider model selection
</objective>

<execution_context>
@/Users/thelorax/.config/Claude/get-shit-done/workflows/execute-plan.md
@/Users/thelorax/.config/Claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.config/opencode/get-shit-done/references/model-profiles.md
@.config/opencode/get-shit-done/references/model-profile-resolution.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Extend model profiles to include OpenAI Codex and Gemini</name>
  <files>.config/opencode/get-shit-done/references/model-profiles.md</files>
  <action>
    Extend the existing model profiles table to include new columns for OpenAI Codex and Gemini models:
    
    - Add `openai-codex` column with appropriate model assignments for code-focused tasks
    - Add `gemini-pro` column for analysis and reasoning tasks
    - Update agent assignments considering:
      - gsd-executor: Use codex for code generation, claude for complex reasoning
      - gsd-codebase-mapper: Use gemini for analysis, claude for structured output
      - gsd-debugger: Use codex for code inspection, claude for problem solving
    
    Add new profile philosophy section explaining when to use each model provider:
    - OpenAI Codex: Code generation, refactoring, syntax-heavy tasks
    - Gemini: Document analysis, large context reasoning, cost-sensitive operations
    - Claude: Complex reasoning, planning, decision making
  </action>
  <verify>grep -E "(openai-codex|gemini)" .config/opencode/get-shit-done/references/model-profiles.md</verify>
  <done>Model profiles table includes OpenAI Codex and Gemini columns with appropriate agent assignments</done>
</task>

<task type="auto">
  <name>Task 2: Create intelligent model routing logic</name>
  <files>.config/opencode/get-shit-done/references/model-routing-logic.md</files>
  <action>
    Create a new file that defines intelligent routing logic based on task characteristics:
    
    1. Task Type Detection:
       - Code generation: file extensions (.js, .ts, .py), keywords (implement, create, build)
       - Analysis: keywords (analyze, review, examine, assess)
       - Planning: keywords (plan, design, architecture, strategy)
       - Debugging: keywords (fix, debug, error, issue)
    
    2. Routing Rules:
       - Code generation tasks → prefer OpenAI Codex for cost/speed
       - Large document analysis → prefer Gemini for context handling
       - Complex reasoning/planning → prefer Claude for quality
       - Budget profile → route more aggressively to cheaper alternatives
    
    3. Fallback Logic:
       - Primary model unavailable → fallback to Claude
       - Rate limiting → automatic retry with alternative model
       - Error handling → graceful degradation
    
    4. Performance Characteristics:
       - Include model comparison matrix (speed, cost, context, quality)
       - Define thresholds for automatic routing decisions
  </action>
  <verify>test -f .config/opencode/get-shit-done/references/model-routing-logic.md && wc -l .config/opencode/get-shit-done/references/model-routing-logic.md</verify>
  <done>Intelligent routing logic documented with task detection rules and fallback strategies</done>
</task>

<task type="auto">
  <name>Task 3: Update model resolution to support intelligent routing</name>
  <files>.config/opencode/get-shit-done/references/model-profile-resolution.md</files>
  <action>
    Update the existing model profile resolution to integrate with intelligent routing:
    
    1. Extend the resolution pattern to include task context:
       - Add task_type parameter extraction from agent prompts
       - Include routing logic reference
    
    2. Update the lookup process:
       - First check if intelligent routing applies
       - If routing enabled, use routing logic to select optimal model
       - Otherwise, fall back to profile-based selection
    
    3. Add configuration options:
       - routing_enabled flag in config.json
       - routing_aggressiveness level (conservative, balanced, aggressive)
       - per-agent routing overrides
    
    4. Document the new resolution flow:
       - Parse task context → Determine task type → Apply routing rules → Select model
       - Include examples of routing decisions for different scenarios
  </action>
  <verify>grep -E "(routing|task_type)" .config/opencode/get-shit-done/references/model-profile-resolution.md</verify>
  <done>Model resolution process enhanced to support intelligent routing with task-aware model selection</done>
</task>

</tasks>

<verification>
1. Model profiles include OpenAI Codex and Gemini options
2. Intelligent routing logic defined with clear task detection rules
3. Model resolution process updated to support routing
4. All files are properly formatted and documented
</verification>

<success_criteria>
- OpenAI Codex and Gemini integrated into model balancing system
- Intelligent routing selects optimal models based on task characteristics
- Backward compatibility maintained with existing profile system
- Clear documentation for configuration and usage
</success_criteria>

<output>
After completion, create `.planning/quick/2-enhance-model-balancing-to-include-opena/2-SUMMARY.md`
</output>