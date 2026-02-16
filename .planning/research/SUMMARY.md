# Project Research Summary

**Project:** Get-Shit-Done Framework Extensions
**Domain:** Meta-Prompting / Agentic Workflow Orchestration
**Researched:** 2026-02-16
**Confidence:** MEDIUM-HIGH

## Executive Summary

GSD is a meta-prompting framework that orchestrates specialized AI agents to execute software projects through structured phases. The framework currently uses a fixed 4-dimension research structure (stack, features, architecture, pitfalls), zero npm dependencies, and markdown-based agent definitions. Research shows this approach aligns with 2025-2026 industry trends toward spec-driven development and Agent-Flavored Markdown, but lacks flexibility that modern enterprise users expect from agentic workflows.

The recommended path forward introduces selective research dimensions (Claude-inferred, user-editable), phase iteration with git-based rollback, and cross-phase memory - all while preserving GSD's zero-dependency constraint and markdown-native philosophy. This evolution follows proven patterns from LangGraph (checkpointing), CrewAI (flexible workflows), and Microsoft Agent Framework (state machines with compensation), but implements them using only Node.js built-ins and file-based state management.

Critical risks center on three areas: (1) context leakage between agent sessions requiring dual-tier memory architecture, (2) rollback misconceptions where compensating actions (not database-style transactions) are needed, and (3) over-complexity before validation where teams add multi-agent orchestration prematurely. These risks map directly to specific roadmap phases and can be mitigated through incremental validation, explicit state boundaries, and observability-first implementation.

## Key Findings

### Recommended Stack

GSD's zero-dependency architecture using Node.js built-ins, Markdown + YAML frontmatter, and file-based state management is the correct foundation. This approach aligns with the 2025 industry shift toward spec-driven development where specifications are primary artifacts and code is generated output. The framework already implements core architectural patterns correctly: orchestration over choreography, state machine transitions via STATE.md, and progressive disclosure of context.

**Core technologies:**
- Node.js 18+ built-ins (fs, path, crypto) - zero npm dependency execution, aligns with GSD constraint, reduces supply chain risk
- Markdown + YAML frontmatter - industry standard for agent definitions, adopted by Linux Foundation's Agentic AI Foundation, separates HOW from WHAT
- Git-based checkpointing - natural rollback capability, atomic commits, audit trail without external database
- JSON for state persistence - native Node.js support, lightweight, human-readable for STATE.md companion files

**Critical pattern implementations:**
- Meta-prompting (Task-Agnostic Scaffolding) - LLM decomposes tasks and delegates to specialist sub-agents, enables flexible research dimensions
- Hierarchical Multi-Agent (Manager-Based) - orchestrator coordinates specialist agents, matches GSD's current spawning pattern
- Sequential Pipeline with Stateful Handoffs - linear phase progression with STATE.md persistence at boundaries, extends naturally with rollback
- Progressive Disclosure - load tools/context on-demand to avoid 40-50% token waste from unused definitions

### Expected Features

Research into meta-prompting frameworks reveals GSD needs to evolve from fixed 4-dimension research to flexible, context-aware orchestration while preserving its markdown-native strengths.

**Must have (table stakes):**
- State persistence across phases - all modern frameworks have checkpointing, critical for multi-turn workflows and recovery
- Atomic operation rollback - stateful systems need error recovery, users expect transaction-based approaches
- Human-in-the-loop approval gates - required for compliance and safety, production deployments demand this
- Progress tracking and observability - teams need workflow execution visibility, debugging failures is impossible without it
- Configuration-based workflow definition - rapid iteration without code changes, both declarative and programmatic APIs expected
- Error handling and retry logic - production systems must handle failures gracefully, includes escalation triggers

**Should have (competitive advantage):**
- Flexible/dynamic research dimensions - enables Claude to infer relevant investigation areas vs rigid structure, core value proposition
- Phase iteration with rollback - allows rework without starting over, supports iterative refinement
- Cross-phase learning and awareness - agents reference outputs from earlier phases, apply insights to later work
- Custom workflow templates - users define their own phase structures and dependencies for domain-specific workflows
- Idea capture system - captures emerging requirements during execution for later work, low complexity high value
- Self-evaluation and iterative improvement - agents self-review phase outputs and refine without manual intervention

