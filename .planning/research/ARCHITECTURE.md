# Architecture Research

**Domain:** Meta-prompting / Agentic Workflow Frameworks
**Researched:** 2026-02-16
**Confidence:** HIGH

## Standard Architecture

### System Overview

Modern agentic workflow systems separate concerns across distinct layers:

```
┌─────────────────────────────────────────────────────────────┐
│                      Command Layer                           │
│  (User interface: CLI commands, workflow invocation)         │
├─────────────────────────────────────────────────────────────┤
│                   Orchestration Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Coordinator  │  │ State Machine│  │   Router     │      │
│  │  (Workflow   │  │ (Transitions)│  │ (Task Dispatch)     │
│  │   Control)   │  │              │  │              │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                 │               │
├─────────┴─────────────────┴─────────────────┴───────────────┤
│                      Agent Layer                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │Planner  │  │Executor │  │Research │  │Verifier │        │
│  │ Agent   │  │ Agent   │  │  Agent  │  │  Agent  │        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘        │
│       │            │            │            │              │
├───────┴────────────┴────────────┴────────────┴──────────────┤
│                   Workflow Definitions Layer                 │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Task Breakdown, Dependencies, Compensation Logic    │   │
│  └──────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                      Memory Layer                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │ State    │  │ Context  │  │Episodic  │                  │
│  │ Store    │  │ History  │  │ Memory   │                  │
│  └──────────┘  └──────────┘  └──────────┘                  │
├─────────────────────────────────────────────────────────────┤
│                      Tools Layer                             │
│  (Filesystem, Git, External APIs, Code Execution)           │
└─────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| **Command Layer** | Entry point, argument parsing, workflow selection | CLI handlers that parse user intent and route to orchestrators |
| **Coordinator** | Breaks down requests into tasks, delegates to agents | Central controller managing workflow execution (orchestration pattern) |
| **State Machine** | Tracks phase transitions, manages state lifecycle | Finite state machine with defined states, transitions, and actions |
| **Router** | Dynamic task dispatch based on context | Pattern matching on task type, context size, agent availability |
| **Agent Layer** | Specialized task execution (planning, coding, research, verification) | Individual agents with focused prompts and tools |
| **Workflow Definitions** | Declarative task sequences, dependencies, rollback logic | DSL or structured data (YAML/JSON/Markdown) defining steps |
| **State Store** | Persistent state across phases | Files (STATE.md, config.json) or databases tracking current position |
| **Context History** | Accumulates interaction traces for continuity | Append-only log of prompts, responses, tool outputs |
| **Episodic Memory** | Cross-phase learning, pattern recognition | Structured summaries of past phases (SUMMARY.md files) |
| **Tools Layer** | Execution primitives for agents | File I/O, shell commands, git operations, API calls |

## Recommended Project Structure

GSD-compatible structure (zero external dependencies, markdown-based):

```
.claude/
├── get-shit-done/
│   ├── bin/
│   │   └── gsd-tools.cjs         # Centralized atomic operations
│   ├── agents/                    # Agent definitions (markdown)
│   │   ├── gsd-planner.md
│   │   ├── gsd-executor.md
│   │   ├── gsd-researcher.md
│   │   └── gsd-verifier.md
│   ├── workflows/                 # Orchestrator layer
│   │   ├── plan-phase.md
│   │   ├── execute-phase.md
│   │   └── verify-work.md
│   ├── references/                # Shared knowledge
│   │   ├── model-profiles.md
│   │   ├── state-management.md
│   │   └── workflow-patterns.md
│   └── templates/                 # Artifact templates
│       ├── PLAN.md
│       ├── SUMMARY.md
│       └── VERIFICATION.md

