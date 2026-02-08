# GSD Orchestrator Balancing Rules

Comprehensive multi-model balancing framework providing orchestrators with intelligent model selection, resource preservation, and cost optimization strategies across all GSD workflows.

## Executive Summary

**Value Proposition:** Intelligent multi-model orchestration that automatically selects optimal models based on task characteristics, workflow context, and resource constraints while maintaining quality and minimizing costs.

**Resource Preservation Goals:**
- 60-80% reduction in Opus usage through intelligent task routing
- Automated quota management with graceful degradation patterns
- Dynamic cost optimization without sacrificing critical quality decisions

**Cost Optimization Targets:**
- 40-60% cost reduction for code generation tasks via OpenAI Codex routing
- 30-50% cost reduction for analysis tasks via Gemini Pro preference
- Automated emergency conservation modes preventing quota exhaustion

**Integration Readiness:** Drop-in replacement for existing profile-only systems with backward compatibility and gradual migration support.

## Core Decision Framework

### Workflow-Specific Model Selection Rules

#### Planning Workflows (gsd-planner, gsd-roadmapper)
**Primary Goal:** Preserve architectural decision quality
- **Model Priority:** Opus > Sonnet > Gemini Pro
- **Routing Override:** Disabled for critical planning tasks
- **Quality Threshold:** Always use highest reasoning capability available
- **Cost Consideration:** Accept higher costs for architecture decisions

```
Planning Task → Opus (unless quota critical)
Routine Planning → Sonnet (if budget profile active)
Documentation Planning → Gemini Pro (large context optimization)
```

#### Execution Workflows (gsd-executor, gsd-debugger)
**Primary Goal:** Optimize implementation speed and cost
- **Model Priority:** Codex > Sonnet > Haiku
- **Routing Override:** Enabled by default for code tasks
- **Quality Threshold:** Sufficient for following explicit instructions
- **Cost Consideration:** Maximize cost-effectiveness

```
Code Generation → Codex (file extensions: .js, .ts, .py, etc.)
Bug Fixing → Codex (keywords: fix, debug, error, issue)
Configuration → Sonnet (complex logic without code generation)
Simple Tasks → Haiku (profile fallback only)
```

#### Research Workflows (gsd-phase-researcher, gsd-project-researcher)
**Primary Goal:** Balance context handling with cost efficiency
- **Model Priority:** Gemini Pro > Sonnet > Haiku
- **Routing Override:** Enabled for large context tasks
- **Quality Threshold:** Adequate for information synthesis
- **Cost Consideration:** Optimize for large document processing

```
Large Context (>20k tokens) → Gemini Pro
Document Analysis → Gemini Pro (keywords: analyze, review, examine)
Code Research → Sonnet (mixed code/text analysis)
Quick Lookups → Haiku (simple information retrieval)
```

#### Verification Workflows (gsd-verifier, gsd-integration-checker)
**Primary Goal:** Ensure quality without over-provisioning
- **Model Priority:** Sonnet > Gemini Pro > Haiku
- **Routing Override:** Context-dependent
- **Quality Threshold:** Sufficient for goal-backward reasoning
- **Cost Consideration:** Balanced quality and efficiency

```
Code Verification → Sonnet (reasoning required, not pattern matching)
Documentation Verification → Gemini Pro (large context handling)
Simple Checks → Haiku (budget profile only)
```

### Context-Aware Routing Triggers

#### Task Type Detection Matrix
```
Code Generation:    File extensions (.js, .ts, .py) + Implementation keywords
Analysis Tasks:     Review/examine keywords + Document processing
Planning Tasks:     Design/architecture/strategy keywords
Debugging Tasks:    Fix/debug/error/issue keywords
```

#### Content Size Routing
```
Small Context (<5k tokens):    Use profile default
Medium Context (5k-20k):       Consider routing based on task type
Large Context (>20k tokens):   Prefer Gemini Pro for analysis
Huge Context (>50k tokens):    Force Gemini Pro or chunking strategy
```