**Defer (v2+):**
- Swarm coordination patterns - peer agents with emergent coordination, alternative to hierarchical control, HIGH complexity
- Multimodal memory - memory across text, structured data, images, emerging research frontier
- Temporal workflow orchestration - enterprise reliability with pause/resume across process restarts, massive scope increase
- Full learning systems - RL-based optimization for memory collection and deployment, research-level problem

### Architecture Approach

Modern agentic workflows separate concerns across distinct layers: Command (CLI), Orchestration (coordinator, state machine, router), Agent (specialized execution), Workflow Definitions (DSL), Memory (state store, context history, episodic memory), and Tools (execution primitives). GSD already implements this correctly with .claude/get-shit-done/ for system files and .planning/ for project state.

**Major components:**
1. Orchestration Layer - coordinator breaks down requests, state machine tracks phase transitions, router dispatches tasks to agents dynamically
2. Agent Layer - specialized agents (planner, executor, researcher, verifier) with focused prompts and tool access, spawn via Task tool
3. Memory Layer - STATE.md for current position, config.json for configuration, phase summaries for episodic memory, context propagation via append-only history
4. Workflow Definitions - markdown-based DSL defining task sequences, dependencies, compensation logic for rollback
5. Tools Layer - gsd-tools.cjs centralizes atomic operations (state mutations, git operations, config parsing) using only Node.js built-ins

**Key patterns for implementation:**
- Orchestration over Choreography - central coordinator manages workflow execution, provides clear control flow and easier debugging
- State Machine with Compensation - model workflow as FSM with explicit compensating transactions for rollback, not database-style transactions
- Context Propagation via Append-Only History - cumulative context throughout execution passed to each agent, enables cross-phase awareness
- Dynamic Dimension Dispatch - workflow determines which research dimensions to execute based on project type and user input, extensible without modifying orchestrator

### Critical Pitfalls

Research identified seven critical pitfalls that map directly to roadmap phases and require preventive architecture from day one.

1. **Over-Complexity Before Validation** - Building multi-agent systems before validating single agent can't handle the task. More than 40% of agentic AI projects could be cancelled by 2027 due to unanticipated cost and complexity. Start simple, validate single-agent performance on real data before adding orchestration. Implement in Phase 1.

2. **Context Leakage Between Agent Sessions** - Multi-agent configurations reduce per-channel leakage but introduce unmonitored internal channels raising total exposure to 68.9%. Requires dual-tier memory (private/shared), burn-after-use mechanism for ephemeral contexts, audit of all inter-agent communication. Critical for Phase 2.

3. **Rollback Misconceptions and Side Effect Duplication** - Teams import database-style rollback patterns into distributed agent systems. In reality, you get compensating actions, not true rollbacks. Design for idempotency, implement compensating actions with business semantics, persist intermediate state for partial completion recovery. Foundational to Phase 3.

4. **Prompt Injection Attack Surface Expansion** - #1 in OWASP LLM Top 10. Meta-prompting adds recursion layers where malicious instructions can hide. User-defined workflows mean user-controlled input flows into system-level prompt generation. Requires input validation at every agent boundary, structured outputs with schema enforcement, separate user content from system instructions. Critical for Phase 4.

5. **The "50 First Dates" Problem** - Agents have no memory between phases and create conflicting markdown files. Linear execution treats phases as independent units with no persistence for agent decisions or constraints. Requires persistent memory store across phase boundaries, git-based versioning, distinction between ephemeral and long-term memory. Addresses in Phases 2 and 3.

6. **Tool Versioning and Schema Drift** - Causes 60% of production agent failures. External tools change APIs without coordinating with agent framework. Requires strict API contracts, semantic versioning, schema enforcement via Structured Outputs, version tool definitions alongside agent code. Essential for Phase 4.

7. **Observability Gap in Production** - Many failures stem from outdated/incomplete context, not model weaknesses. Without traceability for every decision and tool call, debugging becomes impossible. 89% of successful teams have observability before production. Implement tracing for every decision and tool call, offline evaluations on test sets, monitor token usage/cost/latency/success rate. Required from Phase 1 onward.

