# Cineflow Architecture Overview

## 1. Purpose

This document consolidates the current high-level architecture and service boundaries for Cineflow.

It summarizes responsibilities, integration patterns, and tenant-isolation principles already defined across service-specific documents and ADRs.

## 2. Platform Topology

```text
Clients
   |
   v
Gateway
   |
   +--> Admin
   +--> Venue
   +--> Catalog
   +--> Sessions
   +--> Reservations
   +--> Payments

Authentication: Keycloak (token issuer and identity provider)
Async messaging: RabbitMQ (service-to-service events)
Data stores: one PostgreSQL database per service (Pool multi-tenancy)
```

## 3. Service Boundaries (Summary)

| Component | Primary responsibility | Owns business data? |
|---|---|---|
| Gateway | External entry point, token validation, routing, trusted context propagation | No |
| Keycloak | Authentication and token issuance | Identity/infrastructure data |
| Admin | Tenant registration, tenant lifecycle, platform-level admin operations | Yes |
| Venue | Physical auditoriums and seats | Yes |
| Catalog | Movie catalog and metadata | Yes |
| Sessions | Session scheduling and session lifecycle | Yes |
| Reservations | Reservation lifecycle and session-specific seat availability | Yes |
| Payments | Payment attempts, payment status, Stripe integration | Yes |

## 4. Communication Patterns

### Synchronous

- Clients call the platform through Gateway.
- Gateway routes requests to internal services.
- Services may call other services through internal APIs when synchronous validation is needed.

Examples:
- Payments validates reservation state with Reservations before creating a payment attempt.
- Reservations validates session references through Sessions.

### Asynchronous

- Domain services publish and consume business events through RabbitMQ.
- Asynchronous messaging is used for workflow progression where temporal decoupling is required.

Examples:
- Payments publishes payment outcome events.
- Reservations consumes those events to confirm, cancel, or expire reservations according to business rules.

## 5. Tenant Isolation Model

Cineflow uses the **Pool** multi-tenancy strategy:

- Each service owns its own PostgreSQL database.
- Multiple tenants share the same database within that service.
- Tenant-owned records include `tenant_id`.
- PostgreSQL Row-Level Security (RLS) enforces tenant isolation at data level.
- Tenant context is established at trusted platform boundaries and propagated across sync and async flows.

## 6. Data Ownership and Duplication

- Each service owns its own source-of-truth data and schema.
- Cross-service data access must happen through APIs or events, never direct database access.
- Local read projections/denormalized views are allowed when needed for performance or workflow autonomy, as long as ownership boundaries are preserved.

## 7. Key Business Flows (High-Level)

### 7.1 Tenant creation

1. Platform administrator is authenticated via Keycloak.
2. Request enters through Gateway.
3. Admin registers tenant and assigns lifecycle status.
4. Tenant identifier becomes the cross-service reference (`tenant_id`).

### 7.2 Session scheduling

1. Tenant-scoped user is authenticated via Keycloak.
2. Request enters through Gateway with trusted tenant context.
3. Sessions validates movie and physical inventory references through the relevant service integrations.
4. Sessions persists session schedule and lifecycle data.

### 7.3 Reservation and payment

1. Reservations creates a pending reservation and reserves seat availability for a session.
2. Payments creates payment attempt and starts Stripe Checkout.
3. Stripe webhook result is processed by Payments.
4. Payments publishes payment outcome event.
5. Reservations consumes the event and transitions reservation state.

## 8. Current Open Decisions

- Detailed API contracts among Gateway and domain services.
- Final role/permission matrix aligned across Keycloak, Gateway, Admin, and domain services.
- Exact event schemas, delivery guarantees, and versioning policies.
- Operational standards for retries, dead-letter queues, and cross-service observability.