#### Profile-Based Fallback Strategies
```
Quality Profile:    Minimal routing, preserve Opus allocation
Balanced Profile:   Smart routing with quality preservation
Budget Profile:     Aggressive routing to cheaper alternatives
Codex Profile:      Force code task routing, Claude for reasoning
Gemini Profile:     Force analysis routing, Sonnet for code tasks
```

## Resource Preservation Rules

### Quota Management Strategies

#### Opus Quota Thresholds
```
Green Zone (>75% remaining):    Normal Opus usage for planning
Yellow Zone (25-75% remaining): Conservative Opus, route analysis to Gemini
Orange Zone (10-25% remaining): Opus only for critical planning decisions
Red Zone (<10% remaining):      Emergency mode, Sonnet for all except architecture
```

#### Dynamic Degradation Patterns
```
Quality → Balanced:     Reduce Opus usage by 50%, maintain planning quality
Balanced → Budget:      Route 80% of tasks to alternatives, preserve core reasoning
Budget → Emergency:     Haiku + Gemini Pro only, queue critical tasks
Emergency → Halt:       Stop non-critical workflows, alert user
```

#### Token Budget Allocation
```
Planning Phase:     40% of monthly quota (architecture decisions critical)
Execution Phase:    30% of monthly quota (optimize via Codex routing)
Research Phase:     20% of monthly quota (optimize via Gemini routing)
Verification Phase: 10% of monthly quota (efficient verification patterns)
```

### Emergency Conservation Modes

#### Automatic Triggers
- Quota usage >90% with >7 days remaining in billing period
- Rate limiting detected on primary models for >1 hour
- Cost velocity exceeding budget by >200% weekly rate
- Multiple model failures requiring expensive fallbacks

#### Conservation Actions
```
Level 1: Route all code tasks to Codex, all analysis to Gemini
Level 2: Downgrade all non-planning tasks to Sonnet/Haiku
Level 3: Queue non-critical tasks, process only architecture decisions
Level 4: User intervention required, halt automated workflows
```

#### Recovery Procedures
```
Monitor quota restoration and model availability
Gradually restore normal routing based on resource recovery
Process queued tasks in priority order (planning > execution > research > verification)
Update conservation thresholds based on usage patterns
```

## Cost Optimization Strategies

### Alternative Model Routing

#### Code Generation Optimization
```
Primary Route:   JavaScript/TypeScript → OpenAI Codex
Secondary Route: Python/Go/Rust → OpenAI Codex
Fallback Route:  Complex Logic → Claude Sonnet
Cost Savings:    60-70% compared to Opus-only approach
```

#### Large Context Handling
```
Primary Route:   Analysis >20k tokens → Gemini Pro
Secondary Route: Mixed content → Gemini Pro with Claude fallback
Fallback Route:  Code-heavy large context → Claude Sonnet
Cost Savings:    40-50% compared to Claude-only approach
```

#### Batch Processing Optimization
```
Research Phases:    Batch document analysis to Gemini Pro
Code Reviews:       Batch file analysis to Codex
Verification:       Batch checks to appropriate specialist models
Cost Savings:       20-30% through reduced context switching
```

### Per-Agent Cost-Effectiveness Guidelines

#### High-Value Opus Usage
- gsd-planner: Architecture and design decisions
- Critical debugging requiring deep reasoning
- Complex requirement analysis and goal decomposition

#### Optimal Sonnet Usage
- gsd-executor: Following explicit implementation plans
- gsd-verifier: Quality verification and gap analysis
- Mixed code/reasoning tasks with moderate complexity

#### Strategic Haiku Usage
- gsd-codebase-mapper: File exploration and pattern extraction
- Simple verification tasks in budget profiles
- Quick information retrieval and formatting

#### External Model Integration
- **OpenAI Codex:** All code generation and debugging tasks
- **Gemini Pro:** Large document analysis and research synthesis
- **Claude Models:** Complex reasoning and fallback scenarios

