# Stack Research

**Domain:** Meta-Prompting / Agentic Workflow Orchestration
**Researched:** 2026-02-16
**Confidence:** HIGH

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Node.js Built-ins | 18+ | Runtime environment, zero-dependency execution | GSD's constraint. Built-ins (fs, path, crypto) provide file I/O, module loading, state serialization without external dependencies. Aligns with zero npm dependency requirement. |
| Markdown + YAML Frontmatter | N/A | Agent definition, configuration, specification format | Industry standard (AGENTS.md, Agent-Flavored Markdown). Separates HOW (frontmatter) from WHAT (markdown content). Human-readable, tool-parseable, familiar to developers and domain experts. Adopted by Linux Foundation's Agentic AI Foundation. |
| JSON | N/A | State persistence, configuration storage | Node.js native support. Lightweight, structured, human-readable for STATE.md companion. No external serialization libraries needed. |
| Spec-Driven Development (SDD) | N/A | Development paradigm | 2025 industry shift toward specifications as primary artifact, code as generated output. Enables questioning → research → requirements → roadmap → plan → execute → verify workflow. Supported by major IDEs (IBM Project Bob, AWS Kiro). |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| None required | N/A | GSD operates with zero npm dependencies | Built-in Node.js modules (fs, path, process, child_process) handle all orchestration, file I/O, and subprocess spawning needs |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| Node.js REPL | Testing agent logic | For rapid prototyping of orchestration patterns |
| `node --inspect` | Debugging | Native debugging for workflow execution |
| Git | Version control | Track agent definitions, state evolution, workflow changes |

## Installation

```bash
# No installation required - zero dependencies
# Requires Node.js 18+ only
node --version  # Should be >= 18.0.0
```

## Alternatives Considered

### For Agent Orchestration Frameworks

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Custom zero-dependency orchestrator (GSD) | LangGraph | When you need graph-based state machines, durable execution, time-travel debugging, production-grade error recovery. Best for complex branching workflows with parallel execution. |
| Custom zero-dependency orchestrator (GSD) | CrewAI | When you need role-based agent teams with 2-3x faster execution. Best for Fortune 500 enterprise scenarios with high-throughput requirements. Requires Python. |
| Custom zero-dependency orchestrator (GSD) | Microsoft Agent Framework (AutoGen + Semantic Kernel) | When you need .NET/Python compatibility, enterprise integration, asynchronous event-driven architecture. GA Q1 2026. Best for Microsoft ecosystem. |
| Markdown + YAML Frontmatter | Pure JSON/YAML config | When you need programmatic-only configuration without human-readable documentation embedded in agent definitions. Loses natural language instructions benefit. |
| Spec-Driven Development | Traditional code-first development | When specifications are unclear or rapidly changing. SDD requires well-defined specs upfront. Code-first better for exploratory prototyping. |

### For State Management Patterns

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Centralized STATE.md (file-based) | LangGraph checkpointing | When you need automatic state snapshots at every step, time-travel debugging, replay from exact failures. Requires LangGraph dependency. |
| Centralized STATE.md (file-based) | Vector database (Pinecone, Milvus, Qdrant) | When you need semantic search over historical states, RAG-powered context retrieval, billions of state vectors. Overkill for simple phase-based workflows. |
| Centralized STATE.md (file-based) | Event-driven (Kafka, Redis Streams) | When you need distributed multi-agent coordination, partitioned workloads, consumer groups. Requires infrastructure overhead. |

### For Context Management

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| File-based context propagation | Model Context Protocol (MCP) | When you need standardized tool integration, dynamic context loading, third-party MCP server ecosystem. Adds protocol complexity. Best for Claude Code/Gemini integration. |
| File-based context propagation | Central state manager (React/Redux pattern) | When you need loose coupling between agents, testable state mutations, time-travel debugging. Requires shared memory architecture. |
| Progressive disclosure (load on demand) | Full context preloading | When token usage is not a constraint and you need all tools/context available immediately. Wastes 40-50% of context window with unused tool definitions. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| LangChain (full framework) | Heavy dependency footprint, over-engineered for markdown-based orchestration. GSD's zero-dependency constraint violated. | Node.js built-ins + custom orchestration logic |
| Python-based frameworks (CrewAI, AutoGen v0.4) | Language mismatch. GSD is Node.js-only. Cross-language orchestration adds complexity. | Node.js native solutions |
| Complex state machines (XState, Robot) | External dependencies. Overkill for linear phase progression with occasional branching. | Simple state transitions in STATE.md |
| Global npm packages | Violates zero-dependency principle. Deployment friction, version conflicts, security audit burden. | Node.js standard library only |
| GraphQL for agent communication | Unnecessary query language abstraction. Agents communicate via file writes, not client-server protocol. | Direct file I/O with JSON |
| WebSockets/HTTP servers for orchestration | Introduces network layer, deployment complexity, error surface. GSD is single-process, file-based. | File-based message passing |

