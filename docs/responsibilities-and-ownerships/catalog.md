# Catalog Service

## 1. Purpose

The Catalog Service is responsible for managing the movie catalog of each cinema on the platform.

Each cinema owns its own catalog and can create, retrieve, update, and delete movies.

The service is multi-tenant and follows the Pool strategy, sharing a PostgreSQL database across all cinemas.

## 2. Responsibilities

The Catalog Service is responsible for:

- Manual movie registration.
- Movie retrieval and updates.
- Management of movie information.
- Management of genres, age ratings, directors, and cast information.
- Movie deletion, respecting dependencies with other services.
- Providing movie information for queries and searches.

## 3. Out of Scope

The Catalog Service is not responsible for:

- Cinema registration and tenant lifecycle management (Admin).
- Physical auditoriums and seats (Venue).
- Screening schedules, showtimes, and seat availability (Reservations).
- Reservations and payments.
- Authentication and user management.

## 4. Initial Movie Model

The proposed initial model contains:

| Field | Description |
|---|---|
| `id` | Movie identifier |
| `tenant_id` | Owning cinema |
| `title` | Movie title |
| `normalized_title` | Normalized title used for duplicate detection |
| `synopsis` | Movie synopsis |
| `duration` | Movie duration |
| `release_date` | Release date |
| `created_at` | Creation timestamp |
| `updated_at` | Last update timestamp |

Other information, such as genres, age ratings, directors, and cast members, is part of the functional scope, but its data representation has not yet been defined.

## 5. Business Rules

### 5.1 Uniqueness

A movie cannot be registered more than once in the same cinema's catalog.

Duplicate detection considers:

- Cinema (`tenant_id`);
- Normalized title (`normalized_title`);
- Release date (`release_date`).

Proposed constraint:

`UNIQUE (tenant_id, normalized_title, release_date)`

Different cinemas can register the same movie.

A cinema can register movies with the same title, provided they have different release dates.

### 5.2 Registration

Movie registration will be exclusively manual.

There will be no integration with external movie databases.

### 5.3 Deletion

Movie deletion will be permanent (hard delete).

Before deleting a movie, the Catalog Service must query the Reservations Service to check whether any screenings reference it.

- If the movie is being used, deletion must be rejected.
- If the movie is not being used, the record can be deleted.

The Catalog Service must not directly access the Reservations database.

Coordination between the usage check and deletion must prevent race conditions that could create invalid references.

## 6. Multi-Tenancy

The service will use the Pool strategy:

- A shared PostgreSQL database across tenants.
- Records belonging to cinemas will contain a `tenant_id`.
- PostgreSQL Row-Level Security (RLS) will enforce tenant isolation.
- Tenant context must be reliably propagated across operations.

## 7. Search

PostgreSQL will initially handle queries and searches.

In the future, Elasticsearch or OpenSearch may be evaluated to provide more advanced search capabilities.

PostgreSQL will remain the source of truth.

Integration with external search engines is not part of the initial scope.