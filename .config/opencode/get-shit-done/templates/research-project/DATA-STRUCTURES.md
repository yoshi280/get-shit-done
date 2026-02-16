# Data Structures Research Template

Template for `.planning/research/DATA-STRUCTURES.md` — data structure analysis and recommendations for the project domain and stack.

<template>

```markdown
# Data Structures Research

**Domain:** [domain type]
**Language:** [primary language]
**Stack:** [primary technologies from project context]
**Researched:** [date]
**Confidence:** [HIGH/MEDIUM/LOW]

## Feature-to-Data-Structure Mapping

For each major feature, identify which data structures best support it.

### Feature: [Feature Name]

**What it needs to represent:**
[Description of the data relationships, access patterns, and constraints]

**Recommended data structure:**
[Language-specific type — e.g., Python `dict[str, list[Order]]`, Go `sync.Map`, etc.]

**Why this structure:**
[Rationale — access pattern match, time complexity, memory characteristics]

**Alternatives considered:**

| Structure | Pros | Cons | Why Not Chosen |
|-----------|------|------|----------------|
| [alternative] | [advantages] | [disadvantages] | [reason] |
| [alternative] | [advantages] | [disadvantages] | [reason] |

---

### Feature: [Feature Name]

[Repeat for each major feature...]

---

## Ranked Data Structure Recommendations

Structures ranked by overall utility to this project — how many features they support, how central they are, and how well they fit the access patterns.

| Rank | Structure | Language Type | Features Supported | Access Pattern | Time Complexity (common ops) |
|------|-----------|--------------|-------------------|----------------|------------------------------|
| 1 | [structure] | [e.g., `dict[str, T]`] | [feature list] | [read-heavy / write-heavy / mixed] | [O(1) lookup, O(n) scan, etc.] |
| 2 | [structure] | [e.g., `list[T]`] | [feature list] | [access pattern] | [complexities] |
| 3 | [structure] | [e.g., `set[T]`] | [feature list] | [access pattern] | [complexities] |
| 4 | [structure] | [e.g., `deque[T]`] | [feature list] | [access pattern] | [complexities] |
| 5 | [structure] | [e.g., `heapq`] | [feature list] | [access pattern] | [complexities] |

## Language-Specific Implementations

### Built-in Structures

| Structure | Language Type | When to Use | Gotchas |
|-----------|-------------|-------------|---------|
| [hash map] | [e.g., Python `dict`] | [use case] | [ordering guarantees, thread safety, etc.] |
| [array/list] | [e.g., Python `list`] | [use case] | [resizing behavior, memory layout, etc.] |
| [set] | [e.g., Python `set`] | [use case] | [hashability requirements, etc.] |
| [queue/deque] | [e.g., Python `deque`] | [use case] | [max size, thread safety, etc.] |

### Standard Library Structures

| Structure | Import Path | When to Use | Advantage Over Built-in |
|-----------|------------|-------------|------------------------|
| [structure] | [e.g., `collections.OrderedDict`] | [use case] | [what it adds] |
| [structure] | [e.g., `collections.defaultdict`] | [use case] | [what it adds] |
| [structure] | [e.g., `heapq`] | [use case] | [what it adds] |

### Third-Party / Framework Structures

| Structure | Library | When to Use | Trade-off vs. Built-in |
|-----------|---------|-------------|----------------------|
| [structure] | [library] | [use case] | [what you gain vs. lose] |
| [structure] | [library] | [use case] | [what you gain vs. lose] |

## Data Modeling Patterns

### Relationship Patterns

How to model common relationships in this domain using the language's type system.

| Relationship | Pattern | Implementation | Example |
|-------------|---------|----------------|---------|
| One-to-many | [pattern] | [e.g., `dict[ParentID, list[Child]]`] | [domain example] |
| Many-to-many | [pattern] | [e.g., adjacency list, junction structure] | [domain example] |
| Hierarchical | [pattern] | [e.g., nested dict, tree class] | [domain example] |
| Temporal | [pattern] | [e.g., sorted list with timestamps] | [domain example] |

### Composite Structures

When single structures aren't enough — common combinations for this domain.

#### [Pattern Name]
**Purpose:** [what problem it solves]
**Structure:** [e.g., "dict + sorted list for indexed time-series access"]
**Implementation sketch:**
```
[Brief pseudocode or type definition showing how structures compose]
```
**Used for:** [which features benefit]

---

#### [Pattern Name]
[Repeat for each composite pattern...]

## Persistence Mapping

How in-memory structures map to the persistence layer (database, file, cache).

| In-Memory Structure | Persisted As | Serialization Notes |
|--------------------|--------------|--------------------|
| [e.g., `dict[str, User]`] | [e.g., SQL users table] | [ORM mapping, JSON serialization, etc.] |
| [e.g., `set[Permission]`] | [e.g., enum column or junction table] | [how to round-trip] |
| [e.g., `list[Event]`] | [e.g., append-only log table] | [ordering guarantees] |

## Performance Characteristics

### Memory Profiles

| Structure | Overhead Per Element | When Memory Matters |
|-----------|---------------------|-------------------|
| [structure] | [e.g., ~56 bytes for empty Python dict] | [threshold or dataset size] |
| [structure] | [overhead] | [when to care] |

### Concurrency Considerations

| Structure | Thread-Safe? | Concurrent Alternative | When It Matters |
|-----------|-------------|----------------------|-----------------|
| [structure] | [yes/no/read-only] | [e.g., `queue.Queue`, `concurrent.futures`] | [scenario] |
| [structure] | [yes/no/read-only] | [alternative] | [scenario] |

## Anti-patterns

Data structure choices that seem reasonable but cause problems in this domain.

| Anti-pattern | Why It's Tempting | What Goes Wrong | Better Choice |
|-------------|-------------------|-----------------|---------------|
| [e.g., nested dicts for config] | [easy to write] | [no validation, typo-prone keys] | [dataclass/typed config] |
| [e.g., list for membership checks] | [simple] | [O(n) lookups at scale] | [set or dict] |
| [e.g., global mutable state] | [convenient] | [testing nightmare, race conditions] | [dependency injection] |

## Sources

- [Language documentation for data structures]
- [Performance benchmarks referenced]
- [Domain-specific data modeling resources]
- [Framework documentation for ORM/serialization]

---
*Data structures research for: [domain]*
*Researched: [date]*
```