## Stack Patterns by Variant

**If building for Claude Code/OpenCode/Gemini CLI integration:**
- Use AGENTS.md / Agent Skills format for compatibility
- Implement MCP servers for tool integration if third-party context needed
- Because these CLIs expect markdown-based agent definitions with YAML frontmatter

**If adding flexible research dimensions:**
- Store dimension metadata in frontmatter (e.g., `research_dimensions: ["STACK", "FEATURES", "ARCHITECTURE"]`)
- Use Claude inference to extract dimensions from project context, write to config.json
- Allow user override via YAML edits
- Because dimensions should be discoverable, editable, and persisted across phases

**If implementing phase iteration/rollback:**
- Checkpoint STATE.md before each phase transition (`STATE.md.backup-phase-N`)
- Store phase execution logs with timestamps
- Implement rollback as file restore + state pointer reset
- Because 2025 enterprise adoption requires confidence in reversibility (see developer expectations)

**If enabling cross-phase awareness:**
- Maintain `LEARNINGS.md` or append to STATE.md with `phase_context` sections
- Propagate artifacts forward (e.g., Phase 2 research feeds Phase 3 roadmap)
- Use progressive summarization to avoid context bloat
- Because multi-agent systems need durable context that survives phase boundaries

**If supporting custom user-defined workflows:**
- Define workflow as ordered list in config.json: `["questioning", "research", "requirements", "roadmap", "plan", "execute", "verify"]`
- Allow user to reorder, skip, or inject custom phases
- Validate that dependencies are met (e.g., research must precede roadmap)
- Because enterprise users have domain-specific SDLCs that don't fit rigid templates

## Version Compatibility

| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| Node.js 18+ | GSD zero-dependency orchestrator | Requires ES modules, top-level await, native fetch |
| Node.js 20+ | GSD zero-dependency orchestrator | Recommended. Better error stack traces, faster startup |
| Markdown parser (none) | YAML frontmatter | Use regex or simple split on `---` delimiters. No markdown-it or remark needed. |

## Architectural Patterns for Meta-Prompting

### Pattern 1: Task-Agnostic Scaffolding (Meta-Prompting)
**What:** LLM acts as meta-coordinator, decomposing tasks and delegating to specialist sub-agents
**When:** Complex multi-step workflows where task structure is dynamic
**Implementation:**
```javascript
// Orchestrator prompts meta-agent to decompose task
const metaPrompt = `Given task: ${userTask}, decompose into research dimensions.`;
const dimensions = await llm.complete(metaPrompt); // e.g., ["STACK", "FEATURES", "PITFALLS"]
// Spawn dimension-specific agents
for (const dim of dimensions) {
  await spawnAgent(`research-${dim.toLowerCase()}`, { dimension: dim });
}
```
**Why for GSD:** Enables flexible research dimensions (Claude-inferred, user-editable)

### Pattern 2: Reflection (Self-Correction)
**What:** Agent generates output, critiques it, refines iteratively
**When:** Quality matters more than speed. Low error tolerance (compliance, code generation)
**Implementation:**
```javascript
// Generate → Critique → Refine loop
let output = await agent.generate(task);
let critique = await agent.reflect(output, criteria);
while (critique.needsImprovement && iterations < maxIterations) {
  output = await agent.refine(output, critique);
  critique = await agent.reflect(output, criteria);
  iterations++;
}
```
**Why for GSD:** Research outputs (STACK.md, PITFALLS.md) benefit from self-validation
**Tradeoff:** 2-3x token cost, higher latency. McKinsey 2025: 64% of orgs report quality improvement justifies cost.

