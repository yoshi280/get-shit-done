---
phase: 3-create-a-comprehensive-multi-model-balan
plan: 3
type: execute
wave: 1
depends_on: []
files_modified:
  - .config/opencode/get-shit-done/references/gsd-orchestrator-balancing-rules.md
autonomous: true

must_haves:
  truths:
    - "Orchestrator has clear rules for model selection across all GSD workflows"
    - "Resource preservation strategies are documented with specific thresholds"
    - "Cost optimization guidelines provide actionable decision frameworks"
  artifacts:
    - path: ".config/opencode/get-shit-done/references/gsd-orchestrator-balancing-rules.md"
      provides: "Comprehensive orchestrator discretion rules for model balancing"
      contains: "resource_preservation|cost_optimization|workflow_specific_rules"
      min_lines: 150
  key_links:
    - from: "gsd-orchestrator-balancing-rules.md"
      to: "model-profiles.md"
      via: "profile selection logic"
      pattern: "profile.*selection"
    - from: "gsd-orchestrator-balancing-rules.md"
      to: "model-routing-logic.md"
      via: "routing decision integration"
      pattern: "routing.*integration"
---

<objective>
Create a comprehensive multi-model balancing rules summary document that provides GSD orchestrators with clear, actionable guidelines for model selection, resource preservation, and cost optimization across all workflows.

Purpose: Enable consistent, intelligent model selection decisions across GSD instances while preserving resources and optimizing costs
Output: Complete orchestrator discretion framework for multi-model balancing with immediate integration capabilities
</objective>

<execution_context>
@/Users/thelorax/.config/Claude/get-shit-done/workflows/execute-plan.md
@/Users/thelorax/.config/Claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.config/opencode/get-shit-done/references/model-profiles.md
@.config/opencode/get-shit-done/references/model-routing-logic.md
@.config/opencode/get-shit-done/references/model-profile-resolution.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Create comprehensive orchestrator balancing rules document</name>
  <files>.config/opencode/get-shit-done/references/gsd-orchestrator-balancing-rules.md</files>
  <action>
    Create a comprehensive orchestrator discretion rules document that synthesizes and extends the existing model balancing system:

    1. **Executive Summary**:
       - Clear value proposition for multi-model balancing
       - Resource preservation goals and cost optimization targets
       - Integration readiness statement

    2. **Core Decision Framework**:
       - Workflow-specific model selection rules (planning, execution, verification, research)
       - Context-aware routing triggers (task type, content size, complexity)
       - Profile-based fallback strategies with clear thresholds

    3. **Resource Preservation Rules**:
       - Quota management strategies for Opus/Sonnet usage
       - Dynamic degradation patterns (quality → balanced → budget)
       - Emergency conservation modes with specific triggers
       - Token budget allocation across workflow phases

    4. **Cost Optimization Strategies**:
       - Alternative model routing for code generation (Codex preference)
       - Large context handling with Gemini Pro routing
       - Batch processing optimization for research phases
       - Per-agent cost-effectiveness guidelines

    5. **Workflow-Specific Rules**:
       - Planning workflows: Preserve reasoning quality (Opus preference)
       - Execution workflows: Optimize for implementation speed (Codex/Sonnet)
       - Research workflows: Balance context handling with cost (Gemini Pro)
       - Verification workflows: Ensure quality without over-provisioning

    6. **Integration Guidelines**:
       - Configuration templates for immediate deployment
       - Gradual migration strategies for existing projects
       - Monitoring and alerting setup for cost tracking
       - Performance metrics and optimization feedback loops

    7. **Decision Trees and Flowcharts**:
       - Visual decision matrices for orchestrator logic
       - Context-based routing flowcharts
       - Emergency escalation patterns
       - Quality vs cost trade-off guidelines

    Reference and synthesize content from existing model-profiles.md, model-routing-logic.md, and model-profile-resolution.md to create a unified, actionable framework.
  </action>
  <verify>test -f .config/opencode/get-shit-done/references/gsd-orchestrator-balancing-rules.md && wc -l .config/opencode/get-shit-done/references/gsd-orchestrator-balancing-rules.md | awk '{print $1 >= 150 ? "PASS" : "FAIL"}'</verify>
  <done>Comprehensive orchestrator balancing rules document exists with complete decision framework for multi-model selection</done>
</task>