.planning/                          # State layer (per-project)
├── config.json                     # Configuration
├── STATE.md                        # Current position, context
├── PROJECT.md                      # Project definition
├── ROADMAP.md                      # Phase definitions
├── phases/                         # Phase artifacts
│   ├── 1-foundation/
│   │   ├── plans/
│   │   │   ├── 1-PLAN.md
│   │   │   └── 1-SUMMARY.md
│   │   └── verification/
│   │       └── VERIFICATION.md
│   └── 2-features/
└── research/                       # Research artifacts
    ├── SUMMARY.md
    ├── STACK.md
    ├── FEATURES.md
    ├── ARCHITECTURE.md
    └── PITFALLS.md
```

### Structure Rationale

- **Separation of system vs. project state**: System files (~/.claude/) are version-controlled in GSD repo; project files (.planning/) are per-project
- **Markdown as DSL**: Human-readable, git-friendly, no parser dependencies
- **Flat agent namespace**: All agents in one directory, loaded by name
- **Workflows as markdown**: Orchestrators are markdown files with embedded tool calls
- **Atomic operations in single JS file**: gsd-tools.cjs centralizes state mutations, git operations, config parsing (no npm dependencies)

## Architectural Patterns

### Pattern 1: Orchestration over Choreography

**What:** Central coordinator manages workflow execution and delegates to specialized agents rather than peer-to-peer agent communication.

**When to use:** Always for GSD-style systems. Provides clear control flow, easier debugging, and avoids cyclic dependencies.

**Trade-offs:**
- Pro: Single point of observability, clear execution trace
- Pro: Explicit error handling and compensation logic
- Pro: Easier to add new agents without updating existing ones
- Con: Orchestrator becomes single point of failure (mitigate with state persistence)

**Example:**
```javascript
// Orchestrator pseudocode
async function executePlannedPhase(phase) {
  const state = await loadState();
  const plan = await loadPlan(phase, state.current_plan);

  // Sequential task execution with state checkpoints
  for (const task of plan.tasks) {
    const result = await spawnAgent('gsd-executor', {
      task: task,
      context: state.accumulated_context
    });

    if (!result.success) {
      // Compensation: rollback changes
      await compensateTask(task);
      await updateState({ status: 'failed', last_error: result.error });
      return;
    }

    // Checkpoint after each task
    await updateState({ completed_tasks: [...state.completed_tasks, task.id] });
  }

  await updateState({ status: 'complete', current_plan: state.current_plan + 1 });
}
```

### Pattern 2: State Machine with Compensation

**What:** Model workflow execution as finite state machine with explicit compensating transactions for rollback.

**When to use:** When phases have multiple steps and failures require partial rollback (not just "start over").

**Trade-offs:**
- Pro: Explicit failure handling, no implicit rollback magic
- Pro: Can resume from checkpoints rather than full restart
- Pro: Clear audit trail of what was undone
- Con: Requires defining compensation logic for each step
- Con: Some changes are non-compensable (e.g., external API calls)

**Example:**
```javascript
// Phase state machine
const PhaseStateMachine = {
  states: {
    PLANNING: {
      actions: ['create_plan'],
      transitions: { success: 'EXECUTING', failure: 'FAILED' },
      compensation: async () => { /* Delete PLAN.md */ }
    },
    EXECUTING: {
      actions: ['execute_tasks'],
      transitions: { success: 'VERIFYING', failure: 'COMPENSATING' },
      compensation: async () => { /* Revert file changes via git */ }
    },
    VERIFYING: {
      actions: ['run_verification'],
      transitions: { success: 'COMPLETE', failure: 'COMPENSATING' },
      compensation: async () => { /* No compensation needed */ }
    },
    COMPENSATING: {
      actions: ['rollback_previous_states'],
      transitions: { success: 'FAILED', failure: 'FAILED' }
    },
    COMPLETE: { terminal: true },
    FAILED: { terminal: true }
  }
};

