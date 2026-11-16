# ADR-004: Admin as an Independent Service

- **Status:** Accepted
- **Date:** 2026-11-11

## Context

Tenant management could initially be implemented as part of Venue.

However, Admin has a distinct responsibility: providing centralized administrative operations for the platform.

The project requires an administrative API to register, inspect, update, suspend, and activate cinemas. Admin also needs its own authentication and authorization boundaries.

## Decision

Implement **Admin as an independent service** with its own API and persistence.

Admin will not be embedded inside Venue.

Its initial scope focuses on tenant management and platform-level administrative operations.

## Consequences

### Positive

- Clear separation between administrative and business capabilities.
- Independent deployment and evolution.
- Dedicated administrative authentication and authorization.
- Venue does not become a central dependency for tenant management.

### Negative

- Additional service to deploy and maintain.
- Separate database and operational configuration.
- More infrastructure and integration complexity than an internal module.

## Initial Scope

Admin will initially manage:

- Tenant registration.
- Tenant lookup and listing.
- Tenant information updates.
- Tenant activation.
- Tenant suspension.
- Tenant deactivation.

Other administrative capabilities may be introduced as requirements emerge.

Admin does not own business operations belonging to other services.
