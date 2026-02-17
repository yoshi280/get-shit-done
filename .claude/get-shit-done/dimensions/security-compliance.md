---
name: security-compliance
short_description: Security practices and compliance requirements
tags: [security, auth, compliance, privacy, vulnerabilities]
suggested_project_types: [web-app, api, service, mobile]
---

Research security practices and compliance requirements for this project's domain.

Investigate:
- Authentication and authorization patterns appropriate for this domain: what mechanisms
  experts use, which libraries are standard, and what the common implementation mistakes are
- OWASP Top 10 applicability: which of the standard vulnerabilities are most relevant to
  this project type and the domain-specific manifestations of each
- Input validation and output encoding requirements: what inputs must be validated, how,
  and where in the stack validation should be enforced
- Data privacy requirements relevant to this domain: GDPR, CCPA, or sector-specific
  regulations (HIPAA, PCI-DSS, SOC2) that apply and the minimum compliance requirements
- Secrets management: how credentials, API keys, and sensitive configuration should be
  stored, rotated, and accessed in this type of project
- Dependency vulnerability management: scanning tools and update cadences standard for
  this stack
- Transport security: TLS requirements, certificate management, and HTTPS enforcement
  patterns for this domain
- Security headers, CORS, and CSRF protection patterns appropriate for this project type

Produce actionable security guidance prioritized by risk. Distinguish between "must have
before launch" and "should add before public scale." Cite OWASP documentation, official
security advisories, or framework security guides as sources.