## Workflow-Specific Rules

### Planning Workflows: Preserve Reasoning Quality

#### Model Selection Strategy
```
Primary:    Opus for all architecture decisions
Secondary:  Sonnet for routine planning tasks
Emergency:  Gemini Pro for documentation-heavy planning
Never:      Haiku for planning (insufficient reasoning capability)
```

#### Quality Preservation Techniques
- Protect Opus allocation for planning phases
- Route non-critical planning subtasks to Sonnet
- Use Gemini Pro for research-heavy planning components
- Queue planning tasks during emergency conservation

#### Cost Optimization Approach
- Minimize planning iterations through better context preparation
- Pre-analyze requirements using cheaper models before Opus planning
- Batch related planning decisions to reduce context switching

### Execution Workflows: Optimize Implementation Speed

#### Model Selection Strategy
```
Code Files Present:     Codex > Sonnet > Haiku
Configuration Tasks:    Sonnet > Codex > Gemini Pro
Infrastructure Setup:   Sonnet > Gemini Pro > Codex
Bug Fixing:            Codex > Sonnet (debugging expertise)
```

#### Speed Optimization Techniques
- Automatic routing to Codex for recognized file extensions
- Parallel task execution with model specialization
- Reduced context preparation for implementation tasks
- Fast fallback chains for model unavailability

#### Cost Optimization Approach
- Aggressive routing to Codex for all code generation
- Use Sonnet only for complex logic requiring reasoning
- Avoid Opus entirely unless plan requires architectural changes
- Batch similar implementation tasks to same model

### Research Workflows: Balance Context with Cost

#### Model Selection Strategy
```
Large Documents (>20k): Gemini Pro > Claude > Codex
Mixed Content:          Gemini Pro > Sonnet > Haiku
Code Analysis:          Sonnet > Gemini Pro > Codex
Quick Research:         Haiku > Sonnet (budget profiles)
```

#### Context Optimization Techniques
- Automatic large context detection and Gemini routing
- Document chunking strategies for oversized content
- Intelligent summarization before detailed analysis
- Parallel research streams for complex topics

#### Cost Optimization Approach
- Prefer Gemini Pro for all document-heavy research
- Use Claude only when reasoning quality is critical
- Implement aggressive caching for repeated research queries
- Batch related research topics to minimize context switching

### Verification Workflows: Ensure Quality Without Over-Provisioning

#### Model Selection Strategy
```
Code Verification:      Sonnet > Gemini Pro > Haiku
Document Verification:  Gemini Pro > Sonnet > Haiku
Logic Verification:     Sonnet > Opus (critical cases only)
Format Verification:    Haiku > Sonnet (simple pattern matching)
```

#### Quality Assurance Techniques
- Use Sonnet for goal-backward reasoning verification
- Avoid Haiku except for simple pattern matching tasks
- Escalate to Opus only for critical verification failures
- Implement verification caching for repeated patterns

#### Cost Optimization Approach
- Minimize verification iterations through better test design
- Use appropriate model capability for verification complexity
- Batch verification tasks when possible
- Implement smart retry logic before escalating models

## Integration Guidelines

### Configuration Templates for Immediate Deployment

#### Default Balanced Configuration
```json
{
  "model_profile": "balanced",
  "routing_enabled": true,
  "routing_aggressiveness": "balanced",
  "fallback_model": "sonnet",
  "quota_management": {
    "opus_threshold_warning": 0.75,
    "opus_threshold_conservation": 0.25,
    "opus_threshold_emergency": 0.10
  },
  "cost_optimization": {
    "code_routing_enabled": true,
    "analysis_routing_enabled": true,
    "batch_processing": true
  }
}
```

