# Technical Requirements Document: Ararat

**Version**: 3.0  
**Last Updated**: March 1, 2026  
**Status**: Draft

---

## How to Use This Document

The TRD is the **single source of truth** for implementation. During development, reference ONLY the TRD — never the PRD.

**For AI agents and developers:**
1. Start here — read the Section Map to identify which files are relevant to your task
2. Check "Depends On" to find related context you may also need
3. Check "Related ADRs" for architectural constraints that apply
4. Read only the relevant files — do NOT read all TRD files unless doing a cross-cutting change
5. After implementing, update the Implementation Notes in the relevant TRD file(s)

---

## Section Map

### Part 1 — Foundation (01–06)

| # | File | Covers | Key Entities | Depends On | Phase |
|---|------|--------|-------------|------------|-------|
| 1 | [System Architecture](./01-system-architecture.md) | High-level architecture, service boundaries, tech stack, all-app overview | All services | — | MVP |
| 2 | [Multi-Tenancy](./02-multi-tenancy.md) | Data isolation, tenant context propagation, per-gym settings | Tenant, SystemSetting | 01, 03, 04 | MVP |
| 3 | [Data Model](./03-data-model.md) | ER diagram, all entity definitions with field specs | 30+ entities (Member, Parent, Attendance, Payment, etc.) | 01, 02 | MVP |
| 4 | [Authentication](./04-auth.md) | Parent OTP, admin 2FA, JWT tokens, RBAC, frontend auth flows | User, UserSession | 02, 03, 05 | MVP |
| 5 | [API Design](./05-api-design.md) | REST conventions, response envelope, pagination, error codes, rate limiting, webhooks | — | 04 | MVP |
| 6 | [Internationalization](./06-i18n.md) | Trilingual (EN/KO/ES), server templates, client-side i18n, cultural UX | NotificationTemplate | — | MVP |

### Part 2 — App Architecture (07–08)

| # | File | Covers | Key Entities | Depends On | Phase |
|---|------|--------|-------------|------------|-------|
| 7 | [Web Frontend Architecture](./07-web-frontend-architecture.md) | Monorepo structure, shared packages, routing, auth flow, state management, forms, responsive design | Parent App, Admin App, Monitor App | 01, 04, 05, 06 | MVP / Phase 2 |
| 8 | [Kiosk App Architecture](./08-kiosk-app-architecture.md) | Native iOS Swift, MVVM+SwiftUI, face recognition pipeline, local storage, offline mode | Kiosk, face embeddings (local) | 01, 04, 10 | Phase 2 |

### Part 3 — Features (09–18)

| # | File | Covers | Key Entities | Depends On | Phase |
|---|------|--------|-------------|------------|-------|
| 9 | [Registration](./09-registration.md) | Registration flows, duplicate detection, COPPA consent, profiles, withdrawal — all-app UX | Member, Parent, ParentChild | 03, 04, 08, 11, 20 | MVP |
| 10 | [Attendance](./10-attendance.md) | Check-in methods, dual-check flow, audit trail, absence alerts, kiosk UX — all-app UX | Attendance, AttendanceAudit, AlertRule | 03, 08, 13 | MVP |
| 11 | [Payments](./11-payments.md) | Stripe integration, recurring billing, grace period, family billing, refunds — all-app UX | Payment, Invoice, Membership, MembershipPlan | 02, 03, 09, 12 | MVP |
| 12 | [심사 Belt Promotion](./12-simsa.md) | Exam scheduling, eligibility, registration, result entry, certificates — all-app UX | Simsa, SimsaRegistration, SimsaResult | 03, 11, 13 | MVP |
| 13 | [Notifications](./13-notifications.md) | Multi-channel routing, dispatch flow, alert rules, messenger adapter — all-app UX | Notification, NotificationTemplate, AlertRule | 06, 10, 11, 12 | MVP |
| 14 | [Newsletter](./14-newsletter.md) | 가정통신문 editor, audience targeting, delivery, read receipts — all-app UX | Newsletter, NewsletterRecipient | 06, 13, 19 | Phase 2 |
| 15 | [Parent Feed](./15-parent-feed.md) | Training notes, photo sharing, observations, weekly summaries, milestones — all-app UX | ActivityFeedPost, ActivityFeedPhoto, ActivityFeedReaction | 10, 13, 19 | Phase 2 |
| 16 | [Reporting](./16-reporting.md) | Attendance, financial, roster, retention, 심사 reports, dashboard — admin UX | — (reads from all entities) | 03, 10, 11, 12 | MVP / Phase 2 |
| 17 | [Admin Tools](./17-admin-tools.md) | Audit log, task system, gym settings, staff management, class management — admin UX | AuditLog, Task | 04, 13, 16 | MVP |
| 18 | [Monitor App](./18-monitor-app.md) | TV display: schedule, live attendance, announcements, setup/pairing — display UX | — (read-only) | 10 | Phase 2 |

### Part 4 — Infrastructure (19–22)

