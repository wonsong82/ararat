# ADR-012: React + Vite Frontend

**Status**: Accepted  
**Date**: 2026-02-25  
**Deciders**: Project owner, AI agent

## Context

Ararat has 3 web-based client applications: Parent/Member App (mobile-responsive), Admin App (desktop-focused dashboard), and Monitor App (TV display). All are API-driven SPAs that consume the NestJS REST API.

## Decision

Use **React + Vite** (TypeScript) for all web-based client applications.

## Alternatives Considered

- **React + Next.js**: SSR/SSG capabilities and file-based routing. However, Ararat's web apps are authenticated dashboards behind login — SEO is irrelevant. Next.js adds complexity (server components, hydration) that provides no value for this use case.
- **Vue.js + Nuxt**: Simpler learning curve and good DX, but smaller ecosystem than React. Fewer UI component libraries, fewer developers available for hire.

## Consequences

**Positive:**
- TypeScript shared with NestJS backend — shared types, validation schemas, and utility libraries
- Vite provides fast dev server (HMR) and optimized production builds without Next.js complexity
- React has the largest ecosystem of UI component libraries (for Admin dashboard: tables, charts, forms)
- SPA architecture is ideal for authenticated dashboard apps (no SSR needed)
- Monorepo-friendly — shared packages between Parent App, Admin App, and Monitor App

**Negative:**
- No SSR — not suitable if SEO becomes important later (unlikely for authenticated apps)
- Three separate SPA builds to maintain (Parent, Admin, Monitor) — mitigated by shared component library

**Affects**: [01-system-architecture](../01-system-architecture.md), [17-monitor-app](../17-monitor-app.md)