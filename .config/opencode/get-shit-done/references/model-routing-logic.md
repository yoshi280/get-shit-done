# Model Routing Logic

Intelligent model selection based on task characteristics, optimizing for cost, performance, and quality.

## Task Type Detection

### Code Generation Tasks
**Indicators:** File extensions, implementation keywords, structural work

- **File Extensions:** `.js`, `.ts`, `.py`, `.go`, `.rs`, `.java`, `.cpp`, `.c`, `.php`, `.rb`, `.swift`, `.kt`
- **Keywords:** implement, create, build, generate, write, code, function, class, module, component, api, endpoint
- **Context:** Creating new functionality, refactoring existing code, implementing specifications

### Analysis Tasks  
**Indicators:** Review keywords, examination work, research activities

- **Keywords:** analyze, review, examine, assess, evaluate, investigate, research, understand, explore, study
- **Context:** Code review, documentation analysis, system architecture review, requirement analysis

### Planning Tasks
**Indicators:** Strategic keywords, design work, architecture decisions

- **Keywords:** plan, design, architecture, strategy, roadmap, organize, structure, outline, blueprint, workflow
- **Context:** System design, project planning, technical decision making, process definition

### Debugging Tasks
**Indicators:** Problem-solving keywords, error investigation, troubleshooting

- **Keywords:** fix, debug, error, issue, problem, bug, troubleshoot, diagnose, resolve, repair
- **Context:** Error investigation, performance issues, system failures, code corrections

## Routing Rules

### Primary Model Selection

| Task Type | Primary Model | Rationale |
|-----------|--------------|-----------|
| Code Generation | OpenAI Codex | Optimized for code synthesis, faster generation, cost-effective |
| Analysis | Gemini Pro | Large context window, strong reasoning, cost-effective for documents |
| Planning | Claude Opus | Superior reasoning quality, architectural decision support |
| Debugging | OpenAI Codex | Code understanding, pattern recognition, debugging experience |

### Context-Based Routing

**Large Context (>50k tokens)**
- Prefer Gemini Pro for analysis tasks
- Use Claude Sonnet for code tasks with large context
- Fallback to chunking strategies if context exceeds model limits

**Budget Profile Considerations**
- Route more aggressively to cheaper alternatives
- Code Generation: Codex → Sonnet → Haiku
- Analysis: Gemini → Sonnet → Haiku
- Planning: Keep Opus for critical decisions, use Sonnet for routine planning

**Quality Profile Considerations**
- Use highest quality models even if more expensive
- Planning: Always use Opus
- Code Generation: Prefer Claude for complex implementations
- Analysis: Use Opus for critical analysis, Gemini for routine reviews

## Fallback Logic

### Primary Model Unavailable
1. **API Errors (500, 503):** Retry with exponential backoff (2s, 4s, 8s)
2. **Authentication Issues:** Alert user, halt execution
3. **Service Unavailable:** Switch to Claude as universal fallback

### Rate Limiting
1. **429 Errors:** Check retry-after header
2. **Quota Exceeded:** Auto-switch to alternative model in same capability tier
3. **Temporary Limits:** Queue request with automatic retry

### Error Handling
```
Codex Error → Claude Sonnet (code tasks)
Gemini Error → Claude Sonnet (analysis tasks)
Claude Error → System halt (requires user intervention)
```

### Graceful Degradation
- If optimal model fails, use next-best alternative
- Log routing decisions for debugging
- Maintain task quality while adapting to constraints

## Performance Characteristics

### Model Comparison Matrix

| Model | Speed | Cost | Context | Quality | Code Focus | Analysis Strength |
|-------|-------|------|---------|---------|------------|------------------|
| OpenAI Codex | ★★★★★ | ★★★★☆ | ★★★☆☆ | ★★★★☆ | ★★★★★ | ★★★☆☆ |
| Gemini Pro | ★★★★☆ | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★☆☆ | ★★★★★ |
| Claude Opus | ★★★☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★★★ |
| Claude Sonnet | ★★★★☆ | ★★★☆☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ |
| Claude Haiku | ★★★★★ | ★★★★★ | ★★★☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★☆☆ |

### Routing Decision Thresholds

**Automatic Routing Triggers:**
- Code file extensions present: +2 points toward Codex
- Analysis keywords: +2 points toward Gemini
- Large context (>20k tokens): +1 point toward Gemini
- Budget profile active: +1 point toward cheaper alternative
- Quality profile active: +1 point toward Claude

**Decision Matrix:**
- 3+ points toward model: Auto-route
- 1-2 points: Consider routing based on secondary factors
- 0 points: Use profile default

### Performance Monitoring
- Track response times by model and task type
- Monitor cost per successful completion
- Measure quality scores (when available)
- Adjust routing rules based on performance data

## Configuration Options

### Routing Settings
```json
{
  "routing_enabled": true,
  "routing_aggressiveness": "balanced",
  "fallback_model": "sonnet",
  "retry_attempts": 3,
  "retry_delay_ms": 2000
}
```

### Aggressiveness Levels
- **conservative:** Only route on clear task type matches
- **balanced:** Route based on thresholds and context
- **aggressive:** Prefer alternative models whenever cost-effective

### Per-Agent Overrides
```json
{
  "agent_overrides": {
    "gsd-executor": {
      "routing_enabled": true,
      "prefer_model": "codex"
    },
    "gsd-codebase-mapper": {
      "routing_enabled": true,
      "prefer_model": "gemini"
    }
  }
}
```

## Implementation Guidelines

### Integration Points
1. **Task Spawn:** Check routing rules before model selection
2. **Prompt Analysis:** Extract task type indicators from prompt content
3. **Context Assessment:** Evaluate token count and content type
4. **Model Resolution:** Apply routing logic before profile lookup

### Monitoring and Debugging
- Log all routing decisions with rationale
- Track model performance metrics
- Alert on frequent fallbacks or errors
- Provide routing decision transparency to users

### Backward Compatibility
- Routing is opt-in via configuration
- Profile-based selection remains default
- No breaking changes to existing workflows
- Gradual migration path for existing projects