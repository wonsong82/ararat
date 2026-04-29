# ADR-016: Backend Testing — Jest

**Status**: Accepted  
**Date**: 2026-03-01  
**Deciders**: Project owner, AI agent

## Context

The NestJS backend (ADR-011) requires a testing framework for unit tests, integration tests, and API endpoint tests. NestJS ships with built-in Jest support including `@nestjs/testing` utilities, test module builders, and preconfigured `jest` settings in its scaffold. A testing framework must be chosen and locked before implementation begins.

## Decision

Use **Jest** as the backend testing framework.

- **Unit tests**: Jest with NestJS `Test.createTestingModule()` for service/controller isolation
- **Integration tests**: Jest with `supertest` for HTTP endpoint testing against a test database
- **Coverage**: Jest built-in coverage reporter (`--coverage`)
- **Configuration**: `jest` config in `api/package.json` or `api/jest.config.ts`
- **Test file convention**: Co-located `*.spec.ts` files (NestJS default: `foo.service.spec.ts`)

## Alternatives Considered

- **Vitest**: Faster execution via Vite's transform pipeline, but NestJS has no official Vitest integration. `@nestjs/testing` utilities are designed for Jest. Would require custom setup and lose NestJS-specific test helpers.
- **Mocha + Chai**: Mature but requires more boilerplate. No built-in coverage. NestJS scaffolding doesn't support it out of the box.

## Consequences

### Positive
- Zero-config with NestJS — `@nestjs/cli` generates test files with Jest boilerplate
- `@nestjs/testing` module works seamlessly (dependency injection mocking, module overrides)
- Large ecosystem of Jest matchers and plugins
- Built-in code coverage reporting

### Negative
- Slower than Vitest for large test suites (mitigated by `--runInBand` for CI, `--watch` for dev)
- CommonJS-first module resolution can conflict with ESM libraries (mitigated by `ts-jest` transforms)

### Neutral
- Jest is the de facto standard for Node.js backend testing — widely understood by contributors

**Affects**: [05-api-design](../05-api-design.md), [22-infrastructure](../22-infrastructure.md)
