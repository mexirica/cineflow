# Keycloak — Identity and Access Boundary

## 1. Purpose

Keycloak is the platform Identity Provider (IdP) for Cineflow.

It is responsible for identity authentication, token issuance, and identity-related access metadata consumed by platform services.

## 2. Responsibilities

Keycloak is responsible for:

- User authentication.
- Access and refresh token issuance.
- Identity session management.
- Identity federation capabilities (when needed in future scope).
- Client/application identity for service integrations that require OAuth2/OIDC.
- Publishing identity claims required by platform authorization flows.

## 3. Out of Scope

Keycloak is not responsible for:

- Tenant lifecycle ownership.
- Business-domain authorization rules.
- Domain data persistence for Admin, Venue, Catalog, Sessions, Reservations, or Payments.

Tenant registration and lifecycle remain owned by Admin.

## 4. Relationship with Admin and Gateway

- **Admin** owns tenant identity and tenant lifecycle in business terms.
- **Keycloak** owns user authentication and token issuance.
- **Gateway** validates Keycloak tokens and propagates trusted identity/tenant context to internal services.

This separation avoids coupling platform identity infrastructure with business-domain ownership.

## 5. Token and Claim Model (Conceptual)

Tokens consumed by Gateway and internal services should carry enough information to support authorization and tenant isolation.

Conceptual claims:

- Subject/user identifier.
- Roles/authorities.
- Tenant identifier for tenant-scoped users.
- Token issuer and expiration metadata.

Platform administrators may operate without a tenant-scoped context for platform-level endpoints, while tenant users are always constrained to their tenant context.

## 6. Authorization Model (High Level)

Two conceptual access profiles:

- **Platform Administrator:** allowed to execute platform-level administrative operations.
- **Cinema User:** allowed to access tenant-scoped business operations only within its own tenant.

Detailed role and permission matrices remain open decisions and must be aligned across Gateway, Admin, and domain services.

## 7. Security Requirements

- All public API authentication must rely on Keycloak-issued tokens.
- Token validation must happen at platform boundary (Gateway) and be defensively rechecked where required by sensitive operations.
- Services must never accept tenant impersonation through client-provided unmanaged headers.
- Token lifetime, refresh policy, and key rotation must be operationally defined before production rollout.

## 8. Open Decisions

- Realm/tenant partitioning strategy in Keycloak.
- Exact role taxonomy and mapping to platform/domain permissions.
- Service-to-service authentication strategy (client credentials, token exchange, or both).
- User provisioning and lifecycle synchronization between Admin and Keycloak.