#### Budget-Optimized Configuration
```json
{
  "model_profile": "budget",
  "routing_enabled": true,
  "routing_aggressiveness": "aggressive",
  "fallback_model": "haiku",
  "quota_management": {
    "opus_threshold_warning": 0.50,
    "opus_threshold_conservation": 0.15,
    "opus_threshold_emergency": 0.05
  },
  "cost_optimization": {
    "code_routing_enabled": true,
    "analysis_routing_enabled": true,
    "batch_processing": true,
    "force_alternatives": true
  },
  "agent_overrides": {
    "gsd-executor": { "prefer_model": "codex", "force_routing": true },
    "gsd-codebase-mapper": { "prefer_model": "gemini-pro" },
    "gsd-phase-researcher": { "prefer_model": "gemini-pro" }
  }
}
```

#### Quality-Focused Configuration
```json
{
  "model_profile": "quality",
  "routing_enabled": false,
  "routing_aggressiveness": "conservative",
  "fallback_model": "opus",
  "quota_management": {
    "opus_threshold_warning": 0.90,
    "opus_threshold_conservation": 0.50,
    "opus_threshold_emergency": 0.25
  },
  "cost_optimization": {
    "code_routing_enabled": false,
    "analysis_routing_enabled": false,
    "batch_processing": false
  }
}
```

#### Research-Heavy Configuration
```json
{
  "model_profile": "gemini-pro",
  "routing_enabled": true,
  "routing_aggressiveness": "balanced",
  "fallback_model": "sonnet",
  "quota_management": {
    "opus_threshold_warning": 0.80,
    "opus_threshold_conservation": 0.30,
    "opus_threshold_emergency": 0.10
  },
  "cost_optimization": {
    "code_routing_enabled": true,
    "analysis_routing_enabled": true,
    "batch_processing": true,
    "large_context_optimization": true
  },
  "agent_overrides": {
    "gsd-phase-researcher": { "prefer_model": "gemini-pro", "force_routing": true },
    "gsd-project-researcher": { "prefer_model": "gemini-pro", "force_routing": true },
    "gsd-research-synthesizer": { "prefer_model": "gemini-pro" }
  }
}
```

### Gradual Migration Strategies

#### Phase 1: Assessment and Preparation (Week 1)
1. Audit current model usage patterns and costs
2. Identify high-cost workflows suitable for optimization
3. Install monitoring and alerting infrastructure
4. Create project-specific configuration templates

#### Phase 2: Conservative Rollout (Weeks 2-3)
1. Enable routing for execution workflows only
2. Monitor cost savings and quality metrics
3. Adjust routing aggressiveness based on results
4. Fine-tune quota management thresholds

#### Phase 3: Full Integration (Weeks 4-5)
1. Enable routing for research and verification workflows
2. Implement emergency conservation modes
3. Deploy automated cost optimization features
4. Establish ongoing monitoring and optimization procedures

#### Phase 4: Advanced Optimization (Weeks 6-8)
1. Implement batch processing optimizations
2. Deploy predictive quota management
3. Enable advanced cost-quality trade-off algorithms
4. Establish feedback loops for continuous improvement

### Monitoring and Alerting Setup

#### Cost Tracking Dashboards
```
Real-time cost velocity monitoring
Model usage distribution analysis
Quota utilization trending
Cost-per-task efficiency metrics
```

#### Performance Metrics Collection
```
Task completion times by model
Quality scores (when available)
Error rates and fallback frequency
User satisfaction indicators
```

#### Automated Alerting Configuration
```
Quota threshold breaches (75%, 25%, 10%)
Unusual cost velocity spikes (>200% normal)
Model availability issues affecting workflow
Quality degradation detection
```

#### Optimization Feedback Loops
```
Weekly cost-effectiveness reviews
Monthly routing rule adjustments
Quarterly model performance evaluations
Annual strategic model allocation planning
```

## Decision Trees and Flowcharts

### Master Model Selection Decision Tree

