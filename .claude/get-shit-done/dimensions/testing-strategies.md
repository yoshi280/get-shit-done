---
name: testing-strategies
short_description: Test approaches and quality assurance patterns
tags: [testing, quality, coverage, automation, ci]
suggested_project_types: [web-app, cli, library, api, mobile, service]
---

Research testing strategies and quality assurance patterns for this project's domain.

Investigate:
- Testing frameworks standard for this stack: which test runners, assertion libraries, and
  mocking tools the community has converged on and why
- Test organization patterns: how to structure test files and directories to match the
  production code layout; naming conventions used in reference projects
- Unit vs. integration vs. end-to-end balance: what proportion of each type is recommended
  for this domain and the rationale (fast feedback vs. confidence tradeoffs)
- Coverage targets: what coverage levels are meaningful for this type of project, which
  coverage metrics matter most (line, branch, path), and what diminishing returns look like
- Mocking and stubbing strategies: what to mock, what not to mock, and the patterns for
  mocking external services, databases, and time-dependent behavior in this stack
- CI pipeline test configuration: how to structure tests for fast feedback in CI, which
  tests run on every commit vs. only on merge, and parallelization patterns
- Test data management: how to create, reset, and maintain test fixtures and seed data for
  this type of project
- Testing edge cases specific to this domain: concurrency, large data volumes, external
  service failures, and security-relevant inputs

Produce testing guidance with concrete examples of test structure. Include recommended
coverage thresholds. Note which testing patterns are essential vs. nice-to-have for launch.
Cite official testing documentation for the recommended frameworks.
