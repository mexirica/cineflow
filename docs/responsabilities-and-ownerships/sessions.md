# Sessions Service

## 1. Purpose

The Sessions Service is responsible for creating and managing movie screening sessions for each cinema.

A session represents a scheduled exhibition of one movie in one auditorium at a specific date and time.

## 2. Responsibilities

The Sessions Service is responsible for:

- Manual session creation.
- Session retrieval and listing.
- Session lifecycle updates (for example, activation and cancellation, when applicable).
- Scheduling validation.
- Preventing overlapping sessions in the same auditorium.
- Enforcing session immutability constraints after publication, according to business rules.
- Exposing session information to dependent services.

## 3. Out of Scope

The Sessions Service is not responsible for:

- Tenant registration and tenant lifecycle (Admin).
- Physical auditorium and seat ownership (Venue).
- Movie metadata ownership (Catalog).
- Reservation creation or seat-hold logic (Reservations).
- Payment processing (Payments).
- Authentication and identity management (Keycloak/Gateway boundary).

## 4. Core Data Ownership

Sessions owns session scheduling and lifecycle data.

### Session (conceptual)

- `id`
- `tenant_id`
- `movie_id`
- `auditorium_id`
- `starts_at`
- `ends_at`
- `status`
- `created_at`
- `updated_at`

`movie_id` and `auditorium_id` are external references validated through service integrations.

## 5. Business Rules (Initial)

### 5.1 Session references

- A session must reference an existing movie in Catalog.
- A session must reference an existing auditorium in Venue.
- All references must belong to the same `tenant_id`.

### 5.2 Time consistency

- `ends_at` must be greater than `starts_at`.
- A session cannot be created in an invalid time window.

### 5.3 Overlap prevention

- Two active sessions cannot overlap in the same auditorium for the same tenant.
- Overlap checks must be enforced at creation and relevant update operations.

### 5.4 Deletion constraints

- A session can be deleted only when no reservations exist for that session.
- Validation must be performed through Reservations integration.

## 6. Multi-tenancy

The service follows the Pool strategy:

- One shared PostgreSQL database per service.
- Tenant-owned records include `tenant_id`.
- PostgreSQL Row-Level Security (RLS) enforces tenant isolation.
- Requests and background processing must execute with trusted tenant context.

## 7. Integrations

Sessions must not access other services' databases directly.

| Service | Integration purpose | Mode |
|---|---|---|
| Admin | Tenant identity context, when needed | Synchronous (indirect/platform context) |
| Venue | Validate auditorium references | Synchronous API |
| Catalog | Validate movie references | Synchronous API |
| Reservations | Check reservation existence before session deletion | Synchronous API |

Sessions may publish events when session lifecycle changes are relevant to downstream consumers.

Potential events:

- `SessionCreated`
- `SessionUpdated`
- `SessionCancelled`
- `SessionDeleted`

Event contracts and versioning remain open decisions.

## 8. Architecture and Boundaries

Sessions is the source of truth for schedule and screening lifecycle.

Reservations consumes session information but owns seat availability and reservation state.

Venue owns physical seat inventory but does not own session timeline.

Catalog owns movie metadata but does not own scheduling.

## 9. Initial Scope

The initial implementation should focus on:

- Manual session creation.
- Tenant-scoped validation of movie and auditorium references.
- Overlap prevention in the same auditorium.
- Session retrieval/listing.
- Session deletion guarded by reservation existence checks.

Additional complexity should be introduced only when required by concrete business workflows.

## 10. Open Decisions

- Exact session lifecycle states and transitions.
- Rules for updating published sessions.
- Grace periods and operational cutoffs before session start.
- Contract details for Catalog and Venue validation APIs.
- Contract details for reservation-existence checks with Reservations.
- Asynchronous event schema and delivery guarantees.
