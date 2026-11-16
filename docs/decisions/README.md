# Architecture Decision Records

This directory contains the Architecture Decision Records (ADRs) for the cinema management platform.

ADRs document important architectural decisions, their context, the chosen approach, and their consequences.

## Decision Index

| ADR | Decision | Status |
|---|---|---|
| [ADR-001](ADR-001-multi-tenancy-pool.md) | Multi-Tenancy Strategy: Pool | Accepted |
| [ADR-002](ADR-002-tenant-represents-independent-cinema.md) | Tenant Represents One Independent Cinema | Accepted |
| [ADR-003](ADR-003-admin-owns-tenant-lifecycle.md) | Admin Owns Tenant Identity and Lifecycle | Accepted |
| [ADR-004](ADR-004-admin-as-independent-service.md) | Admin as an Independent Service | Accepted |
| [ADR-005](ADR-005-venue-owns-physical-inventory.md) | Venue Owns Physical Cinema Inventory | Accepted |
| [ADR-006](ADR-006-auditorium-initial-model.md) | Auditorium Initial Model | Accepted |
| [ADR-007](ADR-007-auditorium-capacity-derived-from-seats.md) | Auditorium Capacity Derived from Seats | Accepted |
| [ADR-008](ADR-008-tenant-registration-without-provisioning.md) | Tenant Registration Without Provisioning | Accepted |


## ADR Conventions

- Use one file per architectural decision.
- Accepted ADRs should not be silently rewritten when a decision changes.
- A superseding decision should be documented in a new ADR, with links to the previous decision.
- Proposed decisions should use `Proposed` status until explicitly accepted.
