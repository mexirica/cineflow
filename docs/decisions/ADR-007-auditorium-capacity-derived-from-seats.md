# ADR-007: Auditorium Capacity Derived from Seats

- **Status:** Accepted
- **Date:** 2026-11-11

## Context

An auditorium has a physical seating capacity determined by its seat inventory.

Storing capacity independently would duplicate information and could become inconsistent when seats are added or removed.

## Decision

Auditorium capacity is derived from its associated Seat records.

Capacity is not stored as an independent source-of-truth attribute on Auditorium.

## Consequences

### Positive

- No duplicated capacity information.
- Seat inventory remains the source of truth.
- Adding or removing seats changes the derived capacity.

### Negative

- Capacity calculations require querying seat inventory.
- The definition of which seats count toward capacity must be clarified when designing the Seat model.

## Future Considerations

The exact capacity calculation depends on future Seat attributes and business rules. For example, inactive, blocked, or non-sellable seats may need different treatment.

Caching or denormalization may be considered later if performance requires it, without changing the source of truth.
