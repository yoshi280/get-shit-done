---
name: data-structure-analyst
short_description: Recommends optimal data structures for features and components
tags: [analysis]
triggers: [plan-phase]
suggested_project_types: [all]
authority:
  can_create_tasks: true
  can_create_phases: false
---

Analyze data structures for this phase's requirements using the project's primary language.

Investigate and document:

1. **Requirement-to-structure mapping** — For each requirement ID in this phase, recommend the best-fit data structure. Use the language's ACTUAL types (e.g., `dict[str, list[Order]]` not "hash map"). Include access pattern rationale (why this structure fits the requirement's read/write patterns) and operation complexity.
2. **Access pattern analysis** — For each recommended structure, document the specific access patterns this phase requires. Include amortized complexity for insert, lookup, delete, iterate, and filter operations. Flag any O(n^2) or worse patterns that could become bottlenecks.
3. **Shared structure identification** — Identify data structures used by multiple tasks or requirements. Document cross-task coordination risks (concurrent modification, cache invalidation, stale references). Recommend synchronization strategies where needed.
4. **Persistence boundaries** — Map in-memory structures to their persistence layer (database tables, cache keys, file formats, API payloads). Document serialization gotchas: dates, enums, nested objects, circular references, large collections.
5. **Complexity hotspots** — Rank structures by blast radius (how many requirements break if the structure is wrong). Highest-blast-radius structures get the most scrutiny. Include growth projections (what happens when data scales 10x, 100x).
6. **Concrete recommendations** — For each recommendation, provide code examples in the project's language using actual type syntax. Reference REQ-IDs explicitly. Include the alternative structure and when you'd switch to it.

For each recommendation:
- Use the language's actual type syntax, not abstract CS names
- Include operation complexity for the specific access patterns this phase needs
- Show code examples for non-obvious composite patterns
- Note where ORM abstractions help vs hurt for persistence mapping

Quality gates:
- Every phase requirement ID mapped to at least one structure
- Language-specific types used throughout (not abstract names)
- Complexity hotspots ranked by blast radius
- Persistence mapping includes serialization gotchas
- Shared structures have coordination strategy documented
