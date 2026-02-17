---
name: devops-deployment
short_description: Deployment pipelines and infrastructure patterns
tags: [deployment, ci-cd, infrastructure, monitoring, hosting]
suggested_project_types: [web-app, api, service]
---

Research deployment pipelines and infrastructure patterns for this project's domain.

Investigate:
- Deployment platform options appropriate for this project type: managed platforms
  (Vercel, Fly.io, Railway, Render), container orchestration (ECS, GKE, EKS), and
  serverless options — with honest tradeoffs for each at small and large scale
- CI/CD pipeline patterns: what a minimal but production-quality pipeline looks like for
  this stack, which pipeline tools are standard, and what stages are required vs. optional
- Environment management: how to structure dev, staging, and production environments;
  what must be identical across environments and what legitimately differs
- Container vs. serverless tradeoffs for this project type: when containers are worth the
  operational overhead and when serverless is the better default choice
- Infrastructure-as-code patterns: which IaC tools are standard for this stack and what
  the minimum IaC investment is for a maintainable deployment
- Monitoring and alerting: what to instrument for this type of application, which metrics
  indicate real problems vs. noise, and what observability tooling is standard
- Zero-downtime deployment strategies applicable to this domain: blue-green, rolling
  updates, canary releases — when each is worth the complexity
- Secret and configuration management in deployment: how to pass environment-specific
  configuration safely in each deployment model

Produce deployment guidance that scales from initial launch to production hardening.
Distinguish between "launch minimum" and "production best practice." Cite official
documentation for recommended platforms and tooling.
