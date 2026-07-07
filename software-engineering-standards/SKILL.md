---
name: software-engineering-standards
description: Universal software engineering best practices, architecture principles, and quality standards applicable to any project regardless of stack or domain. Use when starting a new project, reviewing code, setting up CI/CD, designing architecture, writing tests, implementing security, creating design systems, managing documentation, or establishing team conventions. Distilled from enterprise-grade production systems.
---

# Software Engineering Standards

Universal best practices for building production-grade software. These principles are stack-agnostic and apply to any project.

## 12 Architecture Principles

1. **State machine first** — Model every stateful entity (users, orders, deployments, agents) as an explicit state machine with defined transitions. No implicit state.
2. **Events over polling** — Use typed events for inter-component communication. Persist events for audit trails and replay. Never poll when you can subscribe.
3. **Rollback-by-default** — Changes are discarded unless quality metrics improve. The burden of proof is on the change, not the reviewer.
4. **Recovery before escalation** — Every failure type has an automated recovery recipe. Human escalation occurs only after auto-recovery fails.
5. **Compound knowledge** — When a pattern is used 5+ times, extract it into a reusable module, template, or skill. Track success rates.
6. **Deterministic where possible** — Use fixed logic (lint, format, parse, validate) for predictable steps. Reserve complex reasoning for creative or ambiguous decisions.
7. **Minimal human surface** — After initial requirements, automate everything possible. Target <5 min/week of manual intervention per project.
8. **Graduated quality gates** — Quality escalates through stages: local → package → workspace → merge → deploy → production. Each gate has specific criteria.
9. **Partial success is first-class** — If 3 out of 4 steps succeed, preserve progress and move to review. Never discard partial work.
10. **Policy is executable** — Rules, compliance requirements, and quality thresholds are code, not documents. Enforce them automatically.
11. **Context is tiered** — Maintain context across sessions using a tiered memory architecture (Session → Project → Global).
12. **Continuous evolution** — Systems must continuously improve their own rules and processes using an Evolution Loop (Observe → Analyze → Propose → Apply → Measure).

## Seven-Phase Build Sequence

All feature development follows this strict gated sequence. Do not proceed to the next phase until the current one is validated.

| Phase | Action | Key Principle |
|-------|--------|--------------|
| 1. **Specify** | Define requirements | Capture intent in a living document with acceptance criteria |
| 2. **Design** | Define visual identity | Create component designs with all states documented |
| 3. **Plan** | Outline the build | Task breakdown with complexity estimates and dependencies |
| 4. **Mockup Flow** | Map all screens and states | Define every user interaction and data requirement |
| 5. **Frontend** | Build UI components | Implement with mock data first; validate UX before backend |
| 6. **Backend** | Implement data logic | Derive the backend from the frontend's data requirements |
| 7. **Validate** | Test against spec | Ensure the product meets all Phase 1 requirements |

**Critical rule:** The backend is a function of the frontend's needs. Always build the frontend first with mock data.

## Unified QA Cycle

Every code change runs through 5 atomic steps. All must pass or the change is rejected.

| Step | What | Examples |
|------|------|---------|
| 1. **Static Analysis** | Linters, SAST, secret detection, dependency audits | ESLint, Semgrep, `npm audit`, gitleaks |
| 2. **Unit Tests** | New code covered, existing pass, security tests included | Vitest, Jest, pytest |
| 3. **Integration Tests** | API contracts, auth/authz flows, data integrity | Supertest, httpx |
| 4. **E2E Validation** | User flows, visual regression, accessibility, performance | Playwright, Cypress |
| 5. **Documentation** | API docs, changelogs, screen docs updated | Auto-generated from code + manual review |

### QA Dimensions

| Dimension | Weight | Minimum to Ship |
|-----------|--------|-----------------|
| Correctness | 30% | 90% |
| Security | 25% | 95% |
| Accessibility | 15% | 90% |
| Performance | 15% | 85% |
| Documentation | 15% | 80% |

### QA Intensity Levels

| Level | Trigger | Checks |
|-------|---------|--------|
| 1 Light | CSS or copy change | Static analysis, visual regression, accessibility |
| 2 Standard | New UI component | Level 1 + unit tests + component E2E |
| 3 Full | API or backend change | Level 2 + integration tests + performance |
| 4 Critical | Auth flow modification | Level 3 + full auth E2E + penetration patterns |
| 5 Supply Chain | Dependency update | Level 3 + deep dependency scan + license audit |

## 7-Gate Quality Pipeline

| Gate | Time Budget | What |
|------|-------------|------|
| 1. Instant | <1s | Cached lint daemon on save |
| 2. Local | <30s | Type check + unit tests on changed files |
| 3. Targeted | <2min | Integration tests for affected features |
| 4. Metric | <5min | Quality vector comparison (improved → keep, degraded → rollback) |
| 5. Full | <15min | Complete test suite, max 2 CI rounds |
| 6. Review | Variable | Code review (human or automated) |
| 7. Deploy | 5min | Canary deployment with health monitoring |

## Security Standards

### Three-Layer Model

