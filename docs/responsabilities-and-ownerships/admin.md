# Admin Service — Responsibilities & Data Ownership

## 1. Purpose

The Admin Service is responsible for managing the platform's administrative operations and tenant lifecycle.

A tenant represents one independent cinema using the platform.

Admin is the single source of truth for:
- Tenant registration and identity.
- Tenant lifecycle and status.
- Platform-level administrative operations.

The service exposes an administrative API used to manage cinemas registered on the platform.

Admin does not own the business data managed by individual services.

## 2. Core Responsibilities

### Tenant Management

Admin manages the registration and lifecycle of tenants.

Responsibilities:
- Register new tenants.
- Generate unique tenant identifiers.
- Store tenant information.
- Retrieve tenant information.
- Update tenant information.
- Activate, suspend, and deactivate tenants.
- Enforce tenant lifecycle rules.

Current tenant statuses:

- `active`: tenant can use the platform.
- `suspended`: tenant is temporarily prevented from using the platform.
- `deactivated`: tenant has been deactivated.

A newly registered tenant is created directly with `active` status.

The initial registration process consists of inserting a tenant record into PostgreSQL. No distributed provisioning or resource initialization workflow is required.

The exact lifecycle transitions and business rules remain open decisions.

### Administrative Operations

Admin provides centralized administrative capabilities for platform operators.

Responsibilities:
- Expose administrative APIs.
- Restrict administrative operations to authorized users.
- Provide visibility into registered tenants.
- Execute tenant lifecycle operations.
- Maintain administrative operation consistency.

Additional administrative capabilities may be introduced as platform requirements evolve.

## 3. Data Ownership

Admin owns platform-level administrative data.

### Tenant

Represents a customer of the platform. In the current product scope, one tenant corresponds to one independent cinema.

Conceptually, a Tenant contains:

- `id`: unique tenant identifier.
- `name`: tenant or cinema name.
- `status`: current lifecycle status.
- `created_at`
- `updated_at`

Additional registration and contact information may be introduced when required by product needs.

### Ownership relationship

```text
Admin
  |
  +-- Tenant
         |
         +-- Identity
         +-- Lifecycle
         +-- Status
```

The Tenant is a platform-level identity.

Other services reference it through `tenant_id`, but do not own or manage its registration or lifecycle.

## 4. Multi-Tenancy Strategy

### Selected approach: Pool

The platform uses the Pool strategy.

Each business service operates with a shared PostgreSQL database across multiple tenants.

Tenant-owned records contain a `tenant_id` column, and isolation is enforced through PostgreSQL Row-Level Security (RLS).

Admin maintains the tenant registry independently from the operational data owned by business services.

Each service owns its own database. Multiple tenants share the same database within each service.

Important principles:
- There is no dedicated database per tenant.
- Admin does not manage database allocation or tenant-to-database bindings.
- Admin does not provision databases when registering tenants.
- Tenant registration does not require initialization workflows in other services.
- Business services are responsible for tenant isolation in their own databases.

The `tenant_id` is the shared identifier used across services to associate records with the correct tenant.

## 5. Service Boundaries

Admin manages platform-level administration, not the business operations of a cinema.

### In scope
- Tenant registration.
- Tenant identity and lifecycle.
- Tenant activation, suspension, and deactivation.
- Tenant-level administrative information.
- Administrative API.
- Authorization of platform administrative operations.

### Out of scope
- Auditorium and seat management.
- Movie catalog and metadata.
- Screening schedules.
- Reservations and bookings.
- Payment processing.
- Cinema employee accounts and business-level permissions.
- Database provisioning and database bindings.
- Business-specific configuration owned by other services.

Admin should not become the owner of every piece of information associated with a tenant.

It owns platform-level identity and administrative operations. Each business service owns its own tenant-scoped business data.

## 6. Authentication and Authorization

The administrative API must be restricted to authorized platform administrators.

The platform uses Keycloak as its authentication provider.

Administrative authorization must be separated from tenant-level business authorization.

Conceptually:

- **Platform Administrator:** can manage tenants and execute authorized platform-level operations.
- **Cinema User:** belongs to a tenant and accesses only the functionality permitted within that tenant.

Admin must validate administrative permissions independently of the tenant context used by business services.

The exact role model and authorization policies remain open decisions.

## 7. Tenant Context

Business services need to identify the tenant associated with each incoming request.

Tenant identity must be established through a trusted platform boundary and propagated to internal services.

Important principles:
- Tenant identity must come from a trusted source.
- Clients must not be allowed to impersonate another tenant simply by changing a request header.
- Each service must validate and enforce tenant isolation independently.
- Tenant context must be propagated consistently across HTTP requests and asynchronous messages.
- PostgreSQL RLS must enforce isolation at the database level.

Business services should not need to query Admin synchronously for every business operation just to resolve the tenant identifier.

## 8. External Integration

Admin exposes synchronous APIs for administrative operations.

It may also publish lifecycle events when other services need to react to tenant changes.

Potential events:
- `TenantRegistered`
- `TenantActivated`
- `TenantSuspended`
- `TenantDeactivated`

Events are optional integration mechanisms, not prerequisites for tenant registration.

Event contracts, delivery guarantees, and versioning will be defined when implementing these workflows.