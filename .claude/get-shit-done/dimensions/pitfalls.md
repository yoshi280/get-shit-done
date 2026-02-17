---
name: pitfalls
short_description: Common mistakes and gotchas to avoid
tags: [pitfalls, mistakes, gotchas, debugging]
suggested_project_types: [web-app, cli, library, api, mobile, service]
---

Research common pitfalls, mistakes, and gotchas in this project's domain.

Investigate:
- Critical domain-specific mistakes that cause rewrites, data loss, or security incidents;
  for each: what goes wrong, why developers make this mistake, how to detect it early,
  and how to prevent it
- Technical debt patterns: shortcuts that seem reasonable during development but create
  compounding problems later; include when each shortcut is "never acceptable" vs.
  "acceptable for MVP only"
- Integration gotchas when connecting to external services or APIs common in this domain
- Performance traps: patterns that work at small scale but fail as usage grows, including
  the approximate threshold where each trap becomes a problem
- Configuration and environment pitfalls that cause "works locally, broken in production"
  failures
- "Looks done but isn't" checklist: features that appear complete in demos but are missing
  critical pieces for production (error states, edge cases, concurrency issues)
- Security mistakes specific to this domain beyond the general OWASP checklist

Produce actionable pitfall documentation organized by severity. Each pitfall should include
warning signs for early detection. Cross-reference which development phases should address
each pitfall. Cite post-mortems, community discussions, or official "gotchas" documentation
where available.