```
Start: New Task Request
├─ Is Routing Enabled?
│  ├─ No → Use Profile Table Lookup → End
│  └─ Yes → Continue to Task Analysis
├─ Task Type Detection
│  ├─ Code Generation Detected?
│  │  ├─ Yes → Route to Codex (unless quality profile) → End
│  │  └─ No → Continue
│  ├─ Analysis Task Detected?
│  │  ├─ Yes → Large Context (>20k tokens)?
│  │  │  ├─ Yes → Route to Gemini Pro → End
│  │  │  └─ No → Consider Profile Rules → Continue
│  │  └─ No → Continue
│  ├─ Planning Task Detected?
│  │  ├─ Yes → Preserve Profile Selection (Opus preferred) → End
│  │  └─ No → Continue
│  └─ Debugging Task Detected?
│     ├─ Yes → Route to Codex → End
│     └─ No → Continue to Profile Lookup
└─ Profile Table Lookup → End
```

### Context-Based Routing Flowchart

```
Context Analysis Start
├─ Measure Content Size
│  ├─ >50k tokens → Force Gemini Pro (if analysis) or Chunking Strategy
│  ├─ >20k tokens → Prefer Gemini Pro (analysis) or Sonnet (code)
│  └─ <20k tokens → Continue to Content Type Analysis
├─ Content Type Classification
│  ├─ Predominantly Code → Prefer Codex
│  ├─ Predominantly Documentation → Prefer Gemini Pro
│  ├─ Mixed Content → Apply Task Type Rules
│  └─ Unknown → Use Profile Default
└─ Apply Aggressiveness Factor
   ├─ Conservative → Only route on clear matches
   ├─ Balanced → Apply standard thresholds
   └─ Aggressive → Prefer alternatives when cost-effective
```

### Emergency Escalation Pattern

```
Resource Monitoring
├─ Quota Usage >75%? → Yellow Alert → Conservative Routing
├─ Quota Usage >90%? → Orange Alert → Emergency Conservation
├─ Quota Usage >95%? → Red Alert → Critical Tasks Only
├─ Model Failures >3/hour? → Service Alert → Fallback Chains
└─ Cost Velocity >200% Normal? → Budget Alert → Aggressive Optimization
```

### Quality vs Cost Trade-Off Guidelines

```
Quality Requirements Assessment
├─ Architecture Decision Required?
│  ├─ Yes → Use Best Available Model (Opus preferred) → Accept Cost
│  └─ No → Continue to Cost Analysis
├─ Implementation Task?
│  ├─ Yes → Code Generation? → Route to Codex → Save Cost
│  └─ No → Continue to Context Analysis
├─ Research/Analysis Task?
│  ├─ Yes → Large Context? → Route to Gemini Pro → Optimize Cost
│  └─ No → Continue to Profile Rules
└─ Use Profile Default → Balanced Approach
```

## System Integration Section

### Cross-References to Existing Model Balancing System

This orchestrator balancing rules document seamlessly extends and coordinates with the existing GSD model balancing ecosystem:

#### Integration with model-profiles.md
- **Profile Definitions:** Inherits all profile table definitions (quality, balanced, budget, openai-codex, gemini-pro)
- **Agent Mappings:** Uses existing agent-to-model mappings as fallback when routing doesn't apply
- **Philosophy Preservation:** Maintains profile philosophy while adding intelligent routing layer
- **Backward Compatibility:** All existing profiles continue to function exactly as before

Reference: `@.config/opencode/get-shit-done/references/model-profiles.md`

#### Integration with model-routing-logic.md
- **Task Detection:** Implements the same task type detection patterns and keywords
- **Routing Rules:** Extends the routing rules with orchestrator-level decision making
- **Performance Characteristics:** Uses the same model comparison matrix for routing decisions
- **Fallback Logic:** Implements the same fallback chains with orchestrator-aware enhancements

Reference: `@.config/opencode/get-shit-done/references/model-routing-logic.md`

#### Integration with model-profile-resolution.md
- **Resolution Process:** Extends the enhanced resolution pattern with orchestrator-level coordination
- **Configuration Options:** Inherits and extends configuration options for multi-workflow scenarios
- **Usage Patterns:** Implements the same usage patterns with workflow-level optimization