### Pattern 3: Hierarchical Multi-Agent (Manager-Based)
**What:** Manager agent assigns tasks to specialized workers, coordinates outputs
**When:** Clear role separation, need centralized control
**Implementation:**
```javascript
// Manager coordinates specialist agents
const manager = new ManagerAgent();
const researchers = [stackAgent, featuresAgent, architectureAgent];
const tasks = manager.decompose(projectContext);
const results = await Promise.all(tasks.map(t => assignToSpecialist(t, researchers)));
const synthesis = manager.synthesize(results);
```
**Why for GSD:** Matches GSD's orchestrator → specialist spawning pattern
**Current:** gsd-project-researcher spawns dimension-specific research

### Pattern 4: Sequential Pipeline with Stateful Handoffs
**What:** Linear phase progression with state persisted at boundaries
**When:** Dependencies are clear, phases build on prior outputs
**Implementation:**
```javascript
// Each phase writes to STATE.md, next phase reads
await questioningPhase(); // Writes STATE.md with questions
const questions = readState();
await researchPhase(questions); // Writes STATE.md with research
const research = readState();
await roadmapPhase(research); // Writes STATE.md with roadmap
```
**Why for GSD:** Already implemented. Extends naturally with rollback (checkpoint before writes)

### Pattern 5: Event-Driven with Central State Manager
**What:** Agents react to state changes, publish events to shared store
**When:** Need loose coupling, testability, multiple agents reading/writing concurrently
**Implementation:**
```javascript
// Central state with subscribers
class StateManager {
  constructor() { this.state = {}; this.subscribers = []; }
  update(key, value) {
    this.state[key] = value;
    this.subscribers.forEach(sub => sub.notify(key, value));
  }
}
const state = new StateManager();
state.subscribe(roadmapAgent, (key) => key === 'research_complete');
```
**Why for GSD:** Alternative to file polling for multi-agent coordination
**Tradeoff:** Adds complexity vs. simple file-based handoffs

### Pattern 6: Progressive Disclosure (Context Management)
**What:** Load tools/context on-demand, not preloaded
**When:** Many tools available, context window limited
**Implementation:**
```javascript
// Instead of: prompt = systemPrompt + allTools + userMessage (50K tokens)
// Do: prompt = systemPrompt + relevantTools(userMessage) + userMessage (10K tokens)
function relevantTools(message) {
  // Semantic search or keyword matching
  return tools.filter(t => t.isRelevantTo(message)).slice(0, 5);
}
```
**Why for GSD:** If tool count grows, avoid preloading all in orchestrator prompt
**Evidence:** Claude Code reduced MCP context bloat 46.9% (51K → 8.5K tokens) with Tool Search

## Anti-Patterns to Avoid

### Anti-Pattern 1: Chatbot Mandate
**What:** Forcing conversational interface when automation is better
**Why bad:** User intent is spec-driven (markdown files), not conversational. GSD should execute deterministically from specs, not chat.
**Instead:** Read specs, execute, report structured results

### Anti-Pattern 2: Silent Confabulation
**What:** RAG-powered agents inventing facts without citation
**Why bad:** Research outputs (STACK.md) must be verifiable. LOW confidence without sources = misleading roadmap.
**Instead:** Mandate source URLs in research output. Use WebSearch → verify with official docs pattern.

### Anti-Pattern 3: Infinite Tool Calling Loops
**What:** Agent repeatedly calls same tool or loops without exit condition
**Why bad:** Wastes tokens, stalls workflow, no human intervention trigger
**Instead:** Set max iterations (e.g., 5 reflection cycles), timeout, HITL escalation

### Anti-Pattern 4: Negative Instructions
**What:** Telling agents "Don't do X" instead of "Do Y"
**Why bad:** LLMs reason by examples, not negation. "Don't use deprecated libs" is vague.
**Instead:** "Use [specific current lib]" with rationale

### Anti-Pattern 5: Context Rot (Unbounded Growth)
**What:** Appending all state to growing context without summarization
**Why bad:** Models degrade as context grows. Diminishing returns past 50% window usage.
**Instead:** Progressive summarization. Archive old phase context, propagate only essentials.

### Anti-Pattern 6: Assigning Deterministic Tasks to Agents
**What:** Using LLM to send emails, parse JSON, run git commands
**Why bad:** Wastes model capability, introduces hallucination risk, slower than direct execution
**Instead:** Agents decide WHAT to do, system executes HOW (spawn Bash, Write, Edit tools)

### Anti-Pattern 7: Instructing by API Name
**What:** "Use the fetchData tool" instead of "Retrieve user data"
**Why bad:** Agents reason semantically, not by metadata. Tight coupling to tool naming.
**Instead:** Describe intent, let agent map to available tools