<task type="auto">
  <name>Task 2: Create integration configuration templates</name>
  <files>.config/opencode/get-shit-done/references/gsd-orchestrator-balancing-rules.md</files>
  <action>
    Extend the orchestrator balancing rules document with ready-to-use integration configurations:

    1. **Configuration Templates Section**:
       - Default balanced configuration with intelligent routing
       - Budget-optimized configuration for cost-sensitive workflows
       - Quality-focused configuration for critical architecture work
       - Research-heavy configuration with Gemini Pro optimization

    2. **Per-Workflow Optimization Profiles**:
       - Planning-heavy projects (architecture, design, strategy)
       - Implementation-heavy projects (coding, building, deployment)
       - Research-heavy projects (analysis, documentation, investigation)
       - Mixed workflow projects (balanced approach)

    3. **Monitoring and Alerting Templates**:
       - Cost threshold alerts and automatic profile adjustments
       - Quality degradation detection and escalation
       - Performance metrics tracking configurations
       - Resource utilization dashboards

    4. **Migration Scripts and Guidelines**:
       - Step-by-step migration from profile-only to intelligent routing
       - Validation scripts for configuration testing
       - Rollback procedures for problematic configurations
       - Performance comparison tools

    5. **Best Practices Checklist**:
       - Pre-deployment validation steps
       - Ongoing optimization recommendations
       - Troubleshooting common issues
       - Performance tuning guidelines

    Ensure all templates are immediately deployable with minimal customization required for different GSD instances.
  </action>
  <verify>grep -c "Configuration Templates\|Monitoring\|Migration\|Best Practices" .config/opencode/get-shit-done/references/gsd-orchestrator-balancing-rules.md | awk '{print $1 >= 4 ? "PASS" : "FAIL"}'</verify>
  <done>Integration-ready configuration templates and deployment guidelines added to orchestrator balancing rules</done>
</task>

<task type="auto">
  <name>Task 3: Add cross-reference integration with existing system</name>
  <files>.config/opencode/get-shit-done/references/gsd-orchestrator-balancing-rules.md</files>
  <action>
    Complete the orchestrator balancing rules document with seamless integration references to the existing model balancing system:

    1. **System Integration Section**:
       - Cross-references to model-profiles.md for profile definitions
       - References to model-routing-logic.md for detailed routing algorithms
       - References to model-profile-resolution.md for technical implementation
       - Clear separation of concerns between documents

    2. **Orchestrator-Specific Extensions**:
       - Rules that extend beyond individual agent model selection
       - Workflow-level optimization strategies not covered in existing docs
       - Multi-phase coordination and resource planning
       - Strategic decision frameworks for complex project scenarios

    3. **Implementation Roadmap**:
       - Immediate deployment steps (0-1 day)
       - Short-term optimization goals (1-7 days)
       - Medium-term enhancement targets (1-4 weeks)
       - Long-term strategic improvements (1-3 months)

    4. **Validation and Testing Framework**:
       - Automated testing procedures for balancing rules
       - Performance benchmarking methodologies
       - A/B testing frameworks for optimization
       - Success metrics and KPI definitions

    5. **Documentation Maintenance Guidelines**:
       - Update procedures when new models are added
       - Performance data integration for rule refinement
       - Feedback loop establishment with GSD users
       - Regular review and optimization schedules

    Ensure the document serves as both standalone reference and seamless extension of the existing model balancing ecosystem.
  </action>
  <verify>grep -E "(model-profiles\.md|model-routing-logic\.md|model-profile-resolution\.md)" .config/opencode/get-shit-done/references/gsd-orchestrator-balancing-rules.md && grep -c "Integration\|Roadmap\|Validation\|Maintenance" .config/opencode/get-shit-done/references/gsd-orchestrator-balancing-rules.md | awk '{print $1 >= 4 ? "PASS" : "FAIL"}'</verify>
  <done>Orchestrator balancing rules document fully integrated with existing model balancing system with clear cross-references and extension points</done>
</task>

</tasks>

<verification>
1. Comprehensive orchestrator balancing rules document exists
2. Document contains resource preservation and cost optimization strategies
3. Integration templates and deployment guidelines are included
4. Cross-references to existing model balancing system are properly established
5. Document is immediately usable by other GSD instances
</verification>

<success_criteria>
- Orchestrator discretion rules provide clear decision framework for all GSD workflows
- Resource preservation strategies include specific thresholds and escalation procedures
- Cost optimization guidelines offer actionable alternatives for different scenarios
- Integration templates enable immediate deployment across GSD instances
- Document seamlessly extends existing model balancing ecosystem
</success_criteria>

<output>
After completion, create `.planning/quick/3-create-a-comprehensive-multi-model-balan/3-SUMMARY.md`
</output>