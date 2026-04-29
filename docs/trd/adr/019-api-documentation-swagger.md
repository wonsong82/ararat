# ADR-019: API Documentation — @nestjs/swagger

**Status**: Accepted  
**Date**: 2026-03-01  
**Deciders**: Project owner, AI agent

## Context

The Ararat backend exposes a REST API consumed by 4 clients (Parent App, Admin App, Monitor App, Kiosk App). API documentation must be accurate, always up-to-date, and explorable by developers. Manual documentation drifts from actual implementation. An auto-generation approach tied to the source code ensures documentation stays in sync.

## Decision

Use **@nestjs/swagger** to auto-generate OpenAPI (Swagger) documentation from NestJS decorators.

- **Decorators**: `@ApiTags()`, `@ApiOperation()`, `@ApiResponse()`, `@ApiBearerAuth()` on all controllers
- **DTO documentation**: `@ApiProperty()` on all request/response DTOs — generates schema definitions
- **Serve endpoint**: Swagger UI available at `/api/docs` (development and staging only, disabled in production)
- **OpenAPI spec export**: JSON spec at `/api/docs-json` for client SDK generation if needed
- **Validation sync**: DTOs use `class-validator` decorators that `@nestjs/swagger` reads — single source for both validation and documentation

## Alternatives Considered

- **Manual OpenAPI YAML**: Full control over spec but drifts from code. Requires discipline to update on every endpoint change. High maintenance burden.
- **TypeDoc**: Generates docs from TSDoc comments but doesn't produce OpenAPI spec. Not suitable for API documentation.
- **Redocly**: Better UI than Swagger UI but requires a separate OpenAPI spec file — doesn't integrate with NestJS decorators.

## Consequences

### Positive
- Documentation auto-generated from code — always matches actual API behavior
- DTOs serve triple duty: validation (class-validator), serialization (class-transformer), documentation (@nestjs/swagger)
- Interactive Swagger UI for testing endpoints during development
- OpenAPI spec can generate client SDKs if mobile/external clients need them

### Negative
- Decorator verbosity — every controller and DTO needs swagger decorators (mitigated by NestJS CLI plugin that auto-infers from TypeScript types)
- Swagger UI disabled in production — requires staging access for API exploration

### Neutral
- @nestjs/swagger is the official NestJS solution — widely adopted and well-maintained

**Affects**: [05-api-design](../05-api-design.md), [22-infrastructure](../22-infrastructure.md)
