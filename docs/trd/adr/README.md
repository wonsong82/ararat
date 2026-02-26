# Architecture Decision Records (ADR)

This directory contains all architecture decisions made for the Ararat platform. Each ADR documents the context, decision, alternatives considered, and consequences.

## ADR Index

| ADR | Decision | Status | Date | Affects |
|-----|----------|--------|------|---------|
| [001](./001-us-market-only.md) | US Market Only | Accepted | 2025-02-25 | 10-payments, 12-notifications, 06-i18n, 04-auth |
| [002](./002-trilingual-day1.md) | Trilingual Support from Day 1 | Accepted | 2025-02-25 | 06-i18n, 12-notifications, 13-newsletter |
| [003](./003-multi-tenant-architecture.md) | Multi-Tenant Single-Deployment | Accepted | 2025-02-25 | 01-system-architecture, 02-multi-tenancy, 03-data-model |
| [004](./004-face-recognition-on-device.md) | Face Recognition On-Device Only | Accepted | 2025-02-25 | 09-kiosk-face-recognition, 07-registration, 19-security-compliance |
| [005](./005-messenger-agnostic-adapter.md) | Messenger Agnostic Adapter Pattern | Accepted | 2025-02-25 | 12-notifications |
| [006](./006-no-shuttle-van.md) | Shuttle Van Features Removed | Accepted | 2025-02-25 | Overall product scope |
| [007](./007-no-gps-notifications.md) | GPS-Triggered Notifications Removed | Accepted | 2025-02-25 | 08-attendance, 09-kiosk-face-recognition |
| [008](./008-api-first-restful-backend.md) | API-First RESTful Backend | Accepted | 2025-02-25 | 01-system-architecture, 05-api-design |
| [009](./009-stripe-only-payments.md) | Stripe-Only Payments | Accepted | 2025-02-25 | 10-payments |
| [010](./010-trd-living-document.md) | TRD as Living Document | Accepted | 2025-02-25 | All TRD sections, AGENTS.md |

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