## Scalability Considerations

| Concern | At 1 Project | At 10 Concurrent Projects | At 100 Concurrent Projects |
|---------|--------------|---------------------------|---------------------------|
| State management | Single STATE.md file | Directory per project (.planning-{id}/) | Database or key-value store (Redis) for state |
| Agent spawning | Inline orchestration | Process pool with worker queue | Distributed task queue (no external deps = rebuild in Node.js or accept dependency) |
| Context window | Full history in prompt | Summarize phases > 3 steps old | Vector DB for semantic retrieval (contradicts zero-dep) |
| Rollback storage | Git commits or .backup files | Same, per-project isolation | S3/blob storage for checkpoints |
| Observability | Console logs, file writes | Structured JSON logs per project | OpenTelemetry (external dep) or custom aggregator |

**Note:** At 100 concurrent projects, zero-dependency constraint may need reevaluation. Distributed coordination typically requires Redis/Kafka/DB, unless rebuilt in Node.js (high effort).

## Context Management: Key 2025-2026 Patterns

### Model Context Protocol (MCP)
**What:** Universal framework standardizing how AI agents connect to tools, models, systems
**Why matters:** Solves context bloat (dynamic loading), makes agent behavior traceable
**Adoption:** Anthropic (November 2024), early 2025 ecosystem adoption, Claude Code native support
**For GSD:** Consider MCP servers if integrating third-party tools (e.g., GitHub, Jira, Confluence)
**Tradeoff:** Adds protocol layer vs. simple file-based context

### Progressive Disclosure
**What:** Show only essential information upfront, load details on-demand
**Why matters:** Models degrade as context grows. Token budgets are finite.
**Evidence:** Claude Code Tool Search reduced context 46.9% (51K → 8.5K tokens)
**For GSD:** If agent count grows, avoid loading all agent definitions in orchestrator prompt

### Context Propagation Across Phases
**What:** Carry forward learnings from prior phases without full context replay
**Patterns:**
- **Branching:** Divergent phases spawn separate context branches
- **Merging:** Convergent phases aggregate contexts via hierarchical summarization
**For GSD:** Phase N writes artifacts (STACK.md, FEATURES.md) → Phase N+1 reads as input → STATE.md tracks what propagated

### Adaptive Memory (Agent Memory vs. RAG)
**What:** Shift from static retrieval (RAG) to read-write memory that evolves with interaction
**Why matters:** Agents need to learn from feedback, maintain state across sessions
**For GSD:** Cross-phase awareness requires durable memory (STATE.md acts as this)
**Future:** If GSD needs semantic search over past executions, consider vector DB (breaks zero-dep)

## Observability Requirements for Production

| Capability | Why Needed | Implementation (Zero-Dep) |
|------------|-----------|---------------------------|
| Tracing | Debug multi-agent coordination, identify bottlenecks | Write execution logs to `.planning/logs/{timestamp}.json` per phase |
| Metrics | Track phase duration, token usage, success rates | Append metrics to STATE.md or separate metrics.json |
| Alerting | Catch failures, infinite loops, HITL escalations | Console errors + exit codes. External monitoring reads logs. |
| Governance | Audit agent decisions, ensure compliance | Log all LLM prompts/responses to audit trail file |
| Rollback | Revert to known-good state on failure | Git commits per phase OR `.backup` files before state writes |

**Industry standard:** OpenTelemetry for distributed tracing (external dependency)
**GSD alternative:** Structured JSON logs + custom aggregation script

## Human-in-the-Loop (HITL) Integration

**When needed:**
- Low error tolerance (legal, financial, medical)
- Sensitive actions (delete data, deploy to prod, send external communications)
- Regulatory compliance (EU AI Act requires human oversight for high-risk AI)

**Implementation patterns:**
1. **Interrupt on threshold:** If confidence < 0.7, pause and request human review
2. **Approval gates:** Before phase transitions, show summary and wait for approval
3. **Edit before commit:** Human reviews generated artifacts (STACK.md) before finalizing

**For GSD:**
```javascript
// Before writing STATE.md
if (requiresApproval(phase)) {
  console.log(`Phase ${phase} complete. Review .planning/research/ files.`);
  await waitForUserApproval(); // Blocks until user runs `gsd:approve`
}
```

**Frameworks with native HITL:**
- LangGraph: `.interrupt()` API with `HITLRequest`/`HITLResponse`
- Microsoft Agent Framework: Built-in HITL for all orchestrations
- CrewAI: Human-as-decision-maker pattern

