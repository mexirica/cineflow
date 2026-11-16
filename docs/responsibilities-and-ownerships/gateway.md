# Gateway Service — Responsibilities & Boundaries

## 1. Purpose

The Gateway is the single entry point for external clients accessing Cineflow services.

It centralizes cross-cutting concerns (authentication, request routing, tenant context propagation, and observability) so business services can focus on domain logic.

## 2. Core Responsibilities

Gateway is responsible for:

- Exposing public API endpoints for clients.
- Routing requests to internal services.
- Validating access tokens issued by Keycloak.
- Enforcing coarse-grained authorization at the platform boundary.
- Establishing a trusted tenant context from authenticated identity claims.
- Propagating request metadata (correlation IDs, tenant context, user context) to downstream services.
- Applying platform-level concerns such as rate limiting and request tracing.

## 3. Out of Scope

Gateway is not responsible for:

- Business rule validation specific to a domain service.
- Domain data ownership.
- Database persistence.
- Replacing service-level authorization and tenant isolation checks.

Each service remains responsible for validating tenant isolation and business permissions in its own boundary.

## 4. Tenant Context

Tenant identity must be derived from trusted authentication context, never from arbitrary client-controlled headers.

Principles:

- Gateway must not trust a tenant ID provided directly by clients.
- Tenant context is derived from validated identity claims.
- Internal propagation must use trusted headers/metadata consumed by internal services.
- Downstream services must still enforce tenant isolation with application checks and PostgreSQL RLS.

## 5. Integration Model

### Synchronous integrations

Gateway communicates synchronously (GRPC) with internal services:

- Admin
- Venue
- Catalog
- Sessions
- Reservations
- Payments

### Asynchronous integrations

Gateway does not own asynchronous business workflows.

Business events are exchanged directly between domain services through RabbitMQ when required by use cases.

## 6. Data Ownership

Gateway does not own business entities and should not have its own domain database.

If operational data is required (for example, routing metadata, rate-limit counters, or audit traces), it is considered technical platform data and not domain ownership.

## 7. Security and Observability Requirements

- Validate JWT tokens and signature against Keycloak.
- Reject unauthenticated or invalid-token requests at the edge.
- Differentiate platform-level access (for administrative endpoints) from tenant-scoped access.
- Propagate correlation IDs for traceability across services.
- Produce structured logs and metrics for request lifecycle visibility.

## 8. Open Decisions

- API composition strategy in Gateway (pure pass-through vs. aggregated endpoints).
- Centralized response normalization policy.
- Exact tenant-context propagation contract (header names and signing/trust model).
- Edge caching strategy for read-heavy endpoints.
