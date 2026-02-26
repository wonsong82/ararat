# Technical Requirements Document: Ararat

**Version**: 2.0  
**Last Updated**: February 25, 2026  
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

| # | File | Covers | Key Entities | Depends On | Phase |
|---|------|--------|-------------|------------|-------|
| 1 | [System Architecture](./01-system-architecture.md) | High-level architecture diagram, service boundaries, tech stack | All services | — | MVP |
| 2 | [Multi-Tenancy](./02-multi-tenancy.md) | Data isolation, tenant context propagation, per-gym settings | Tenant, SystemSetting | 01, 03, 04 | MVP |
| 3 | [Data Model](./03-data-model.md) | ER diagram, all entity definitions with field specs | 30+ entities (Member, Parent, Attendance, Payment, etc.) | 01, 02 | MVP |
| 4 | [Authentication](./04-auth.md) | Parent OTP flow, admin 2FA, JWT tokens, RBAC matrix | User, UserSession | 02, 03, 05 | MVP |
| 5 | [API Design](./05-api-design.md) | REST conventions, response envelope, pagination, error codes, rate limiting, webhooks | — | 04 | MVP |
| 6 | [Internationalization](./06-i18n.md) | Trilingual support (EN/KO/ES), locale files, cultural UX patterns | NotificationTemplate | 12, 13 | MVP |
| 7 | [Registration](./07-registration.md) | Parent/self/admin registration, duplicate detection, COPPA consent, profiles, withdrawal | Member, Parent, ParentChild | 03, 04, 09, 10, 19 | MVP |
| 8 | [Attendance](./08-attendance.md) | Check-in methods, dual-check flow, audit trail, absence alerts | Attendance, AttendanceAudit, AlertRule | 03, 09, 12 | MVP |
| 9 | [Kiosk & Face Recognition](./09-kiosk-face-recognition.md) | On-device face recognition, enrollment, offline mode, re-enrollment, multi-kiosk sync | Kiosk, face embeddings (local) | 07, 08, 19 | Phase 2 |
| 10 | [Payments](./10-payments.md) | Stripe integration, recurring billing, grace period, family billing, refunds, invoices | Payment, Invoice, Membership, MembershipPlan | 02, 03, 07, 11 | MVP |
| 11 | [심사 Belt Promotion](./11-simsa.md) | Exam scheduling, eligibility, registration, result entry, certificates | Simsa, SimsaRegistration, SimsaResult | 03, 10, 12 | MVP |
| 12 | [Notifications](./12-notifications.md) | Multi-channel routing, dispatch flow, alert rules engine, messenger adapter pattern | Notification, NotificationTemplate, AlertRule | 06, 08, 10, 11 | MVP |
| 13 | [Newsletter](./13-newsletter.md) | 가정통신문 editor, audience targeting, delivery, read receipts | Newsletter, NewsletterRecipient | 06, 12, 18 | Phase 2 |
| 14 | [Parent Feed](./14-parent-feed.md) | Training notes, photo sharing, observations, weekly summaries, milestones | ActivityFeedPost, ActivityFeedPhoto, ActivityFeedReaction | 08, 12, 18 | Phase 2 |
| 15 | [Reporting](./15-reporting.md) | Attendance, financial, roster, retention, 심사 reports; dashboard visualizations | — (reads from all entities) | 03, 08, 10, 11 | MVP / Phase 2 |
| 16 | [Admin Tools](./16-admin-tools.md) | Audit log, task system, announcements, gym settings management | AuditLog, Task | 04, 12, 15 | MVP |
| 17 | [Monitor App](./17-monitor-app.md) | TV display: schedule, live attendance, announcements | — (read-only) | 08 | Phase 2 |
| 18 | [File Storage](./18-file-storage.md) | S3 storage, CDN, presigned uploads, retention policies | Files in S3 | 07, 13, 14 | MVP |
| 19 | [Security & Compliance](./19-security-compliance.md) | COPPA, BIPA, encryption, audit trail, data deletion workflow | Consent records, AuditLog | 07, 09 | MVP |
| 20 | [Performance](./20-performance.md) | Response time targets, uptime SLA, scale targets | — | All | All |
| 21 | [Infrastructure](./21-infrastructure.md) | Cloud platform, CI/CD, database, caching, queue, monitoring, backup, scaling | — | All | MVP |

---

## Architecture Decision Records (ADR)

Architectural decisions are documented in [`adr/`](./adr/). See the [ADR Index](./adr/README.md) for the full list.

| ADR | Decision | Affects |
|-----|----------|---------|
| [001](./adr/001-us-market-only.md) | US Market Only | 04, 06, 10, 12 |
| [002](./adr/002-trilingual-day1.md) | Trilingual from Day 1 | 06, 12, 13 |
| [003](./adr/003-multi-tenant-architecture.md) | Multi-Tenant Single-Deployment | 01, 02, 03 |
| [004](./adr/004-face-recognition-on-device.md) | Face Recognition On-Device Only | 07, 09, 19 |
| [005](./adr/005-messenger-agnostic-adapter.md) | Messenger Agnostic Adapter | 12 |
| [006](./adr/006-no-shuttle-van.md) | Shuttle Van Removed | Scope |
| [007](./adr/007-no-gps-notifications.md) | GPS Notifications Removed | 08, 09 |
| [008](./adr/008-api-first-restful-backend.md) | API-First RESTful Backend | 01, 05 |
| [009](./adr/009-stripe-only-payments.md) | Stripe-Only Payments | 10 |
| [010](./adr/010-trd-living-document.md) | TRD as Living Document | All |

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