## Implications for Roadmap

Based on research, suggested phase structure prioritizes validation before complexity, builds on proven patterns, and addresses critical pitfalls incrementally.

### Phase 1: Selectable Research Dimensions

**Rationale:** Validates core value proposition (flexible dimensions vs fixed structure) with minimal complexity. Tests whether current 4-dimension approach can be simplified before adding more. Establishes observability foundation needed for all subsequent phases.

**Delivers:**
- Dimension registry in gsd-tools.cjs (extensible data structure)
- Claude-inferred dimension selection from project context
- User-editable dimensions via YAML frontmatter
- Observability for research phase (token usage, dimension selection rationale, synthesis quality)

**Addresses features:**
- Flexible/dynamic research dimensions (differentiator)
- Configuration-based workflow definition (table stakes)

**Avoids pitfalls:**
- Over-complexity before validation - single phase, minimal scope, proves concept
- Observability gap - implements tracing before system grows complex

**Research needs:** SKIP - patterns are well-documented (dimension registry, YAML parsing, LLM-based classification)

### Phase 2: Cross-Phase Memory and Context Awareness

**Rationale:** Once dimensions are flexible, phases need to learn from each other. Addresses "50 First Dates" problem before state complexity grows. Requires state persistence foundation from Phase 1.

**Delivers:**
- Dual-tier memory architecture (private per-agent, shared cross-phase)
- Phase summary aggregation and episodic memory in STATE.md
- Context propagation with filtered builders per agent type
- Burn-after-use mechanism for ephemeral conversational contexts

**Addresses features:**
- Cross-phase learning and awareness (differentiator)
- State persistence across phases (table stakes)
- Progress tracking and observability (table stakes)

**Avoids pitfalls:**
- Context leakage between agents - dual-tier architecture with access controls
- "50 First Dates" memory loss - persistent store across phase boundaries
- Unbounded context growth - summarization for contexts >50 entries

**Research needs:** PHASE RESEARCH REQUIRED - Complex integration, need memory architecture patterns for file-based systems (not Redis/Postgres), privacy boundary design

### Phase 3: Phase State Machine with Rollback

**Rationale:** With memory established, add iteration capability. State machine provides explicit failure handling. Git-based rollback leverages existing version control without external dependencies.

**Delivers:**
- Phase state machine (states, transitions, actions, compensation logic)
- Git-based checkpoint/restore at phase boundaries
- Idempotency tracking for operations with side effects
- Rollback operations for single phase with dependency validation

**Addresses features:**
- Atomic operation rollback (table stakes)
- Phase iteration with rollback (differentiator)
- Error handling and retry logic (table stakes)

**Avoids pitfalls:**
- Rollback misconceptions - compensating actions, not database transactions
- Side effect duplication - idempotency keys and duplicate detection
- No compensation logic - explicit undo actions for each state transition

**Research needs:** PHASE RESEARCH REQUIRED - Saga pattern implementation without external frameworks, git-based state restoration, idempotency patterns for LLM operations

### Phase 4: User-Defined Workflows

**Rationale:** Once core mechanics (dimensions, memory, rollback) are stable, enable customization. Highest security risk phase - requires validation and sanitization infrastructure from Phases 1-3.

**Delivers:**
- Workflow DSL in markdown with YAML frontmatter
- Custom dimension definitions (user-extensible registry)
- Workflow validation (dependency checking, recursion limits)
- Input sanitization for prompt injection prevention

**Addresses features:**
- Custom workflow templates (differentiator)
- Configuration-based workflow definition (table stakes)
- Human-in-the-loop approval gates (table stakes for custom workflows)

**Avoids pitfalls:**
- Prompt injection in workflows - input validation at every agent boundary
- Tool schema drift - version pinning for custom tools
- No recursion limits - hard depth caps, timeout enforcement
- Unclear prompts from users - templates, examples, validation

**Research needs:** PHASE RESEARCH REQUIRED - Markdown DSL design (preserve zero-dep), prompt injection prevention patterns (Claude-specific), workflow validation strategies

### Phase 5: Multi-Phase Rollback and Workflow Composition