Reference: `@.config/opencode/get-shit-done/references/model-profile-resolution.md`

### Clear Separation of Concerns

#### Individual Agent Model Selection (Existing System)
- **Scope:** Single agent task execution
- **Decision Point:** Task spawn time
- **Optimization Goal:** Task-level cost/quality balance
- **Configuration:** Per-task routing and profile selection

#### Orchestrator-Level Model Balancing (This Document)
- **Scope:** Multi-agent workflow coordination
- **Decision Point:** Workflow planning time
- **Optimization Goal:** Project-level resource management
- **Configuration:** Workflow-level strategy and resource allocation

### Orchestrator-Specific Extensions

#### Rules Beyond Individual Agent Selection

**1. Workflow-Level Resource Planning**
```
Planning Phase Resource Allocation:
- Reserve 40% of Opus quota for architecture decisions
- Pre-allocate Codex quota for known code generation phases
- Budget Gemini Pro usage for document-heavy research phases
```

**2. Cross-Phase Optimization Strategies**
```
Sequential Phase Optimization:
- Planning → Research → Execution → Verification
- Carry forward context between phases to reduce redundant processing
- Batch similar tasks across phases for cost efficiency
```

**3. Multi-Workflow Coordination**
```
Parallel Project Management:
- Quota sharing across multiple active projects
- Priority-based resource allocation during capacity constraints
- Load balancing across available model endpoints
```

#### Strategic Decision Frameworks for Complex Project Scenarios

**1. Architecture vs Implementation Trade-offs**
```
High Architecture Complexity + Simple Implementation:
- Invest heavily in Opus for planning phase
- Optimize execution with Codex routing
- Minimal verification requirements

Low Architecture Complexity + Complex Implementation:
- Use Sonnet for straightforward planning
- Invest in Codex/Sonnet for implementation quality
- Enhanced verification with appropriate models
```

**2. Research vs Development Balance**
```
Research-Heavy Projects:
- Aggressive Gemini Pro routing for analysis phases
- Preserve Claude models for synthesis and decision making
- Optimize for large context processing efficiency

Development-Heavy Projects:
- Aggressive Codex routing for implementation phases
- Minimal research overhead
- Focus on code quality and debugging capabilities
```

**3. Quality vs Speed vs Cost Optimization**
```
Quality-First Scenarios:
- Critical infrastructure, production systems
- Use best available models regardless of cost
- Implement redundant verification procedures

Speed-First Scenarios:
- Prototyping, proof-of-concept development
- Aggressive routing to fastest models
- Minimal verification overhead

Cost-First Scenarios:
- Learning projects, internal tools
- Maximum routing to alternative models
- Acceptable quality degradation for cost savings
```

## Implementation Roadmap

### Immediate Deployment Steps (0-1 day)

**1. Configuration Setup**
```bash
# Add orchestrator balancing config to existing .planning/config.json
{
  "model_profile": "balanced",
  "orchestrator_balancing": {
    "enabled": true,
    "strategy": "conservative",
    "quota_management": true
  }
}
```

**2. Monitoring Infrastructure**
```bash
# Enable cost tracking and alerting
echo "ORCHESTRATOR_MONITORING=enabled" >> .env
echo "QUOTA_ALERTS=true" >> .env
echo "COST_TRACKING=detailed" >> .env
```

**3. Gradual Activation**
```bash
# Start with execution workflow routing only
gsd-config set orchestrator.routing.workflows "execution"
gsd-config set orchestrator.routing.aggressiveness "conservative"
```

### Short-term Optimization Goals (1-7 days)

**1. Performance Baseline Establishment**
- Monitor current cost patterns without routing
- Establish quality metrics for comparison
- Identify highest-impact optimization opportunities

**2. Conservative Routing Rollout**
- Enable code generation routing to Codex
- Monitor cost savings and quality maintenance
- Adjust routing aggressiveness based on results

