# ADR-008: API-First RESTful Backend

**Status**: Accepted  
**Date**: 2025-02-25  
**Deciders**: Project Owner

## Context

The platform has four client applications: Parent/Member App (web), Admin App (web), Attendance Kiosk (iPad native), and Monitor App (TV display). All four need to access the same data and business logic. Code duplication across clients must be avoided.

## Decision

Build a single RESTful API backend that serves all four client applications. API-first design — define endpoints before building clients. Stateless architecture with JWT authentication. Consistent response envelope format across all endpoints.

## Alternatives Considered

1. **GraphQL**: More flexible querying, but adds complexity (schema management, N+1 queries, caching). Rejected for MVP — REST is simpler and sufficient.
2. **gRPC**: Better performance, but overkill for web clients and adds protobuf complexity. Rejected.
3. **Backend-per-app**: Each app has its own backend. Rejected — massive code duplication, consistency nightmares.
4. **Single REST API (chosen)**: Simple, well-understood, sufficient for all four apps. Accepted.

## Consequences

**Positive**:
- Single source of truth for business logic
- All four apps share the same API
- Easy to test (standard HTTP calls)
- Well-understood by any developer
- Stateless = horizontally scalable

**Negative**:
- Some endpoints may over-fetch or under-fetch for specific clients
- No real-time push from server (requires polling or WebSocket addition for live features)
- Versioning needed as API evolves

**Affects**: [01-system-architecture](../01-system-architecture.md), [05-api-design](../05-api-design.md)
