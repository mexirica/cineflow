# Venue Service — Responsibilities & Data Ownership

## 1. Purpose

The Venue Service manages the physical structure and seating inventory of an independent cinema.

Each tenant represents exactly one independent cinema. Venue is responsible for managing the physical spaces and seats belonging to that tenant.

Venue is the single source of truth for:
- Auditoriums (screening rooms).
- Physical seats within each auditorium.

The service does not manage the tenant's identity or lifecycle. It receives and uses the tenant identifier provided by the platform infrastructure.

## 2. Core Responsibilities

### Auditorium Management

Venue manages the lifecycle and configuration of physical auditoriums.

Responsibilities:
- Create auditoriums.
- Retrieve auditorium information.
- Update auditorium information.
- Activate and deactivate auditoriums.
- Delete auditoriums.
- Enforce auditorium naming uniqueness within a tenant.
- Provide auditorium information to authorized consumers.

Current business rules:
- Auditorium names must be unique within the tenant.
- Status is either `active` or `inactive`.
- Capacity is derived from the auditorium's seats, not stored independently.

### Seat Management

Venue manages the physical seat inventory of each auditorium.

Responsibilities:
- Create and configure seats.
- Update physical seat information.
- Remove seats.
- Retrieve seats belonging to an auditorium.
- Enforce seat consistency within its auditorium.
- Ensure seats cannot be associated with an auditorium from another tenant.

The detailed seat model and validation rules remain open architectural decisions.

## 3. Data Model — High Level

### Auditorium

Conceptually, an auditorium contains:

- `id`: unique identifier.
- `tenant_id`: owning tenant identifier.
- `name`: auditorium name, unique within the tenant.
- `status`: `active` or `inactive`.
- `created_at`
- `updated_at`

Capacity is calculated from the seats belonging to the auditorium.

### Seat

A seat belongs to exactly one auditorium and one tenant.

The detailed attributes, physical identification scheme, and constraints will be defined separately.

## 4. Multi-Tenancy Strategy

### Selected approach: Pool

Initially, Venue uses a shared PostgreSQL database for multiple tenants.

All tenant-owned records contain a `tenant_id` column.

Conceptually:

```text
auditoriums
-----------
id
tenant_id
name
status
created_at
updated_at

seats
-----
id
tenant_id
auditorium_id
...
```

Tenant isolation is enforced using PostgreSQL Row-Level Security (RLS), alongside application-level validation and authorization.

Architectural requirements:
- Every tenant-owned record must be associated with a tenant.
- Queries must execute within a trusted tenant context.
- RLS policies must prevent cross-tenant access.
- The application database role must not bypass RLS.
- Tenant context must not leak between requests or reused database connections.
- Relationships must preserve tenant consistency.

Pool is the initial strategy. Bridge and Silo remain possible future alternatives if isolation, scale, or customer requirements justify them.

## 5. Service Boundaries

Venue owns physical infrastructure, not the operational use of that infrastructure.

### In scope
- Auditorium structure.
- Physical seat inventory.
- Auditorium and seat configuration.
- Physical inventory consistency.

### Out of scope
- Tenant registration and lifecycle.
- Employee accounts, memberships, roles, and permissions.
- Movie metadata.
- Screening schedules.
- Seat availability for a particular screening.
- Reservations and bookings.
- Payments and transactions.

In particular, Venue does not determine whether a seat is available for a screening. It only knows that the physical seat exists and belongs to an auditorium.

## 6. External Integration

Venue exposes its capabilities through public APIs and, where appropriate, asynchronous events.

Other services must not access Venue's PostgreSQL tables directly.

Potential integration mechanisms:

**Synchronous APIs**
- Retrieve auditorium information.
- Retrieve seat inventory.
- Validate physical auditorium or seat references.

**Asynchronous events**

Venue may publish events when its physical inventory changes.

Potential events:
- `AuditoriumCreated`
- `AuditoriumUpdated`
- `AuditoriumDeactivated`
- `SeatConfigurationChanged`

Event contracts, delivery guarantees, and versioning will be defined when implementing these workflows.