**Rationale:** Advanced capabilities building on all prior phases. Dependency graph complexity requires proven state machine from Phase 3. Sub-workflow extraction enables reusable modules.

**Delivers:**
- Multi-phase rollback with dependency graph traversal
- Workflow composition primitives (sub-workflow registry, loader)
- Reusable workflow modules
- Phase grouping for 20+ phase projects

**Addresses features:**
- Phase iteration with rollback (full implementation, not just single phase)
- Custom workflow templates (enhanced with composition)

**Avoids pitfalls:**
- Cross-phase dependencies become tangled - explicit dependency tracking
- Context overflow - phase grouping and summarization for large projects

**Research needs:** SKIP - extends Phase 3 patterns, no new architectural concepts

### Phase Ordering Rationale

- **Phase 1 first**: Proves value proposition with minimal risk, establishes observability foundation needed to debug later phases
- **Memory before rollback**: Phase 2 builds state persistence infrastructure that Phase 3 rollback depends on
- **Core mechanics before customization**: Phases 1-3 establish stable platform before Phase 4 opens to user-defined workflows
- **Security progression**: Each phase builds on validation/isolation from prior phases; Phase 4 (highest risk) benefits from defense-in-depth
- **Incremental validation**: Each phase delivers standalone value, can ship to production independently

### Research Flags

**Phases needing deeper research during planning:**

- **Phase 2 (Cross-Phase Memory):** Complex integration, need memory architecture patterns for file-based systems without external dependencies, privacy boundary design for dual-tier architecture, context summarization strategies for large histories

- **Phase 3 (State Machine/Rollback):** Saga pattern implementation without external frameworks (CrewAI/LangGraph use databases), git-based state restoration mechanisms, idempotency patterns for non-deterministic LLM operations, compensation logic design for multi-step workflows

- **Phase 4 (User-Defined Workflows):** Markdown DSL design preserving zero-dependency constraint, prompt injection prevention patterns specific to Claude API, workflow validation strategies (dependency checking, recursion limits, cost budgets), YAML frontmatter schema for workflow definitions

**Phases with standard patterns (skip research):**

- **Phase 1 (Selectable Dimensions):** Dimension registry is standard data structure, YAML parsing well-documented in Node.js, LLM-based classification uses existing Claude capabilities

- **Phase 5 (Multi-Phase Rollback):** Extends Phase 3 patterns with dependency graph traversal (standard graph algorithms), workflow composition uses proven registry/loader patterns

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Zero-dependency Node.js approach verified across multiple 2025-2026 sources, markdown+YAML is industry standard (AGENTS.md, Agent Skills), git-based state aligns with current GSD implementation |
| Features | MEDIUM-HIGH | Table stakes features confirmed across LangGraph, CrewAI, AutoGen docs; differentiators validated via Claude research system and enterprise adoption patterns; some features (swarm coordination, multimodal memory) are emerging research areas |
| Architecture | HIGH | Orchestration patterns documented extensively in AWS/Azure/Google Cloud guidance, state machine with compensation is proven Saga pattern, file-based architecture compatible with zero-dependency constraint |
| Pitfalls | MEDIUM | Critical pitfalls (context leakage, rollback misconceptions, prompt injection) verified with multiple 2025-2026 sources including OWASP, academic research, production postmortems; some mitigation strategies are framework-specific and need adaptation to GSD's approach |

**Overall confidence:** MEDIUM-HIGH

Research is grounded in current (2025-2026) industry practices with extensive source verification. Lower confidence areas are emerging research frontiers (Phase 2 memory architecture for file-based systems, Phase 3 compensation patterns for LLM operations) where adaptation work is needed.

### Gaps to Address

- **File-based memory architecture patterns:** Most documented approaches use Redis, PostgreSQL, or vector databases. GSD's zero-dependency constraint requires adapting these patterns to JSON files and git versioning. Address during Phase 2 planning with prototype validation.

- **Idempotency for non-deterministic LLM operations:** Standard idempotency patterns assume deterministic operations (same input = same output). LLM calls are probabilistic. Need to define "same enough" criteria for cached responses vs. fresh calls. Address during Phase 3 planning with token-level similarity thresholds.

