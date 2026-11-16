# ADR-001: Multi-Tenancy Strategy — Pool

- **Status:** Accepted
- **Date:** 2026-11-11

## Context

The platform must support multiple independent cinemas while maintaining tenant data isolation.

The considered multi-tenancy strategies are:

- **Pool:** Multiple tenants share the same database and tables.
- **Bridge:** Tenants use separate databases or schemas within shared infrastructure.
- **Silo:** Each tenant has dedicated database infrastructure.

The project also aims to gain practical experience with PostgreSQL multi-tenancy and its isolation challenges.

## Decision

Adopt the **Pool** strategy initially.

Each business service maintains its own PostgreSQL database, shared across all tenants using that service.

Tenant-owned records contain a `tenant_id` column. PostgreSQL Row-Level Security (RLS) is used to enforce tenant isolation at the database level.

## Consequences

### Positive

- Lower infrastructure and operational complexity.
- No database provisioning per tenant.
- Centralized schema migrations per service.
- Practical experience with PostgreSQL RLS and tenant isolation.
- Simpler initial deployment and maintenance.

### Negative

- Tenant isolation requires careful database and application design.
- Incorrect tenant context can expose data across tenants.
- Shared database resources can create noisy-neighbor problems.
- Connection and transaction handling require care to prevent tenant-context leakage.

## Implementation Considerations

- Apply RLS policies to tenant-owned tables.
- Establish tenant context through trusted application boundaries.
- Prevent tenant context leakage between reused database connections.
- Use tenant-aware constraints and indexes.
- Test cross-tenant access explicitly.
- Ensure the application database role cannot bypass RLS.

Bridge and Silo may be reconsidered if future requirements justify stronger isolation or dedicated infrastructure.
