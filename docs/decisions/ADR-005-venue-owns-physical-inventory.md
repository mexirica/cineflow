# ADR-005: Venue Owns Physical Cinema Inventory

- **Status:** Accepted
- **Date:** 2026-11-11

## Context

The platform needs a clear boundary for managing the physical structure of each cinema.

Physical auditorium and seat information is required by other business capabilities, particularly screening and reservation management. However, physical inventory and its operational usage are different responsibilities.

## Decision

Venue is the authoritative owner of physical cinema inventory.

Venue owns:

- Auditoriums.
- Physical seats.

The ownership relationship is:

`Tenant → Auditorium → Seat`

The Tenant is an external identity owned by Admin and referenced through `tenant_id`.

Venue does not own Tenant, Organization, or Cinema entities.

## Out of Scope

Venue does not manage:

- Tenant registration or lifecycle.
- Movie catalog.
- Screening schedules.
- Seat availability for individual screenings.
- Reservations or bookings.
- Payments.
- Employee accounts and business permissions.

## Consequences

### Positive

- Clear ownership of physical inventory.
- No duplication of auditorium or seat records.
- Other services consume Venue capabilities without accessing its database.
- Physical inventory remains independent of screening and reservation state.

### Negative

- Other services need APIs or events to obtain relevant inventory information.
- Cross-service references must be managed carefully.
- Deletion of physical resources must account for external references.

## Important Distinction

Venue owns the physical existence and configuration of seats.

Reservations owns the availability and reservation state of seats for individual screenings.