- **Prompt injection prevention without external validators:** Industry tools (Lakera, Rebuff) detect malicious prompts but add dependencies. Need to implement Claude-native validation using structured outputs and meta-prompting for self-critique. Prototype during Phase 4 planning.

- **Cost budgeting for user-defined workflows:** No documented patterns for enforcing token budgets on arbitrary user workflows. Need circuit breaker mechanism with cost estimation before execution. Design during Phase 4 planning with fallback to human approval for expensive operations.

- **Cross-user isolation in single-process file-based system:** Multi-tenant agentic systems typically use database row-level security or separate process sandboxing. GSD runs single-process with file writes. Validate that .planning/ directory isolation is sufficient or needs enhanced access controls. Test during Phase 2 implementation.

## Sources

Research aggregated from 120+ sources across stack, features, architecture, and pitfalls dimensions. Full citations in individual research files.

### Primary (HIGH confidence)

**Frameworks and Orchestration:**
- LangGraph Documentation (https://www.langchain.com/langgraph) - checkpointing, state management, multi-agent workflows
- CrewAI Documentation (https://docs.crewai.com/) - processes, flows, role-based orchestration
- Microsoft Agent Framework (https://learn.microsoft.com/en-us/agent-framework/) - AutoGen v0.4, Semantic Kernel, HITL patterns
- AWS Prescriptive Guidance (https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/)
- Google Cloud Agentic AI Patterns (https://docs.google.com/architecture/choose-design-pattern-agentic-ai-system)

**Spec-Driven Development:**
- AGENTS.md Specification (https://agents.md/) - Agent-Flavored Markdown standard
- Agent Skills IO (https://agentskills.io/specification) - YAML frontmatter for agent definitions
- Thoughtworks Spec-Driven Development (https://www.thoughtworks.com/en-us/insights/blog/agile-engineering-practices/spec-driven-development-unpacking-2025-new-engineering-practices)

**Security and Privacy:**
- OWASP LLM Top 10 (LLM01:2026 Prompt Injection)
- AgentLeak: Privacy Leakage in Multi-Agent Systems (https://arxiv.org/abs/2602.11510)
- OpenAI Prompt Injection Guide (https://openai.com/index/prompt-injections/)
- Lakera Indirect Prompt Injection Research (https://www.lakera.ai/blog/indirect-prompt-injection)

**State Management:**
- Microsoft Azure Saga Pattern (https://learn.microsoft.com/en-us/azure/architecture/patterns/saga)
- SagaLLM Research (https://www.vldb.org/pvldb/vol18/p4874-chang.pdf) - transaction patterns for LLM workflows
- Dapr Workflow Patterns (https://docs.dapr.io/developing-applications/building-blocks/workflow/workflow-patterns/)

### Secondary (MEDIUM confidence)

**Industry Trends:**
- Anthropic Multi-Agent Research System (https://www.anthropic.com/engineering/multi-agent-research-system) - flexible dimensions, parallel subagents
- Claude Code MCP Context Optimization (46.9% reduction via Tool Search)
- LangChain State of Agent Engineering Report (https://www.langchain.com/state-of-agent-engineering)
- Deloitte AI Agent Orchestration (https://www.deloitte.com/us/en/insights/industry/technology/technology-media-and-telecom-predictions/2026/ai-agent-orchestration.html)

**Cost and Performance:**
- Token optimization strategies (40-70% savings with caching, 40-50% reduction via concise prompting)
- Output tokens cost 4-8x input tokens across providers
- Multi-agent overhead: 2-3x token cost vs single agent (reflection pattern), 2-5x speedup (CrewAI parallel execution)

### Tertiary (LOW confidence - needs validation)

- 40% of agentic AI projects at risk of cancellation by 2027 (industry analyst estimate, not peer-reviewed)
- 89% of successful teams implement observability before production (survey data, sample size unknown)
- 52% implement offline evaluations before deployment (2026 survey, methodology unclear)
- Context window utilization >50% shows diminishing returns (anecdotal, not rigorously tested)

---
*Research completed: 2026-02-16*
*Ready for roadmap: yes*
