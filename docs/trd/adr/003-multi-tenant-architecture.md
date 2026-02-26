# ADR-003: Multi-Tenant Single-Deployment Architecture

**Status**: Accepted  
**Date**: 2025-02-25  
**Deciders**: Project Owner

## Context

The platform serves multiple Taekwondo gyms. Each gym's data must be completely isolated. The architecture must support scaling to 500+ gyms without requiring separate deployments per gym.

## Decision

Use application-level multi-tenancy with a single deployment. Every database table includes a `tenant_id` column. All queries filter by `tenant_id`. PostgreSQL Row-Level Security (RLS) provides an additional safety layer. Tenant context is propagated via JWT (`tenantId` in payload).

## Alternatives Considered

1. **Separate database per tenant**: Strongest isolation but operationally expensive (500 databases = 500 backups, 500 migrations). Rejected.
2. **Separate schema per tenant**: Good isolation, but complicates migrations and connection pooling. Rejected.
3. **Shared tables with tenant_id (chosen)**: Simplest to operate, scales well, RLS provides safety net. Accepted.

## Consequences

**Positive**:
- Single deployment to maintain and update
- Single database to back up and monitor
- Simplified CI/CD pipeline
- Cost-efficient (shared infrastructure)
- Easy to add new tenants (just insert a row)

**Negative**:
- Must be disciplined about `tenant_id` filtering — a missed filter leaks data
- RLS policies must be maintained for every table
- "Noisy neighbor" risk if one gym generates disproportionate load
- Cross-tenant reporting requires careful query design

**Affects**: [01-system-architecture](../01-system-architecture.md), [02-multi-tenancy](../02-multi-tenancy.md), [03-data-model](../03-data-model.md)
