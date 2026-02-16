# Coding Best Practices Research Template

Template for `.planning/research/BEST-PRACTICES.md` — coding standards, patterns, and best practices for the project domain and stack.

<template>

```markdown
# Coding Best Practices Research

**Domain:** [domain type]
**Stack:** [primary technologies from project context]
**Researched:** [date]
**Confidence:** [HIGH/MEDIUM/LOW]

## Language & Framework Conventions

### [Primary Language] Best Practices

| Practice | Why It Matters | Example |
|----------|---------------|---------|
| [convention] | [rationale] | [brief code pattern or reference] |
| [convention] | [rationale] | [brief code pattern or reference] |
| [convention] | [rationale] | [brief code pattern or reference] |

### [Primary Framework] Best Practices

| Practice | Why It Matters | Anti-pattern to Avoid |
|----------|---------------|----------------------|
| [practice] | [rationale] | [what NOT to do] |
| [practice] | [rationale] | [what NOT to do] |
| [practice] | [rationale] | [what NOT to do] |

## Code Safety & Defensive Programming

Practices that prevent bugs at compile time or catch them early at runtime.

### Input Validation & Boundary Checks

- [Practice]: [when and how to apply]
- [Practice]: [when and how to apply]

### Error Handling Patterns

- [Pattern]: [when to use, with rationale]
- [Pattern]: [when to use, with rationale]

### Resource Management

- [Practice]: [how to handle connections, memory, file handles, etc.]
- [Practice]: [how to handle connections, memory, file handles, etc.]

## Code Organization & Structure

### File & Module Structure

| Principle | Application | Rationale |
|-----------|-------------|-----------|
| [principle] | [how to apply in this stack] | [why it matters] |
| [principle] | [how to apply in this stack] | [why it matters] |

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| [files] | [convention] | [example] |
| [functions] | [convention] | [example] |
| [variables] | [convention] | [example] |
| [types/interfaces] | [convention] | [example] |
| [constants] | [convention] | [example] |

### Dependency Management

- [Practice]: [rationale]
- [Practice]: [rationale]

## Testing Best Practices

### Testing Strategy

| Layer | Tool | What to Test | Coverage Target |
|-------|------|-------------|-----------------|
| Unit | [tool] | [scope] | [target] |
| Integration | [tool] | [scope] | [target] |
| E2E | [tool] | [scope] | [target] |

### Test Writing Standards

- [Practice]: [rationale and example]
- [Practice]: [rationale and example]
- [Practice]: [rationale and example]

### What NOT to Test

- [Anti-pattern]: [why it's a waste]
- [Anti-pattern]: [why it's a waste]

## Security Best Practices

Domain-specific secure coding patterns beyond general OWASP guidance.

### Authentication & Authorization

- [Practice]: [how to implement correctly]
- [Practice]: [how to implement correctly]

### Data Handling

- [Practice]: [rationale]
- [Practice]: [rationale]

### Secret Management

- [Practice]: [rationale]
- [Practice]: [rationale]

## Performance Best Practices

Coding patterns that prevent performance issues from the start.

### Data Access Patterns

- [Practice]: [rationale — e.g., N+1 prevention, connection pooling]
- [Practice]: [rationale]

### Rendering / Response Patterns

- [Practice]: [rationale — e.g., lazy loading, pagination, caching]
- [Practice]: [rationale]

### What to Avoid

| Anti-pattern | Why It's Slow | Better Approach |
|-------------|---------------|-----------------|
| [pattern] | [impact] | [alternative] |
| [pattern] | [impact] | [alternative] |

## Code Review Checklist

Quick-reference checklist for verifying code quality during implementation.

- [ ] **[Check]:** [what to verify]
- [ ] **[Check]:** [what to verify]
- [ ] **[Check]:** [what to verify]
- [ ] **[Check]:** [what to verify]
- [ ] **[Check]:** [what to verify]
- [ ] **[Check]:** [what to verify]

## Linting & Formatting Standards

| Tool | Purpose | Key Rules |
|------|---------|-----------|
| [linter] | [what it enforces] | [notable config] |
| [formatter] | [what it enforces] | [notable config] |
| [type checker] | [what it enforces] | [notable config] |

## Sources

- [Official style guides referenced]
- [Community best practice docs]
- [Authoritative blog posts or books]
- [Linter/formatter documentation]

---
*Coding best practices research for: [domain]*
*Researched: [date]*
```

</template>

<guidelines>

**Language & Framework Conventions:**
- Focus on the SPECIFIC stack for this project, not generic advice
- Cite official style guides where they exist (e.g., Airbnb JS, PEP 8, Go Effective Go)
- Include anti-patterns — knowing what NOT to do is as valuable as knowing what to do

**Code Safety:**
- Reference https://web.eecs.umich.edu/~imarkov/10rules.pdf principles where applicable
- Focus on practices that prevent bugs at write time, not just catch them at test time
- Include resource management patterns specific to the stack (e.g., Go defer, Python context managers, JS/TS AbortController)

**Testing:**
- Be prescriptive about WHICH testing tools to use with this stack
- Include coverage targets appropriate for the domain (critical path vs. utility code)
- Call out what's NOT worth testing to prevent over-engineering

**Security:**
- Go beyond OWASP basics — focus on domain-specific security patterns
- Include patterns for the specific stack (e.g., SQL injection prevention in the chosen ORM)
- Address secret management for the deployment target

**Performance:**
- Focus on patterns that prevent problems vs. premature optimization
- Include data access patterns specific to the chosen database/ORM
- Call out known performance gotchas in the chosen framework

**Code Review Checklist:**
- Keep actionable and specific to this stack
- This checklist gets used during execution phases
- Each item should be verifiable (yes/no), not subjective

**Linting & Formatting:**
- Recommend specific tools and configs for the stack
- Opinionated > configurable — pick a standard and stick with it
- Include pre-commit hook recommendations

</guidelines>