async function executePhase(phase) {
  let currentState = 'PLANNING';
  const completedStates = [];

  while (!PhaseStateMachine.states[currentState].terminal) {
    const stateConfig = PhaseStateMachine.states[currentState];

    try {
      for (const action of stateConfig.actions) {
        await executeAction(action, phase);
      }
      currentState = stateConfig.transitions.success;
      completedStates.push({ state: currentState, action });
    } catch (error) {
      // Compensate in reverse order
      for (const completed of completedStates.reverse()) {
        await PhaseStateMachine.states[completed.state].compensation();
      }
      currentState = stateConfig.transitions.failure;
    }
  }

  return currentState;
}
```

### Pattern 3: Context Propagation via Append-Only History

**What:** Maintain cumulative context history throughout workflow execution, passed to each agent.

**When to use:** Always. Enables agents to understand prior decisions without explicit cross-agent communication.

**Trade-offs:**
- Pro: Simple to implement (file append or array push)
- Pro: Complete audit trail of execution
- Pro: Agents automatically have "memory" of previous steps
- Con: Context grows over time (mitigate with summarization)
- Con: Requires careful pruning for large workflows

**Example:**
```javascript
// Context structure
const WorkflowContext = {
  phase: 1,
  history: [
    { agent: 'gsd-researcher', output: 'Research complete: Use React + Vite', timestamp: '...' },
    { agent: 'gsd-planner', output: 'Plan created: 3 tasks', timestamp: '...' },
    { agent: 'gsd-executor', output: 'Task 1 complete: Init Vite project', timestamp: '...' }
  ],
  decisions: [
    { decision: 'Use React (not Vue)', rationale: 'Team expertise', locked: true }
  ],
  artifacts: [
    { path: '.planning/phases/1/plans/1-PLAN.md', type: 'plan' },
    { path: 'package.json', type: 'code' }
  ]
};

// Agents receive filtered context
function buildAgentContext(fullContext, agentType) {
  return {
    ...fullContext,
    relevant_history: fullContext.history.filter(h => isRelevantTo(h, agentType)),
    // Summarize if context too large
    summary: fullContext.history.length > 50 ? summarizeHistory(fullContext.history) : null
  };
}
```

### Pattern 4: Dynamic Dimension Dispatch

**What:** Workflow dynamically determines which research dimensions to execute based on project type and user input.

**When to use:** Research phase where not all dimensions (stack, features, architecture, pitfalls) apply to every project.

**Trade-offs:**
- Pro: Avoids wasted work on irrelevant research
- Pro: User can add custom dimensions
- Pro: Extensible without modifying orchestrator
- Con: Requires initial classification step
- Con: Dimension definitions must be standardized

**Example:**
```javascript
// Dynamic dimension dispatch
async function researchPhase(projectDescription) {
  // Step 1: Infer dimensions
  const suggestedDimensions = await inferDimensions(projectDescription);
  // Returns: ['stack', 'architecture', 'security'] (skipped 'features' and 'pitfalls')

  // Step 2: User edits dimensions
  const dimensions = await promptUserForDimensions(suggestedDimensions);
  // User adds: 'compliance' dimension

  // Step 3: Spawn researcher per dimension
  const results = await Promise.all(
    dimensions.map(dim => spawnResearcher(dim, projectDescription))
  );

  // Step 4: Synthesize findings
  return await synthesizeResearch(results);
}

// Dimension definitions (extensible)
const DIMENSION_REGISTRY = {
  'stack': {
    template: 'templates/research-project/STACK.md',
    agent: 'gsd-project-researcher',
    prompt: 'Research technology stack for: {description}'
  },
  'architecture': {
    template: 'templates/research-project/ARCHITECTURE.md',
    agent: 'gsd-project-researcher',
    prompt: 'Research architectural patterns for: {description}'
  },
  // Custom dimension added by user
  'compliance': {
    template: 'templates/research-project/COMPLIANCE.md',
    agent: 'gsd-project-researcher',
    prompt: 'Research compliance requirements for: {description}'
  }
};
```

## Data Flow

### Request Flow

```
User Command (/gsd:plan-phase 1)
    ↓
Orchestrator (workflows/plan-phase.md)
    ↓
State Load (gsd-tools.cjs state load)
    ↓
Agent Spawn (Task tool → gsd-planner agent)
    ↓
