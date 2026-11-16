# ADR-008: Tenant Registration Without Provisioning

- **Status:** Accepted
- **Date:** 2026-11-11

## Context

Tenant onboarding initially appeared to require infrastructure provisioning and resource initialization.

However, the selected Pool strategy uses shared databases across tenants.

The current registration workflow does not require:

- Creating a dedicated database.
- Allocating tenant-specific database resources.
- Initializing resources across multiple business services.
- Running distributed provisioning workflows.

## Decision

Tenant registration consists of creating a tenant record in the Admin PostgreSQL database.

New tenants are created directly with `active` status.

There is no `provisioning` status or provisioning workflow in the initial architecture.

## Consequences

### Positive

- Simpler registration workflow.
- No unnecessary orchestration.
- No provisioning retries or compensation logic.
- Fewer lifecycle states.

### Negative

- Future onboarding requirements involving multiple services may require a more sophisticated workflow.
- Activating a tenant does not imply that every possible business configuration has been initialized.

## Future Considerations

If the architecture evolves toward Bridge or Silo, or requires distributed resource initialization, provisioning may be introduced.

Until then, registration remains a straightforward database operation.