## Sources

### Meta-Prompting Frameworks
- [Meta-Prompting: Enhancing Language Models with Task-Agnostic Scaffolding](https://arxiv.org/pdf/2401.12954)
- [Meta Prompting | Prompt Engineering Guide](https://www.promptingguide.ai/techniques/meta-prompting)
- [Prompt engineering techniques: Top 6 for 2026](https://www.k2view.com/blog/prompt-engineering-techniques/)

### Agentic Workflow Orchestration
- [The 2026 Guide to Agentic Workflow Architectures](https://www.stack-ai.com/blog/the-2026-guide-to-agentic-workflow-architectures)
- [Top 10+ Agentic Orchestration Frameworks & Tools in 2026](https://aimultiple.com/agentic-orchestration)
- [Agentic AI Workflows: Why Orchestration with Temporal is Key](https://intuitionlabs.ai/articles/agentic-ai-temporal-orchestration)
- [Top AI Agentic Workflow Patterns Enterprises Should Use in 2026](https://dextralabs.com/blog/ai-agentic-workflow-patterns-for-enterprises/)

### Context Management
- [MCP & Multi-Agent AI: Building Collaborative Intelligence 2026](https://onereach.ai/blog/mcp-multi-agent-ai-collaborative-intelligence/)
- [Architecting efficient context-aware multi-agent framework for production](https://developers.googleblog.com/architecting-efficient-context-aware-multi-agent-framework-for-production/)
- [Advancing Multi-Agent Systems Through Model Context Protocol](https://arxiv.org/html/2504.21030v1)
- [Claude Code Just Cut MCP Context Bloat by 46.9%](https://medium.com/@joe.njenga/claude-code-just-cut-mcp-context-bloat-by-46-9-51k-tokens-down-to-8-5k-with-new-tool-search-ddf9e905f734)

### LangGraph
- [LangGraph: Agent Orchestration Framework for Reliable AI Agents](https://www.langchain.com/langgraph)
- [Building Agentic Workflows with LangGraph and Granite](https://www.ibm.com/think/tutorials/build-agentic-workflows-langgraph-granite)
- [GitHub - langchain-ai/langgraph](https://github.com/langchain-ai/langgraph)

### CrewAI
- [CrewAI Framework 2025: Complete Review](https://latenode.com/blog/ai-frameworks-technical-infrastructure/crewai-framework/crewai-framework-2025-complete-review-of-the-open-source-multi-agent-ai-platform)
- [GitHub - crewAIInc/crewAI](https://github.com/crewAIInc/crewAI)
- [Top 7 Agentic AI Frameworks in 2026: LangChain, CrewAI, and Beyond](https://www.alphamatch.ai/blog/top-agentic-ai-frameworks-2026)

### AutoGen / Microsoft Agent Framework
- [AutoGen v0.4: Reimagining the foundation of agentic AI](https://www.microsoft.com/en-us/research/articles/autogen-v0-4-reimagining-the-foundation-of-agentic-ai-for-scale-extensibility-and-robustness/)
- [Semantic Kernel + AutoGen = Microsoft Agent Framework](https://visualstudiomagazine.com/articles/2025/10/01/semantic-kernel-autogen--open-source-microsoft-agent-framework.aspx)
- [GitHub - microsoft/autogen](https://github.com/microsoft/autogen)

### State Management
- [Developer's guide to multi-agent patterns in ADK](https://developers.googleblog.com/developers-guide-to-multi-agent-patterns-in-adk/)
- [Four Design Patterns for Event-Driven, Multi-Agent Systems](https://www.confluent.io/blog/event-driven-multi-agent-systems/)
- [Beyond Agent-to-Agent: Mastering Central State Management](https://medium.com/@rlinlen/beyond-agent-to-agent-mastering-central-state-management-in-multi-agent-systems-with-strands-8b3e0f665902)
- [Multi-Agent Systems: Architecture, Patterns, and Production Design](https://www.comet.com/site/blog/multi-agent-systems/)

### Rollback and Iteration
- [20 Agentic AI Workflow Patterns That Actually Work in 2025](https://skywork.ai/blog/agentic-ai-examples-workflow-patterns-2025/)
- [10 Things Developers Want from their Agentic IDEs in 2025](https://redmonk.com/kholterhoff/2025/12/22/10-things-developers-want-from-their-agentic-ides-in-2025/)
- [Practical Agentic Engineering Workflow: Production Guide 2025](https://www.digitalapplied.com/blog/practical-agentic-engineering-workflow-2025)

### Spec-Driven Development
- [Spec-driven development: Unpacking one of 2025's key new AI-assisted engineering practices](https://www.thoughtworks.com/en-us/insights/blog/agile-engineering-practices/spec-driven-development-unpacking-2025-new-engineering-practices)
- [AGENTS.md](https://agents.md/)
- [Introducing Agent-Flavored Markdown (AFM)](https://wso2.com/library/blogs/introducing-agent-flavored-markdown)
- [Chapter 5: Spec-Driven Development with Claude Code](https://agentfactory.panaversity.org/docs/General-Agents-Foundations/spec-driven-development)

### Reflection Pattern
- [Agentic Design Patterns Part 2: Reflection](https://www.deeplearning.ai/the-batch/agentic-design-patterns-part-2-reflection/)
- [Reflective Agentic AI: Workflows That Outperform GPT-4](https://www.landbase.com/blog/how-reflective-agentic-ai-can-outperform-gpt-4-a-deep-dive-into-new-ai-workflows)
- [7 Must-Know Agentic AI Design Patterns](https://machinelearningmastery.com/7-must-know-agentic-ai-design-patterns/)

### Human-in-the-Loop
- [Human-in-the-Loop for AI Agents: Best Practices, Frameworks, Use Cases](https://www.permit.io/blog/human-in-the-loop-for-ai-agents-best-practices-frameworks-use-cases-and-demo)
- [Microsoft Agent Framework Workflows Orchestrations - HITL](https://learn.microsoft.com/en-us/agent-framework/user-guide/workflows/orchestrations/human-in-the-loop)
- [Human-in-the-Loop (HitL) Agentic AI for High-Stakes Oversight 2026](https://onereach.ai/blog/human-in-the-loop-agentic-ai-systems/)

### Agent Memory and RAG
- [Best Vector Databases for RAG: Complete 2025 Comparison Guide](https://latenode.com/blog/ai-frameworks-technical-infrastructure/vector-databases-embeddings/best-vector-databases-for-rag-complete-2025-comparison-guide)
- [RAG → Agentic RAG → Agent Memory: Smarter Retrieval, Persistent Memory](https://yugensys.com/2025/11/19/evolution-of-rag-agentic-rag-and-agent-memory/)
- [AI Memory: How RAG-Powered Memory Systems Will Transform Enterprise AI in 2025](https://ragwalla.com/blog/the-ai-memory-revolution-how-rag-powered-memory-systems-will-transform-enterprise-ai-in-2025)

### Observability
- [AI Agent Observability - Evolving Standards and Best Practices](https://opentelemetry.io/blog/2025/ai-agent-observability/)
- [Agentic AI in Observability: Building Resilient, Accountable IT Systems](https://thenewstack.io/agentic-ai-in-observability-building-resilient-accountable-it-systems/)
- [Six observability predictions for 2026](https://www.dynatrace.com/news/blog/six-observability-predictions-for-2026/)

### Anti-Patterns
- [AI Agentic Patterns and Anti-Patterns](https://glaforge.dev/talks/2025/12/02/ai-agentic-patterns-and-anti-patterns/)
- [Agent Instruction Patterns and Antipatterns](https://elements.cloud/blog/agent-instruction-patterns-and-antipatterns-how-to-build-smarter-agents/)
- [Agent design patterns](https://rlancemartin.github.io/2026/01/09/agent_design/)

### Tool Calling
- [How Tools Are Called in AI Agents: Complete 2025 Guide](https://medium.com/@sayalisureshkumbhar/how-tools-are-called-in-ai-agents-complete-2025-guide-with-examples-42dcdfe6ba38)
- [Function calling using LLMs](https://martinfowler.com/articles/function-call-LLM.html)
- [Unified Tool Integration for LLMs](https://arxiv.org/html/2508.02979v1)

### YAML Frontmatter
- [Agent Definition Guide: Creating Custom Agents](https://forgecode.dev/docs/agent-definition-guide/)
- [Claude Agent Skills: A First Principles Deep Dive](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/)
- [Specification - Agent Skills](https://agentskills.io/specification)

---
*Stack research for: Meta-Prompting / Agentic Workflow Orchestration for get-shit-done*
*Researched: 2026-02-16*