Agent Execution (Read context, write PLAN.md)
    ↓
State Update (gsd-tools.cjs state update)
    ↓
Git Commit (gsd-tools.cjs commit)
    ↓
Response to User
```

### Phase Execution Data Flow

```
STATE.md (current phase, plan counter, context)
    ↓
Orchestrator reads current position
    ↓
Spawns Agent with Context
    ↓ (agent prompt includes)
Project Context (@references from STATE.md)
Prior Phase Summaries (.planning/phases/*/SUMMARY.md)
Current Plan (.planning/phases/{N}/plans/{M}-PLAN.md)
    ↓
Agent produces artifacts (code, SUMMARY.md)
    ↓
Artifacts written to disk
    ↓
STATE.md updated (plan counter++, context accumulation)
    ↓
Context available to next agent/phase
```

### Cross-Phase Context Propagation

```
Phase 1 Complete → SUMMARY.md written
    ↓
STATE.md updated:
  - Accumulated context += "Phase 1: Foundation complete"
  - Decisions += Phase 1 decisions
    ↓
Phase 2 Planning starts
    ↓
Planner receives:
  - STATE.md context (includes Phase 1 summary)
  - Direct @-reference to Phase 1 SUMMARY.md
  - Project-level decisions from PROJECT.md
    ↓
Planner creates Phase 2 plan building on Phase 1 work
```

### State Management Data Flow

All state mutations go through gsd-tools.cjs to maintain consistency:

```
┌─────────────────────────────────────────┐
│         State Mutation Sources          │
│  (Orchestrators, Agents via Tool calls) │
└──────────────────┬──────────────────────┘
                   ↓
         ┌─────────────────────┐
         │   gsd-tools.cjs     │
         │  (Atomic Operations)│
         └──────────┬───────────┘
                    ↓
    ┌───────────────┴────────────────┐
    ↓                                ↓
.planning/STATE.md            .planning/config.json
.planning/ROADMAP.md          Git commits
```

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 1-5 phases | Simple linear pipeline (plan → execute → verify) is sufficient |
| 5-20 phases | Add episodic memory (phase summaries), cross-phase context propagation |
| 20+ phases | Implement phase grouping (milestones), context summarization, selective history |

### Scaling Priorities

1. **First bottleneck:** Context window exhaustion
   - **Fix:** Summarization of old phases, selective @-references instead of full history

2. **Second bottleneck:** Orchestrator complexity
   - **Fix:** Extract sub-workflows, introduce workflow composition

## Anti-Patterns

### Anti-Pattern 1: Monolithic Orchestrator

**What people do:** Put all workflow logic in one massive orchestrator function with nested conditionals.

**Why it's wrong:** Becomes unmaintainable, hard to test, impossible to extend without modifying core logic.

**Do this instead:** Compose workflows from smaller reusable sub-workflows. Extract phase-specific logic into dedicated orchestrators.

```javascript
// Bad
async function megaOrchestrator(command) {
  if (command === 'new-project') {
    // 500 lines of project initialization
  } else if (command === 'plan-phase') {
    // 300 lines of planning logic
  } else if (command === 'execute-phase') {
    // 400 lines of execution logic
  }
  // ... 2000 more lines
}

// Good
async function newProjectOrchestrator() { /* focused logic */ }
async function planPhaseOrchestrator() { /* focused logic */ }
async function executePhaseOrchestrator() { /* focused logic */ }

const WORKFLOW_REGISTRY = {
  'new-project': newProjectOrchestrator,
  'plan-phase': planPhaseOrchestrator,
  'execute-phase': executePhaseOrchestrator
};
```

### Anti-Pattern 2: Implicit State Transitions

**What people do:** Update STATE.md directly from agents without going through centralized state management.

**Why it's wrong:** Race conditions, inconsistent state, no audit trail, hard to debug.

**Do this instead:** All state mutations through gsd-tools.cjs atomic operations. Agents write artifacts, orchestrator updates state.

```javascript
// Bad (agent directly modifying state)
// In gsd-executor.md
await fs.writeFile('.planning/STATE.md', newState);

