---
name: data-structures
short_description: Data models, schemas, and storage patterns
tags: [data, models, schemas, storage, database]
suggested_project_types: [web-app, api, service, mobile]
---

Research data modeling, schema design, and storage patterns for this project's domain.

Investigate:
- Core data models and entities for this domain — what the fundamental objects are, their
  attributes, and the relationships between them
- Schema design patterns: normalization vs. denormalization tradeoffs for this domain's
  read/write patterns, and what the community recommends for this project type
- Storage technology selection: which database types (relational, document, key-value,
  graph, time-series) fit this domain's data access patterns and why
- Indexing patterns: which fields need indexes for the domain's most common queries and
  what the performance impact is at realistic data volumes
- Migration strategies: how to evolve schemas over time without downtime, and which
  migration tooling is standard for this stack
- Data validation: where validation belongs (application layer, database constraints, or
  both) and what validation rules are critical for this domain's integrity
- Caching patterns: what data is worth caching, at what layer, and with what invalidation
  strategy for this type of project

Produce data model recommendations with concrete schema examples. Include the tradeoffs
for key design decisions. Note where schema choices have long-term lock-in implications
that are costly to reverse. Cite official documentation or reference implementations.
