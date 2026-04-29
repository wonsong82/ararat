# ADR-017: Frontend Testing — Vitest

**Status**: Accepted  
**Date**: 2026-03-01  
**Deciders**: Project owner, AI agent

## Context

The web frontend monorepo (3 React + Vite apps per ADR-012) requires a unit/component testing framework. Since all web apps use Vite as the build tool, the testing framework should integrate natively with Vite's transform pipeline to avoid duplicate configuration and ensure fast test execution.

## Decision

Use **Vitest** as the frontend testing framework for all web apps (Parent, Admin, Monitor).

- **Unit tests**: Vitest with `@testing-library/react` for component testing
- **Utility tests**: Pure Vitest for shared packages (`packages/ui/`, `packages/shared/`)
- **Coverage**: Vitest built-in coverage via `v8` provider
- **Configuration**: `vitest.config.ts` in each app and shared package, extending root Vite config
- **Test file convention**: Co-located `*.test.tsx` / `*.test.ts` files
- **DOM environment**: `jsdom` (via `vitest` config `environment: 'jsdom'`)

## Alternatives Considered

- **Jest**: Industry standard but requires separate Babel/SWC transforms for JSX/TSX. Duplicate config with Vite. Slower cold starts. No native ESM support without workarounds.
- **Testing Library only (without runner)**: Not a test runner — requires Jest or Vitest to execute.

## Consequences

### Positive
- Native Vite integration — shares `vite.config.ts` transforms, aliases, and plugins
- Fast execution via Vite's esbuild-powered transforms
- API-compatible with Jest (same `describe`/`it`/`expect` syntax) — low learning curve
- Hot module reload for tests in watch mode (`vitest --watch`)
- Built-in TypeScript support without additional configuration

### Negative
- Smaller plugin ecosystem compared to Jest (mitigated by Jest-compatible API)
- Newer project — less battle-tested for edge cases (mitigated by rapid maturity and Vite team backing)

### Neutral
- Vitest is becoming the standard for Vite-based projects — aligns with ecosystem direction

**Affects**: [07-web-frontend-architecture](../07-web-frontend-architecture.md), [22-infrastructure](../22-infrastructure.md)
