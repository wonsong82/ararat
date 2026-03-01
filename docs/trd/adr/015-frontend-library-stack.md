# ADR-015: Frontend Library Stack

**Status**: Accepted  
**Date**: 2026-02-26  
**Deciders**: Project owner, AI agent

## Context

Ararat has 3 web-based SPAs (Parent App, Admin App, Monitor App) built with React + Vite (ADR-012). Beyond the framework choice, the specific libraries for UI components, styling, state management, forms, routing, charting, tables, and date handling must be decided to ensure consistency across all apps and enable a shared component library.

Key requirements:
- **Shared component library** across 3 web apps (Parent, Admin, Monitor)
- **Trilingual i18n** (EN/KO/ES) on the client side
- **Type safety** end-to-end with NestJS backend (shared Zod schemas)
- **Dashboard-heavy admin app** (tables, charts, forms, filters)
- **Mobile-responsive parent app** (card-based feed, simple forms)
- **Minimal monitor app** (read-only display, auto-refresh)

## Decision

| Concern | Library | Version |
|---------|---------|---------|
| **UI Components** | shadcn/ui + Radix UI | Latest |
| **Styling** | Tailwind CSS | v4+ |
| **State Management (server)** | TanStack Query (React Query) | v5 |
| **State Management (client)** | Zustand | v5 |
| **Form Handling** | React Hook Form + Zod | RHF v7 + Zod v3 |
| **Routing** | TanStack Router | v1 |
| **Data Tables** | TanStack Table | v8 |
| **Charts/Visualization** | Recharts | v2 |
| **Date/Time** | date-fns | v4 |
| **i18n (client)** | react-i18next + i18next | v14 |
| **HTTP Client** | ky (or native fetch wrapper) | Latest |
| **Icons** | Lucide React | Latest |

## Alternatives Considered

### UI Components
- **MUI (Material UI)**: Comprehensive but opinionated styling. Heavy bundle. Difficult to customize beyond Material Design aesthetic.
- **Ant Design**: Feature-rich but very opinionated Chinese design language. Customization requires deep theme overrides. Bundle size concerns.
- **Mantine**: Good DX and hooks library. Less ecosystem adoption than shadcn. Styling is CSS-in-JS based, doesn't pair natively with Tailwind.

**Why shadcn/ui**: Copy-paste component model means we own the code (no dependency lock-in). Built on Radix primitives (accessibility out of the box). Tailwind-native styling. Full control over design. Growing rapidly as the React community standard.

### Styling
- **CSS Modules**: Good isolation but no utility classes. Slower iteration.
- **styled-components / Emotion**: Runtime CSS-in-JS has performance overhead. Moving out of favor in React ecosystem.

**Why Tailwind CSS**: Utility-first enables rapid iteration. Native pairing with shadcn/ui. No runtime overhead (build-time only). Design tokens via CSS variables.

### State Management
- **Redux Toolkit**: Still powerful but heavier ceremony than needed. Most "state" in Ararat is server state (API responses).
- **Jotai**: Atomic state model is elegant but less discoverable for a team.

**Why TanStack Query + Zustand**: TanStack Query handles ~95% of state (server cache, loading, refetching, optimistic updates). Zustand handles the remaining client state (sidebar open, theme, language preference) with minimal boilerplate.

### Routing
- **React Router v7**: Widely used but less type-safe. File-based routing requires additional tooling.

**Why TanStack Router**: Full type safety for route params and search params. Built-in search param validation with Zod. Better DX for authenticated/protected routes.

### Forms
- **Formik**: Older, heavier, less active development.
- **TanStack Form**: Newer but less mature than RHF.

**Why React Hook Form + Zod**: Lightest bundle. Zod schemas are shareable with NestJS backend validation (same schema validates on both sides). Most mature and widely adopted.

## Consequences

**Positive:**
- Full type safety from database → API → frontend (TypeScript + Zod end-to-end)
- Zod schemas shared between NestJS validation pipes and React Hook Form resolvers — single source of truth for validation
- shadcn/ui components are owned code — no dependency version lock-in, full design control
- TanStack ecosystem (Query, Router, Table) shares consistent patterns and types
- Tailwind + shadcn gives consistent design language across all 3 web apps via shared component package
- All libraries are tree-shakeable — minimal bundle size

**Negative:**
- shadcn/ui is copy-paste — updates require manual merging (mitigated by shadcn CLI)
- TanStack Router is newer than React Router — smaller community, fewer tutorials
- Multiple TanStack libraries creates ecosystem coupling (mitigated by their independent release cycles)
- Recharts is less powerful than D3 for highly custom visualizations (sufficient for our dashboard needs)

**Affects**: [01-system-architecture](../01-system-architecture.md), [07-web-frontend-architecture](../07-web-frontend-architecture.md)
