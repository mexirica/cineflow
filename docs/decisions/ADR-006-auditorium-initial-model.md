# ADR-006: Auditorium Initial Model

- **Status:** Accepted
- **Date:** 2026-11-11

## Context

Venue needs an initial model for representing physical auditoriums.

The model should cover current requirements without introducing unnecessary attributes or lifecycle complexity.

## Decision

An Auditorium initially contains:

- `id`: unique identifier.
- `tenant_id`: owning tenant identifier.
- `name`: auditorium name.
- `status`: auditorium lifecycle status.
- `created_at`.
- `updated_at`.

The following business rules apply:

1. Auditorium names must be unique within the same tenant.
2. Different tenants may have auditoriums with the same name.
3. Status is either `active` or `inactive`.
4. Deletion uses hard delete.
5. No description field is included in the initial model.

The name uniqueness constraint is conceptually:

`UNIQUE (tenant_id, name)`

## Consequences

### Positive

- Simple and focused data model.
- Clear tenant-scoped naming rules.
- Explicit auditorium lifecycle.
- No unnecessary descriptive metadata.

### Negative

- Hard deletion requires considering external references.
- A future need for audit history or restoration may require revisiting deletion behavior.
- Auditorium metadata may need to evolve as product requirements grow.

## Future Considerations

The behavior of deleting an auditorium referenced by existing screenings or reservations remains an open integration decision.