| # | File | Covers | Key Entities | Depends On | Phase |
|---|------|--------|-------------|------------|-------|
| 19 | [File Storage](./19-file-storage.md) | S3 storage, CDN, presigned uploads, retention policies | Files in S3 | 09, 14, 15 | MVP |
| 20 | [Security & Compliance](./20-security-compliance.md) | COPPA, BIPA, encryption, audit trail, data deletion workflow | Consent records, AuditLog | 08, 09 | MVP |
| 21 | [Performance](./21-performance.md) | Response time targets, uptime SLA, scale targets | — | All | All |
| 22 | [Infrastructure](./22-infrastructure.md) | Cloud platform, CI/CD, database, caching, queue, monitoring, backup, scaling | — | All | MVP |

---

## Architecture Decision Records (ADR)

Architectural decisions are documented in [`adr/`](./adr/). See the [ADR Index](./adr/README.md) for the full list.

| ADR | Decision | Affects |
|-----|----------|---------|
| [001](./adr/001-us-market-only.md) | US Market Only | 04, 06, 11, 13 |
| [002](./adr/002-trilingual-day1.md) | Trilingual from Day 1 | 06, 13, 14 |
| [003](./adr/003-multi-tenant-architecture.md) | Multi-Tenant Single-Deployment | 01, 02, 03 |
| [004](./adr/004-face-recognition-on-device.md) | Face Recognition On-Device Only | 08, 09, 20 |
| [005](./adr/005-messenger-agnostic-adapter.md) | Messenger Agnostic Adapter | 13 |
| [006](./adr/006-no-shuttle-van.md) | Shuttle Van Removed | Scope |
| [007](./adr/007-no-gps-notifications.md) | GPS Notifications Removed | 08, 10 |
| [008](./adr/008-api-first-restful-backend.md) | API-First RESTful Backend | 01, 05 |
| [009](./adr/009-stripe-only-payments.md) | Stripe-Only Payments | 11 |
| [010](./adr/010-trd-living-document.md) | TRD as Living Document | All |
| [011](./adr/011-nestjs-backend.md) | NestJS Backend Framework | 01, 05, 22 |
| [012](./adr/012-react-vite-frontend.md) | React + Vite Frontend | 01, 07, 18 |
| [013](./adr/013-aws-cloud-platform.md) | AWS Cloud Platform | 01, 19, 22 |
| [014](./adr/014-github-actions-cicd.md) | GitHub Actions CI/CD | 22 |
| [015](./adr/015-frontend-library-stack.md) | Frontend Library Stack | 01, 07 |
| [016](./adr/016-backend-testing-jest.md) | Backend Testing — Jest | 05, 22 |
| [017](./adr/017-frontend-testing-vitest.md) | Frontend Testing — Vitest | 07, 22 |
| [018](./adr/018-e2e-testing-playwright.md) | E2E Testing — Playwright | 07, 22 |
| [019](./adr/019-api-documentation-swagger.md) | API Documentation — @nestjs/swagger | 05, 22 |
| [020](./adr/020-logging-nestjs-pino.md) | Logging — nestjs-pino | 01, 22 |

---

## Glossary

| Term | Definition |
|------|-----------|
| **관장님** (Gwanjangnim) | Gym owner/master. Primary admin user. |
| **도장** (Dojang) | Taekwondo gym/training hall. |
| **심사** (Simsa) | Belt promotion examination. |
| **띠** (Tti) | Belt. Represents student rank. |
| **수련생** (Suryeonsaeng) | Student/trainee. |
| **사범** (Sabeom) | Instructor/master instructor. |
| **학부모** (Hakbumo) | Parent/guardian. |
| **탈퇴** (Talhoe) | Membership withdrawal. |
| **가정통신문** | Formal newsletter from gym to parents. |
| **승품·단** | Official Kukkiwon belt promotion certification. |
| **COPPA** | Children's Online Privacy Protection Act (US Federal). |
| **BIPA** | Biometric Information Privacy Act (Illinois). |
| **CCPA/CPRA** | California Consumer Privacy Act / California Privacy Rights Act. |
| **JWT** | JSON Web Token. Stateless authentication token. |
| **OTP** | One-Time Password. SMS code for authentication. |
| **2FA** | Two-Factor Authentication. TOTP code for admin login. |
| **TOTP** | Time-Based One-Time Password. Authenticator app code. |
| **RBAC** | Role-Based Access Control. Permission system based on user role. |
| **RLS** | Row-Level Security. Database-level access control. |
| **FCM** | Firebase Cloud Messaging. Push notification service. |
| **SMS** | Short Message Service. Text message. |
| **API** | Application Programming Interface. |
| **REST** | Representational State Transfer. API design pattern. |
| **S3** | Amazon Simple Storage Service. Cloud storage. |
| **CDN** | Content Delivery Network. Global content distribution. |
| **RTO** | Recovery Time Objective. Max time to restore service. |
| **RPO** | Recovery Point Objective. Max data loss acceptable. |
| **MRR** | Monthly Recurring Revenue. |
| **NPS** | Net Promoter Score. Customer satisfaction metric. |
| **ARPU** | Average Revenue Per User. |

---

**End of TRD Index**
