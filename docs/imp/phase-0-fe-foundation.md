# Phase 0-fe — Foundation: Frontend Scaffolding

**Status**: Not Started  
**Tasks**: 8 | **Completed**: 0 | **Progress**: 0%

---

> Frontend monorepo setup, shared libraries, app shells, and testing infrastructure. Can start in parallel with Phase 0 backend.

### 0.8 Frontend Scaffolding

- [ ] **Frontend monorepo scaffolding** — `docs/trd/07-web-frontend-architecture.md` — pnpm workspaces inside `web/`, three Vite React apps (web/app, web/admin, web/monitor), web/tsconfig.base.json, shared ESLint + Prettier config, path aliases (@ararat/ui, @ararat/api-client, @ararat/shared), pnpm-workspace.yaml scoped to web/ only
- [ ] **Shared UI component library (packages/ui)** — `docs/trd/07-web-frontend-architecture.md` — shadcn/ui setup with Radix primitives, 25+ components (Button, Input, Select, Dialog, Table, Card, Badge, Avatar, Tabs, Toast, etc.), design tokens (CSS custom properties), shared Tailwind preset with font stack (Inter + Pretendard)
- [ ] **Shared API client (packages/api-client)** — `docs/trd/07-web-frontend-architecture.md` — ky HTTP client instance with base URL config, tenantId injection, auth token beforeRequest hook, 401 auto-refresh afterResponse hook with concurrent request queuing, typed ApiError interface, TanStack Query v5 integration with default options (30s staleTime, 5min gcTime, 2 retries)
- [ ] **Shared types & validation schemas (packages/shared)** — `docs/trd/07-web-frontend-architecture.md` — Zod validation schemas (member, attendance, payment, simsa, auth, notification, newsletter, class, settings) for frontend validation only (backend validates independently via class-validator + DTOs), TypeScript types/interfaces, constants (enums, belt levels, roles, error codes)
- [ ] **Client-side i18n integration** — `docs/trd/07-web-frontend-architecture.md` — react-i18next setup, 13 namespace translation files (common, auth, registration, attendance, payments, simsa, notifications, newsletter, feed, reports, settings, monitor, validation) in en/ko/es, date-fns locale-aware formatting (date, time, currency, relative time), language detection from JWT/navigator.language
- [ ] **Parent App shell** — `docs/trd/07-web-frontend-architecture.md` — TanStack Router file-based routing with route tree generation, _auth layout guard (checks Zustand auth store, silent token refresh on load), bottom tab navigation (Home, Children, Payments, Notifications, Settings), mobile-first responsive layout (375px+), React error boundary
- [ ] **Admin App shell** — `docs/trd/07-web-frontend-architecture.md` — TanStack Router file-based routing with route tree generation, _auth layout guard with role-based route protection, collapsible sidebar navigation (256px → 64px) with grouped menu items, desktop-first responsive layout (1024px+), sidebar state persisted in Zustand uiStore, React error boundary
- [ ] **Frontend testing setup** — `docs/trd/07-web-frontend-architecture.md` — Vitest configuration for all three apps and shared packages, React Testing Library, test utilities (mock query client, mock auth store, render helpers), CI integration