// Good (agent writes artifact, orchestrator updates state)
// In gsd-executor.md
await fs.writeFile('.planning/phases/1/plans/1-SUMMARY.md', summary);
// Return to orchestrator, which then:
await bash('node gsd-tools.cjs state update current_plan 2');
```

### Anti-Pattern 3: Tight Coupling Between Agents

**What people do:** Agent A directly calls Agent B, passing custom data structures.

**Why it's wrong:** Creates dependency graph complexity, prevents parallel execution, hard to replace agents.

**Do this instead:** All inter-agent communication via orchestrator and shared state (STATE.md, artifacts on disk).

```javascript
// Bad (agents coupled)
const plannerResult = await gsdPlanner.createPlan(phase);
const executorResult = await gsdExecutor.execute(plannerResult.tasks);

// Good (orchestrator mediates)
await spawnAgent('gsd-planner', { phase });
// Planner writes PLAN.md to disk
const plan = await readPlan(phase);
await spawnAgent('gsd-executor', { phase, plan_path: plan.path });
// Executor reads PLAN.md independently
```

### Anti-Pattern 4: No Compensation Logic

**What people do:** Assume all operations succeed, no rollback on failure.

**Why it's wrong:** Leaves system in inconsistent state (partial work committed, state says incomplete).

**Do this instead:** Define compensation actions for each state. Use git for code rollback, structured state for tracking what to undo.

```javascript
// Bad (no rollback)
await executeTask1(); // Succeeds
await executeTask2(); // Fails
// System now has Task 1 changes but workflow marked as failed

