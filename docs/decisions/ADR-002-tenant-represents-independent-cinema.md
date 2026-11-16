# ADR-002: Tenant Represents One Independent Cinema

- **Status:** Accepted
- **Date:** 2026-11-11

## Context

The platform needs a tenant model that represents its customers.

An organizational hierarchy with organizations and cinemas was considered. However, the current product scope targets independent cinemas, where each customer manages exactly one physical cinema.

Introducing an additional organization or cinema hierarchy would add complexity without providing current business value.

## Decision

A Tenant represents exactly one independent cinema.

The tenant itself represents the cinema business and its physical location.

There will be no separate `Organization` or `Cinema` entity in the initial model.

## Consequences

### Positive

- Simpler tenant model.
- Fewer entities and relationships.
- Clear ownership boundaries.
- No need for organization-to-cinema management.

### Negative

- The model does not directly support a customer managing multiple cinema locations.
- Supporting cinema chains may require introducing an organizational hierarchy later.

## Future Considerations

If the product evolves to support cinema chains, revisit the model. A possible future hierarchy could be:

`Organization → Tenants → Physical Cinemas`

This is not part of the current scope.
