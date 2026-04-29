# ADR-020: Logging — nestjs-pino

**Status**: Accepted  
**Date**: 2026-03-01  
**Deciders**: Project owner, AI agent

## Context

The NestJS backend needs structured logging for debugging, monitoring, and audit trails. Logs must be machine-parseable (JSON) for CloudWatch ingestion, include request context (tenant ID, user ID, request ID), and have minimal performance impact on API response times. NestJS's built-in `Logger` outputs plain text and lacks structured formatting.

## Decision

Use **nestjs-pino** (Pino logger integration for NestJS) as the logging framework.

- **Logger**: `nestjs-pino` wraps Pino as a NestJS `LoggerService` — replaces the built-in logger globally
- **Format**: Structured JSON in all environments. Human-readable via `pino-pretty` in development only.
- **Request context**: Auto-attaches `requestId`, `tenantId`, `userId`, `method`, `url` to every log line via NestJS middleware
- **Log levels**: `trace`, `debug`, `info`, `warn`, `error`, `fatal` — configurable per environment (`debug` in dev, `info` in production)
- **Configuration**: `LoggerModule.forRoot()` in `app.module.ts`
- **Transport**: stdout → CloudWatch Logs agent picks up container stdout in ECS

## Alternatives Considered

- **Winston**: Feature-rich but heavier. Multiple transports add complexity. Slower than Pino in benchmarks (5-10x slower for structured JSON).
- **NestJS built-in Logger**: Plain text output, no structured format, no request context injection. Insufficient for production monitoring.
- **Bunyan**: Mature but less active development. Pino is its spiritual successor with better performance.

## Consequences

### Positive
- Fastest Node.js JSON logger (low-overhead async writes)
- Automatic request context in every log line — no manual `logger.log({ requestId })` boilerplate
- JSON output integrates directly with CloudWatch Logs Insights queries
- `pino-pretty` makes development logs readable without sacrificing production format
- NestJS-native integration — drop-in replacement for built-in logger

### Negative
- Pino's async logging means some logs may be lost on process crash (mitigated by `pino.final()` flush handler)
- JSON logs are unreadable without `pino-pretty` — development requires the pretty-printer

### Neutral
- Pino is the most popular structured logger in the Node.js ecosystem — well-maintained and battle-tested

**Affects**: [01-system-architecture](../01-system-architecture.md), [22-infrastructure](../22-infrastructure.md)