// Good (with compensation)
const completedTasks = [];
try {
  await executeTask1();
  completedTasks.push({ id: 1, compensation: () => revertTask1() });

  await executeTask2();
  completedTasks.push({ id: 2, compensation: () => revertTask2() });
} catch (error) {
  // Compensate in reverse order
  for (const task of completedTasks.reverse()) {
    await task.compensation();
  }
  throw error;
}
```

## Integration Points

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Command → Orchestrator | Direct function call | Orchestrators are imported as functions |
| Orchestrator → Agent | Task tool (spawns agent) | Agents run in isolated context |
| Agent → Tools | Tool calls (Read, Write, Bash) | Agents have limited tool access |
| Agent → State | Via gsd-tools.cjs CLI | Agents never directly modify state |
| Orchestrator → State | Via gsd-tools.cjs CLI | Atomic operations, transactional |

### Extensibility Boundaries

For adding new capabilities without breaking existing architecture:

**New Agent:**
- Create markdown file in `agents/`
- Add to MODEL_PROFILES table in gsd-tools.cjs
- Spawn via existing Task tool from orchestrators

**New Workflow:**
- Create markdown file in `workflows/`
- Add command in main CLI routing
- Use existing agents via Task tool

**New Dimension (research):**
- Add dimension definition to registry
- Create template in `templates/research-project/`
- Dimension automatically available to research orchestrator

**New State Fields:**
- Add field to STATE.md schema
- Add getter/setter in gsd-tools.cjs state operations
- No changes to existing code

## Build Order for GSD Extensions

Based on component dependencies, implement in this order:

### Phase 1: Foundation (No Dependencies)
1. **Extend gsd-tools.cjs state operations**
   - Add dimension registry CRUD
   - Add phase rollback state tracking
   - Add iteration metadata

2. **Create dimension templates**
   - Standard dimensions (stack, features, architecture, pitfalls)
   - Custom dimension template structure

### Phase 2: Dynamic Dispatch (Depends on Phase 1)
3. **Dimension inference logic**
   - Project type classifier
   - Dimension recommendation engine
   - User editing interface

4. **Updated research orchestrator**
   - Load dimensions from registry (not hardcoded)
   - Spawn researchers dynamically
   - Handle custom dimensions

### Phase 3: State Machine Extensions (Depends on Phase 1)
5. **Phase state machine**
   - Define states, transitions, actions
   - Compensation logic per state
   - Checkpoint/resume mechanism

6. **Rollback operations**
   - Git-based code rollback
   - State snapshot/restore
   - Artifact cleanup

### Phase 4: Cross-Phase Context (Depends on Phase 2, 3)
7. **Context accumulation**
   - Phase summary aggregation
   - Decision tracking across phases
   - Pattern detection from history

8. **Context propagation**
   - Filtered context builders per agent type
   - Summarization for large contexts
   - Selective @-reference system

### Phase 5: Workflow Composition (Depends on all)
9. **Sub-workflow extraction**
   - Reusable workflow modules
   - Workflow registry and loader
   - Composition primitives

10. **Custom workflows**
    - User-defined workflow DSL
    - Workflow validation
    - Integration with existing orchestrators

## Compatibility with Existing GSD Architecture

### Preserved Patterns
- ✅ Markdown-based agents (no changes to agent format)
- ✅ Task tool for agent spawning (existing tool, no modifications)
- ✅ gsd-tools.cjs for atomic operations (extend, don't replace)
- ✅ STATE.md + config.json state management (add fields, keep existing)
- ✅ .planning/ directory structure (add subdirectories, keep existing files)
- ✅ Git integration via gsd-tools.cjs commit (no changes)
- ✅ Zero external dependencies (Node.js built-ins only)

### Extension Points (Non-Breaking)
- 📝 **Dimension registry**: New data structure in gsd-tools.cjs, doesn't affect existing code
- 📝 **Phase state machine**: New workflow orchestrator, existing orchestrators unchanged
- 📝 **Context propagation**: Enhanced agent prompts, backward compatible with current prompts
- 📝 **Workflow composition**: New abstraction layer, existing workflows still callable directly

### Migration Strategy
1. **Add new features in parallel** (don't modify existing)
2. **Gradual opt-in** (features disabled by default, enable via config.json)
3. **Fallback to current behavior** (if new features fail, use existing pipeline)
4. **Incremental testing** (validate each component against existing workflows)

## Sources

### Agentic Workflow Architecture
- [Vellum: Agentic Workflows in 2026](https://www.vellum.ai/blog/agentic-workflows-emerging-architectures-and-design-patterns)
- [Stack AI: The 2026 Guide to Agentic Workflow Architectures](https://www.stack-ai.com/blog/the-2026-guide-to-agentic-workflow-architectures)
- [AWS Prescriptive Guidance: Agentic AI Patterns](https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/introduction.html)
- [Google Cloud: Choose a Design Pattern for Agentic AI](https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system)

### Multi-Agent Orchestration
- [Microsoft: Multi-agent Reference Architecture](https://microsoft.github.io/multi-agent-reference-architecture/docs/context-engineering/Agents-Orchestration.html)
- [Redis: Top AI Agent Orchestration Platforms in 2026](https://redis.io/blog/ai-agent-orchestration-platforms/)
- [OneReach: MCP & Multi-Agent AI 2026](https://onereach.ai/blog/mcp-multi-agent-ai-collaborative-intelligence/)

### State Machines and Rollback
- [Microsoft Azure: Saga Design Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/saga)
- [Dapr: Workflow Patterns](https://docs.dapr.io/developing-applications/building-blocks/workflow/workflow-patterns/)
- [Microservices.io: Saga Pattern](https://microservices.io/patterns/data/saga.html)
- [arXiv: StateFlow - State-Driven Workflows for LLMs](https://arxiv.org/html/2403.11322v5)

### Workflow DSLs
- [Serverless Workflow DSL Specification](https://github.com/serverlessworkflow/specification/blob/main/dsl.md)
- [AWS Step Functions: State Machines](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-statemachines.html)

---
*Architecture research for: GSD Framework Extensions (Selectable Research Dimensions, Phase State Machines, Cross-Agent Context)*
*Researched: 2026-02-16*