**3. Quota Management Implementation**
- Deploy automated quota monitoring
- Implement yellow/orange/red threshold alerts
- Test emergency conservation mode activation

### Medium-term Enhancement Targets (1-4 weeks)

**1. Full Routing Integration**
- Enable routing for all workflow types
- Deploy advanced context-aware routing
- Implement batch processing optimizations

**2. Advanced Analytics**
- Deploy cost-effectiveness dashboards
- Implement predictive quota management
- Establish performance trending analysis

**3. Cross-Project Optimization**
- Implement quota sharing across projects
- Deploy priority-based resource allocation
- Establish load balancing across model endpoints

### Long-term Strategic Improvements (1-3 months)

**1. Machine Learning Integration**
- Deploy adaptive routing based on historical performance
- Implement quality prediction models
- Establish automated optimization feedback loops

**2. Advanced Resource Management**
- Implement predictive capacity planning
- Deploy dynamic cost optimization algorithms
- Establish advanced emergency response procedures

**3. Ecosystem Integration**
- Integration with external cost monitoring tools
- API integration for real-time model performance data
- Automated model capability discovery and integration

## Validation and Testing Framework

### Automated Testing Procedures

**1. Routing Rule Testing**
```bash
# Test task type detection accuracy
gsd-test routing-detection --sample-size 100 --validate-accuracy
# Expected: >90% accuracy for clear task types

# Test model selection consistency
gsd-test model-selection --profile all --routing enabled
# Expected: Consistent selection for identical inputs

# Test fallback chain reliability
gsd-test fallback-chains --simulate-failures
# Expected: Graceful degradation without workflow failure
```

**2. Cost Optimization Testing**
```bash
# Test cost savings against baseline
gsd-test cost-optimization --baseline profile-only --duration 7d
# Expected: 30-60% cost reduction with quality preservation

# Test quota management effectiveness
gsd-test quota-management --simulate-high-usage
# Expected: Smooth degradation without service interruption

# Test emergency mode activation
gsd-test emergency-mode --trigger quota-95-percent
# Expected: Immediate conservation activation
```

### Performance Benchmarking Methodologies

**1. Quality Preservation Benchmarks**
```
Architecture Decision Quality:
- Compare Opus vs routed model outputs for planning tasks
- Measure architectural soundness and completeness
- Target: <5% quality degradation with 40% cost savings

Implementation Accuracy:
- Compare Codex vs Claude outputs for code generation
- Measure functional correctness and code quality
- Target: Equal or better accuracy with 60% cost savings

Research Completeness:
- Compare Gemini vs Claude outputs for analysis tasks
- Measure information completeness and insight quality
- Target: Equal completeness with 50% cost savings
```

**2. Cost-Effectiveness Metrics**
```
Cost per Successful Task:
- Measure total cost divided by completed tasks
- Track across different routing strategies
- Target: 40% reduction compared to profile-only baseline

Time to Completion:
- Measure end-to-end workflow completion time
- Account for model response times and routing overhead
- Target: <10% increase in total time

Resource Utilization Efficiency:
- Measure quota usage patterns and waste reduction
- Track model capacity utilization
- Target: 90% efficient quota utilization
```

### A/B Testing Frameworks

**1. Routing Strategy Comparison**
```
Control Group: Profile-only selection
Test Group A: Conservative routing
Test Group B: Balanced routing  
Test Group C: Aggressive routing

Metrics: Cost, quality, completion time, user satisfaction
Duration: 2 weeks per group
Sample Size: 50 projects per group
```

**2. Optimization Feature Testing**
```
Feature Tests:
- Batch processing vs individual task execution
- Predictive vs reactive quota management
- Static vs adaptive routing thresholds

Measurement Framework:
- Randomized feature flag assignment
- Continuous monitoring of key metrics
- Statistical significance testing
```

### Success Metrics and KPI Definitions