1. **Mandatory Baseline** — Always-on controls for every project (OWASP Top 10)
2. **Compliance Modules** — Activated by project classification (SOC 2, ISO 27001, PCI DSS, HIPAA)
3. **Continuous Operations** — Automated SAST, DAST, SCA scanning in CI

### Compliance Frameworks

All architecture, infrastructure, and code must be designed to support or directly comply with the following enterprise frameworks:

| Framework | Core Engineering Requirements |
|-----------|-------------------------------|
| **ISO 27001** | Information Security Management System (ISMS). Requires explicit asset inventory, access control matrices, cryptography standards, and incident response playbooks. Every system component must have an assigned owner and classification level. |
| **SOC 2** | Trust Services Criteria (Security, Availability, Processing Integrity, Confidentiality, Privacy). Requires comprehensive audit logging (who did what, when), change management tracking (ADRs, PR approvals), and continuous monitoring/alerting for system anomalies. |
| **PCI DSS** | Payment Card Industry Data Security Standard. (If applicable). Requires strict network segmentation, data encryption at rest and in transit, no storage of sensitive authentication data (SAD), and quarterly vulnerability scans. |

### Security Testing Requirements

| Feature Type | Required Tests |
|-------------|----------------|
| Any Endpoint | Unauthenticated rejection, input validation with malicious payloads |
| Data Listing | IDOR test (user A cannot see user B's data), pagination bounds |
| Data Mutation | Authorization check, input sanitization, idempotency |
| File Upload | Type/size validation, content scanning, path traversal prevention |
| Auth Flow | Rate limiting, session management, token expiration |
| Admin Operation | Role verification, privilege escalation attempt, audit log creation |

A feature is not "done" until its security tests are present and passing.

## Architecture Decision Records (ADRs)

Every significant architectural decision produces a numbered ADR following a 6-phase cycle:

1. **Scope** — Identify the architectural question
2. **Research** — Deep-dive into candidates (3–12 sources depending on impact)
3. **Compare** — Weighted decision matrix (capability, security, ecosystem, DX, cost, lock-in)
4. **Design** — Integration architecture with typed interfaces
5. **Connect** — Link to all previous ADRs
6. **Deliver** — Executive summary + full document

Quality standards: numbered references with URLs, decision matrix tables, typed interfaces, security considerations, cost estimates for infrastructure decisions.

## Data Standards

| Standard | Rule |
|----------|------|
| **Primary keys** | Use time-sortable UUIDs (UUIDv7) for optimal B-tree locality |
| **Timestamps** | Every table includes `createdAt` and `updatedAt` |
| **Foreign keys** | Match the type of the parent primary key |
| **Migrations** | Schema changes require an ADR and a reversible migration |
| **Soft deletes** | Prefer `deletedAt` timestamp over hard deletes for audit trails |

## i18n Standards

| Rule | Detail |
|------|--------|
| String externalization | All user-facing strings in locale JSON files, never hardcoded |
| Framework | Use a mature i18n library with namespace support and interpolation |
| Locale files | Maintain parity across all supported languages |
| Persistence | Store language preference in localStorage or user profile |
| New strings | Add to all locale files simultaneously |
| Pluralization | Use ICU message format or library-native plurals |

## Design System Standards

| Principle | Detail |
|-----------|--------|
| Token-based | All colors, spacing, typography defined as design tokens |
| Dark mode | Support dark/light themes via CSS custom properties |
| Accessibility | WCAG 2.2 AA minimum: 4.5:1 contrast, keyboard nav, ARIA labels, reduced motion |
| Component states | Document: default, hover, active, focus, disabled, loading, error, empty |
| Responsive | Mobile-first with defined breakpoints (mobile, tablet, desktop, wide) |
| Documentation | Every component has a Storybook story with all variants |

## CI/CD Pipeline

Minimum viable pipeline for any project:

| Job | Purpose | Fail = Block |
|-----|---------|-------------|
| Lint | Code style and static analysis | Yes |
| Type Check | Type safety verification | Yes |
| Unit Tests | Core logic verification | Yes |
| Integration Tests | API and service contract verification | Yes |
| Build | Compilation and bundling | Yes |
| E2E Tests | User flow verification | Yes (on staging) |
| Deploy | Automated deployment with health checks | Yes |

## Documentation Standards

| Document | Purpose | Update Frequency |
|----------|---------|-----------------|
| **HANDOFF.md** | Complete session/project state for onboarding | Every significant change |
| **ADRs** | Architecture decisions with rationale | Per decision |
| **CHANGELOG** | User-facing changes per release | Per release |
| **API docs** | Endpoint contracts and examples | Per API change |
| **Screen docs** | Page-level documentation (route, purpose, data flow) | Per page change |

## Reference Files

For deeper guidance on specific topics:

- `references/testing-patterns.md` — Test organization, mocking strategies, E2E patterns, visual regression
- `references/git-conventions.md` — Commit messages, branching strategy, PR workflow, release process
- `references/code-intelligence.md` — AST parsing, dependency graphs, memory architecture, skills ecosystem, and evolution loops
