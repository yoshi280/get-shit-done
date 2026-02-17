---
name: architecture
short_description: System design and structural patterns
tags: [architecture, design, structure, organization]
suggested_project_types: [web-app, cli, library, api, mobile, service]
---

Research system architecture and structural patterns for this project's domain.

Investigate:
- Standard architectural patterns used by expert practitioners for this type of project,
  including the components involved, their responsibilities, and how they communicate
- Recommended project structure and directory organization with rationale for each grouping;
  match conventions of the dominant stack for this domain
- Data flow patterns: how data moves from user input through processing to storage and back;
  include both the happy path and error path
- State management approach appropriate for this domain's scale and complexity
- Module boundaries: how to separate concerns so components stay independent and testable
- Scaling considerations: what breaks first as usage grows, and what architectural changes
  address each bottleneck; be realistic about what scale is relevant for this project
- Architectural anti-patterns common in this domain: what people build wrong, why it causes
  problems later, and what to do instead

Produce an architecture guide with enough specificity to make structural decisions. Include
a recommended directory layout. Note where patterns apply only at certain scales. Cite
real projects or frameworks that use the recommended patterns.