**1. Primary Success Metrics**
```
Cost Efficiency:
- Total cost reduction: Target 40-60%
- Cost per completed task: Track weekly
- Quota utilization efficiency: Target >90%

Quality Preservation:
- Architecture decision quality: Target >95% of Opus-only baseline
- Implementation accuracy: Target ≥100% of baseline
- Research completeness: Target ≥95% of baseline

Resource Management:
- Quota overrun frequency: Target <2% of billing periods
- Emergency mode activation: Target <5% of workflows
- Model availability resilience: Target >99% workflow completion
```

**2. Secondary Success Metrics**
```
User Experience:
- Workflow completion time: Target <110% of baseline
- User satisfaction scores: Target >4.0/5.0
- Error rate reduction: Target >20% improvement

Operational Excellence:
- Model switching efficiency: Target <100ms routing overhead
- Monitoring accuracy: Target >95% prediction accuracy
- Alert relevance: Target <10% false positive rate
```

## Documentation Maintenance Guidelines

### Update Procedures for New Models

**1. Model Integration Checklist**
```bash
# Add new model to routing logic
1. Update task type detection patterns
2. Add performance characteristics to comparison matrix
3. Define cost-effectiveness thresholds
4. Test routing accuracy and fallback behavior
5. Update configuration templates
6. Deploy monitoring for new model usage
```

**2. Documentation Updates Required**
```
Files to Update:
- model-profiles.md: Add new profile definitions
- model-routing-logic.md: Add routing rules and characteristics
- gsd-orchestrator-balancing-rules.md: Update orchestrator strategies
- Configuration templates: Add new model options

Review Process:
- Technical accuracy validation
- Cost-effectiveness analysis
- Integration testing with existing workflows
- User documentation updates
```

### Performance Data Integration

**1. Automated Performance Data Collection**
```bash
# Collect model performance metrics
gsd-monitor collect-metrics --models all --duration 30d
gsd-analyze cost-effectiveness --output performance-report.json
gsd-update routing-thresholds --source performance-report.json
```

**2. Rule Refinement Procedures**
```
Weekly Performance Reviews:
- Analyze cost and quality metrics
- Identify routing rule optimization opportunities
- Test proposed threshold adjustments
- Deploy incremental improvements

Monthly Strategic Reviews:
- Evaluate overall orchestrator effectiveness
- Assess new model integration opportunities  
- Review and update configuration templates
- Plan long-term optimization strategies
```

### Feedback Loop Establishment

**1. User Feedback Collection**
```
Feedback Channels:
- Automated workflow completion surveys
- Cost alert and optimization notifications
- Quality degradation detection and reporting
- Manual feedback submission system

Response Procedures:
- Acknowledge feedback within 24 hours
- Investigate reported issues within 48 hours
- Deploy fixes within 1 week for critical issues
- Communicate resolution to affected users
```

**2. Continuous Improvement Process**
```
Improvement Cycle:
1. Collect performance and feedback data (ongoing)
2. Analyze optimization opportunities (weekly)
3. Design and test improvements (bi-weekly)
4. Deploy validated improvements (monthly)
5. Monitor improvement effectiveness (ongoing)
6. Document lessons learned (quarterly)
```

### Regular Review and Optimization Schedules

**1. Daily Monitoring**
- Quota usage and cost velocity
- Model availability and performance
- Error rates and fallback frequency
- Critical alert resolution

**2. Weekly Reviews**
- Cost-effectiveness analysis
- Quality metrics assessment
- Routing accuracy evaluation
- User satisfaction tracking

**3. Monthly Optimization**
- Routing threshold adjustments
- Configuration template updates
- New optimization feature deployment
- Performance benchmark updates

**4. Quarterly Strategic Planning**
- Comprehensive effectiveness review
- Long-term optimization roadmap
- New model evaluation and integration
- Documentation and training updates

This comprehensive orchestrator balancing rules document provides immediate integration capabilities with the existing GSD model balancing system while extending it with workflow-level optimization strategies, resource preservation rules, and cost optimization frameworks. All configurations are immediately deployable and all procedures are designed for seamless integration with existing GSD workflows.