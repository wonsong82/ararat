# Phase 0 — Foundation

**Status**: Not Started  
**Tasks**: 20 | **Completed**: 0 | **Progress**: 0%

---

> Project scaffolding, core infrastructure, shared frameworks. Must complete before any feature work.

### 0.1 Project Scaffolding & Infrastructure

- [ ] **Monorepo setup** — `docs/trd/01-system-architecture.md` — Initialize project structure, package manager, workspace config, linter, formatter, TypeScript config
- [ ] **Docker Compose environment** — `docs/trd/21-infrastructure.md` — PostgreSQL, Redis, LocalStack (S3) containers, Makefile with `make up`, `make down`, `make logs`, `.env.example`
- [ ] **CI/CD pipeline skeleton** — `docs/trd/21-infrastructure.md` — GitHub Actions for lint, type-check, test on every push; Docker image build on merge to main

### 0.2 Database & Multi-Tenancy

- [ ] **PostgreSQL schema & migrations setup** — `docs/trd/03-data-model.md` — Migration tool, initial schema with Tenant table, UUID primary keys, `tenant_id` column convention
- [ ] **Core entity tables** — `docs/trd/03-data-model.md` — User, Member, Parent, ParentChild, Staff, Belt, Class, ClassSchedule, ClassEnrollment, MemberBelt tables with all field specs
- [ ] **Row-Level Security (RLS) policies** — `docs/trd/02-multi-tenancy.md` — RLS on all tenant-scoped tables, database role per tenant context
- [ ] **Tenant context propagation middleware** — `docs/trd/02-multi-tenancy.md` — Extract `tenantId` from JWT, inject into request context, enforce on all downstream queries
- [ ] **SystemSetting table & seed data** — `docs/trd/02-multi-tenancy.md` — Per-gym configurable settings (belt progression, absence thresholds, grace period, discounts, notification channels, etc.)

### 0.3 Authentication & Authorization

- [ ] **JWT infrastructure** — `docs/trd/04-auth.md` — Token generation, validation middleware, refresh token flow with Redis storage, token expiry (1h access / 30d refresh)
- [ ] **Parent OTP authentication** — `docs/trd/04-auth.md` — Phone number validation (US format), Twilio SMS integration, 6-digit OTP with 10min expiry, 3 retry limit
- [ ] **Admin email + 2FA authentication** — `docs/trd/04-auth.md` — Email/password login, bcrypt hashing, TOTP (authenticator app), account lockout after 5 failed attempts (15min)
- [ ] **RBAC middleware** — `docs/trd/04-auth.md` — Role-based permission checks (Owner, Manager, Instructor, Parent, Member) per the RBAC matrix, route-level guards

### 0.4 API Framework

- [ ] **REST API skeleton** — `docs/trd/05-api-design.md` — NestJS setup, RESTful routing conventions, tenant-scoped URL pattern (`/api/v1/tenants/{tenantId}/...`)
- [ ] **Standard response envelope** — `docs/trd/05-api-design.md` — Success/error response format, cursor-based pagination, error codes and structured error responses
- [ ] **Rate limiting & request validation** — `docs/trd/05-api-design.md` — Per-tenant rate limits (Redis-backed), input validation middleware, request sanitization

### 0.5 Internationalization (i18n)

- [ ] **i18n framework setup** — `docs/trd/06-i18n.md` — Translation key infrastructure, locale files (`en.json`, `ko.json`, `es.json`), language detection from JWT/Accept-Language header
- [ ] **Server-side template rendering** — `docs/trd/06-i18n.md` — Language-aware notification template rendering with personalization tokens (`{{member_name}}`, `{{gym_name}}`, etc.)

### 0.6 File Storage

- [ ] **S3 integration & presigned uploads** — `docs/trd/18-file-storage.md` — Presigned URL generation, direct client upload to S3, upload confirmation endpoint, file metadata storage, size limits
- [ ] **CDN configuration** — `docs/trd/18-file-storage.md` — CloudFront distribution, signed URLs for private content, cache TTL policies (1 day photos, 1 hour certificates)

### 0.7 Testing & Quality

- [ ] **Testing framework setup** — `docs/trd/21-infrastructure.md` — Unit test runner, integration test setup with test database, coverage reporting, test utilities (factories, fixtures)