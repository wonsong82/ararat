# ADR-018: E2E Testing — Playwright

**Status**: Accepted  
**Date**: 2026-03-01  
**Deciders**: Project owner, AI agent

## Context

Critical user flows (registration, payment, attendance check-in) span the full stack — frontend UI → API → database. Unit and integration tests cover individual layers, but end-to-end (E2E) tests are needed to verify complete flows work correctly across layers. An E2E framework must support multiple browsers, run reliably in CI, and test the React SPAs against a running backend.

## Decision

Use **Playwright** as the E2E testing framework.

- **Scope**: Critical user flows only — registration, login, attendance, payment, belt promotion
- **Configuration**: `playwright.config.ts` at `web/` root
- **Test files**: `web/e2e/*.spec.ts`
- **Browsers**: Chromium (primary), Firefox, WebKit (CI matrix)
- **CI integration**: GitHub Actions with Playwright's official Docker image for consistent execution
- **Test strategy**: E2E tests run AFTER unit/integration tests pass. Not gating on every PR — run on `main` branch and pre-release
- **Fixtures**: Shared test fixtures for authenticated sessions, test tenants, seeded data

## Alternatives Considered

- **Cypress**: Popular but Chromium-only (experimental Firefox). Slower execution. Larger bundle. Architecture limitations (no multi-tab, no cross-origin by default).
- **Puppeteer**: Chromium-only. Lower-level API — more code for common testing patterns. No built-in test runner.
- **Selenium/WebDriver**: Cross-browser but slow, flaky, verbose. Outdated DX compared to modern alternatives.

## Consequences

### Positive
- True cross-browser testing (Chromium, Firefox, WebKit) from a single test suite
- Fast execution via browser contexts (no browser restart between tests)
- Built-in auto-waiting, network interception, and trace viewer for debugging
- Official Docker image for CI — deterministic test environment
- Strong TypeScript support with auto-generated types

### Negative
- Heavier CI pipeline — requires browser binaries (mitigated by Docker image caching)
- E2E tests are inherently slower than unit tests — must be scoped to critical paths only
- Requires running backend + database for tests (mitigated by Docker Compose test environment)

### Neutral
- Playwright is maintained by Microsoft — active development and long-term support expected

**Affects**: [07-web-frontend-architecture](../07-web-frontend-architecture.md), [22-infrastructure](../22-infrastructure.md)
