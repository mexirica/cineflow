# ADR-003: Admin Owns Tenant Identity and Lifecycle

- **Status:** Accepted
- **Date:** 2026-11-11

## Context

All business services need a shared tenant identifier to associate their data with the correct cinema.

Tenant registration and lifecycle management are platform-level responsibilities rather than responsibilities of individual business services.

Placing tenant management inside Venue would create an unnecessary dependency between platform identity and physical cinema inventory.

## Decision

The **Admin Service** is the authoritative owner of tenant identity and lifecycle.

Admin owns:

- Tenant registration.
- Tenant identifiers.
- Tenant information.
- Tenant lifecycle and status.
- Administrative operations related to tenants.

Business services store and use `tenant_id` but do not own the Tenant entity or its lifecycle.

## Consequences

### Positive

- Centralized tenant registry.
- Clear separation between platform administration and business operations.
- Business services remain independent of tenant registration workflows.
- No business service becomes responsible for managing the platform's customers.

### Negative

- Tenant context must be propagated reliably to business services.
- Tenant lifecycle changes may need to affect multiple services.
- Administrative and business authorization must remain clearly separated.

## Ownership Boundaries

| Responsibility | Owner |
|---|---|
| Tenant registration | Admin |
| Tenant identity | Admin |
| Tenant lifecycle | Admin |
| Auditorium management | Venue |
| Physical seat inventory | Venue |
| Tenant-scoped business data | Respective business service |