</template>

<guidelines>

**Feature-to-Data-Structure Mapping:**
- Start from the features, not from the structures — let requirements drive choices
- Consider access patterns: is this read-heavy? Write-heavy? Random access? Sequential?
- Include alternatives considered so the reasoning is transparent

**Ranked Recommendations:**
- Rank by overall utility to the project, not by general popularity
- A structure supporting 4 features ranks higher than one supporting 1, all else equal
- Include time complexity for the operations that matter most to each feature

**Language-Specific Implementations:**
- Use the ACTUAL types from the project's language — `dict[str, list[Order]]` not "hash map"
- Call out gotchas specific to the language's implementation (e.g., Python dict ordering since 3.7, Go map iteration randomness)
- Include standard library structures that are often overlooked (e.g., `collections.Counter`, `bisect`)

**Composite Structures:**
- Real systems rarely use single structures in isolation
- Show how structures compose to solve domain problems
- Keep implementation sketches brief — type signatures, not full code

**Persistence Mapping:**
- Critical for bridging in-memory design with database schema
- Call out serialization round-trip issues early
- Note where ORM choices constrain structure choices

**Anti-patterns:**
- Focus on domain-specific mistakes, not textbook examples
- "Nested dicts instead of typed objects" is more useful than "using arrays for everything"
- Include the migration path from anti-pattern to better choice

</guidelines>
