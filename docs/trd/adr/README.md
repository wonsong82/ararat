# Architecture Decision Records (ADR)

This directory contains all architecture decisions made for the Ararat platform. Each ADR documents the context, decision, alternatives considered, and consequences.

## ADR Index

| ADR | Decision | Status | Date | Affects |
|-----|----------|--------|------|---------|
| [001](./001-us-market-only.md) | US Market Only | Accepted | 2025-02-25 | 11-payments, 13-notifications, 06-i18n, 04-auth |
| [002](./002-trilingual-day1.md) | Trilingual Support from Day 1 | Accepted | 2025-02-25 | 06-i18n, 13-notifications, 14-newsletter |
| [003](./003-multi-tenant-architecture.md) | Multi-Tenant Single-Deployment | Accepted | 2025-02-25 | 01-system-architecture, 02-multi-tenancy, 03-data-model |
| [004](./004-face-recognition-on-device.md) | Face Recognition On-Device Only | Accepted | 2025-02-25 | 08-kiosk-app-architecture, 09-registration, 20-security-compliance |
| [005](./005-messenger-agnostic-adapter.md) | Messenger Agnostic Adapter Pattern | Accepted | 2025-02-25 | 13-notifications |
| [006](./006-no-shuttle-van.md) | Shuttle Van Features Removed | Accepted | 2025-02-25 | Overall product scope |
| [007](./007-no-gps-notifications.md) | GPS-Triggered Notifications Removed | Accepted | 2025-02-25 | 10-attendance, 08-kiosk-app-architecture |
| [008](./008-api-first-restful-backend.md) | API-First RESTful Backend | Accepted | 2025-02-25 | 01-system-architecture, 05-api-design |
| [009](./009-stripe-only-payments.md) | Stripe-Only Payments | Accepted | 2025-02-25 | 11-payments |
| [010](./010-trd-living-document.md) | TRD as Living Document | Accepted | 2025-02-25 | All TRD sections, AGENTS.md |
| [011](./011-nestjs-backend.md) | NestJS Backend Framework | Accepted | 2026-02-25 | 01-system-architecture, 05-api-design, 22-infrastructure |
| [012](./012-react-vite-frontend.md) | React + Vite Frontend | Accepted | 2026-02-25 | 01-system-architecture, 07-web-frontend-architecture, 18-monitor-app |
| [013](./013-aws-cloud-platform.md) | AWS Cloud Platform | Accepted | 2026-02-25 | 01-system-architecture, 19-file-storage, 22-infrastructure |
| [014](./014-github-actions-cicd.md) | GitHub Actions CI/CD | Accepted | 2026-02-25 | 22-infrastructure |
| [015](./015-frontend-library-stack.md) | Frontend Library Stack | Accepted | 2026-02-26 | 01-system-architecture, 07-web-frontend-architecture |
| [016](./016-backend-testing-jest.md) | Backend Testing — Jest | Accepted | 2026-03-01 | 05-api-design, 22-infrastructure |
| [017](./017-frontend-testing-vitest.md) | Frontend Testing — Vitest | Accepted | 2026-03-01 | 07-web-frontend-architecture, 22-infrastructure |
| [018](./018-e2e-testing-playwright.md) | E2E Testing — Playwright | Accepted | 2026-03-01 | 07-web-frontend-architecture, 22-infrastructure |
| [019](./019-api-documentation-swagger.md) | API Documentation — @nestjs/swagger | Accepted | 2026-03-01 | 05-api-design, 22-infrastructure |
| [020](./020-logging-nestjs-pino.md) | Logging — nestjs-pino | Accepted | 2026-03-01 | 01-system-architecture, 22-infrastructure |

## When to Create a New ADR

Create a new ADR when making decisions about:

- Technology choices (frameworks, libraries, services)
- Architecture patterns (data flow, service boundaries)
- Feature scope changes (adding or removing features)
- Compliance strategies (COPPA, BIPA, data handling)
- Integration choices (payment providers, notification channels)

## ADR Format

Each ADR follows this template:

```markdown
# ADR-NNN: Title

**Status**: Proposed | Accepted | Deprecated | Superseded  
**Date**: YYYY-MM-DD  
**Deciders**: [who made this decision]

## Context
[What prompted this decision]

## Decision
[What was decided]

## Alternatives Considered
[What else was on the table]

## Consequences
[Positive and negative impacts]

**Affects**: [list of TRD sections impacted]
```

## Numbering Convention

- ADRs are numbered sequentially: 001, 002, 003, ...
- Never reuse a number — deprecated ADRs keep their number
- If an ADR is superseded, add `Superseded by ADR-NNN` to its status
