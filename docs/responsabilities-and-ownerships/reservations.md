# Reservations Service

## 1. Purpose

The Reservations Service manages customer reservations, screening-specific seat availability, temporary seat holds, and the reservation lifecycle.

It is responsible for ensuring that seats cannot be reserved by multiple customers for the same session and for managing reservation creation, expiration, cancellation, and payment confirmation.

## 2. Responsibilities

The service owns:

- Reservation creation and management.
- Screening-specific seat availability.
- Temporary seat holds.
- Reservation expiration.
- Reservation cancellation.
- Reservation confirmation after payment approval.
- Reservation history.
- Customer reservation lookup.

### Out of scope

- Tenant registration and lifecycle — Admin.
- Physical auditoriums and seats — Venue.
- Movie catalog and movie information — Catalog.
- Session creation, scheduling, and management — Sessions.
- Payment processing and refunds — Payments.
- Email and SMS delivery — Notifications.
- User authentication and account management.

## 3. Multi-tenancy

The service follows the Pool tenancy strategy.

- All tenants share the same PostgreSQL database.
- Every tenant-owned record contains `tenant_id`.
- PostgreSQL Row-Level Security (RLS) enforces tenant isolation.
- Tenant context must be established for each request and background operation.

The service does not own tenant registration or tenant lifecycle.

## 4. Sessions Integration

Sessions is responsible for creating and managing cinema sessions.

Reservations only consumes session information and manages seat availability and reservations associated with each session.

A reservation references a session through `session_id`.

Reservations must not access the Sessions database directly. Session information and validation must be obtained through the Sessions API or agreed integration mechanisms.

### Responsibilities of Sessions

- Manual session creation.
- Movie and auditorium association.
- Scheduling validation.
- Prevention of overlapping sessions in the same auditorium.
- Session immutability after creation.
- Session deletion when no reservations exist.

These responsibilities do not belong to Reservations.

### Session deletion

Sessions must verify with Reservations whether a session has reservations before deleting it.

This verification must happen through an API or another agreed service integration, never through direct database access.

## 5. Reservations

### 5.1 Reservation rules

- Each reservation belongs to exactly one session.
- A reservation can contain multiple seats.
- There is no predefined maximum number of seats per reservation.
- All seats in a reservation must belong to the same session.
- Seat selection cannot be changed after reservation creation.
- If the customer wants different seats, they must cancel the existing reservation and create another one, subject to availability.
- A reservation is not associated with a registered user account.
- Each reservation has a unique identification code.
- Email and phone number are mandatory when creating a reservation.

### 5.2 Customer identification

Customers use the following information to consult or manage a reservation:

- Reservation code.
- Email address.
- Phone number.

The service does not require a `user_id`.

### 5.3 Reservation lifecycle

A reservation can have the following states:

| Status | Description |
|---|---|
| `pending` | Reservation created, awaiting payment confirmation. |
| `confirmed` | Payment approved and confirmation event received. |
| `cancelled` | Reservation cancelled by the customer or due to payment failure. |
| `expired` | Pending reservation expired without payment confirmation. |

### 5.4 State transitions

- `pending` → `confirmed`: payment approved.
- `pending` → `cancelled`: customer cancellation or payment failure.
- `pending` → `expired`: expiration timeout.
- `confirmed` → `cancelled`: customer cancellation within the permitted cancellation window.

`cancelled` and `expired` are terminal states.

A reservation cannot be confirmed before the payment approval event is received.

## 6. Seat Availability and Holds

Reservations owns seat availability for each individual session.

Venue owns the physical seat inventory, while Reservations manages the availability of those seats within a specific session.

### Rules

- A seat cannot have multiple active reservations for the same session.
- Seat holds prevent concurrent customers from reserving the same seat.
- A reservation containing multiple seats must be handled atomically: all requested seats are reserved or none.
- Seats are released immediately when a reservation is cancelled.
- Seats are released when a pending reservation expires.
- Seats from a confirmed reservation are released immediately after a permitted cancellation.

### Current implementation

The existing Reservations implementation uses Redis for temporary seat holds.

- Seat hold TTL: 10 minutes.
- Pending reservation expiration: 15 minutes.

The difference between these two timeouts still needs to be reviewed.

## 7. Cancellation Rules

### Pending reservations

- Customers can cancel pending reservations at any time.
- Cancellation immediately releases the reserved seats.

### Confirmed reservations

- Customers can cancel confirmed reservations until one hour before the session starts.
- After that deadline, cancellation is rejected.
- Cancellation immediately releases the reserved seats.
- Refund processing is not owned by Reservations and must be handled by Payments.

### Expired and cancelled reservations

- Expired reservations cannot be cancelled again.
- Cancelled reservations cannot be cancelled again.

## 8. Payments Integration

Payments is responsible for processing payments.

Reservations does not confirm a reservation based on a payment request or an assumption that payment succeeded. Confirmation happens only after receiving the payment approval event.

### Payment approved

1. Payments processes the payment.
2. Payments publishes a payment approval event through RabbitMQ.
3. Reservations consumes the event.
4. Reservations changes the reservation from `pending` to `confirmed`.

### Payment failed

1. Payments processes the payment.
2. Payments publishes a payment failure event through RabbitMQ.
3. Reservations consumes the event.
4. Reservations changes the reservation to `cancelled`.
5. Reservations immediately releases the seats.

Event consumers must be idempotent to handle duplicate message deliveries safely.

The project assumes the payment flow will deliver the approval event while the reservation is still pending. Handling late approval after expiration is not part of the initial business scope.

## 9. Reservation History

Reservations are retained indefinitely.

- Cancelled reservations remain stored.
- Expired reservations remain stored.
- Reservation codes remain valid for historical consultation.
- Reservation history is preserved.
- Soft delete is not required initially.
- No automatic historical data deletion is planned.

The existing implementation also maintains reservation state history.

## 10. Architecture and Integrations

Reservations must not access other services' databases directly.

| Service | Integration |
|---|---|
| Admin | Tenant context and identity references, when needed. |
| Venue | Validate physical seat references through its API. |
| Sessions | Validate session references and retrieve session information through its API. |
| Catalog | No direct integration required for reservation operations; movie information belongs to Catalog and is managed through Sessions. |
| Payments | RabbitMQ events for payment approval and failure. |
| Notifications | Reservation-related events can be consumed for customer communications. |

RabbitMQ is the selected message broker for inter-service asynchronous communication.

PostgreSQL is the source of truth for reservation records. Redis is used for temporary seat holds.

## 11. Existing Implementation

The uploaded Reservations implementation already contains several relevant mechanisms:

- Pending reservation creation.
- Redis-based seat holds.
- Pending reservation expiration.
- Expiration worker.
- Prevention of concurrent active reservations for the same seat and session.
- Reservation state history.
- Idempotency support.
- Event publishing infrastructure.

The existing code includes a `confirmed` state, but the payment confirmation flow still needs to be completed.

The business rules in this document represent the agreed target behavior and do not imply that every item is already implemented.

## 12. Open Decisions

The following points still require definition:

1. **Seat hold and expiration:** reconcile the current 10-minute Redis hold with the 15-minute pending reservation expiration.
2. **Refund policy:** define the behavior in Payments when a confirmed reservation is cancelled.
3. **Payment initiation:** define how Reservations requests payment and associates payment attempts with reservations.
4. **RabbitMQ contracts:** define the exact schemas and identifiers for payment approval and failure events.
5. **Message failures:** define retry and dead-letter handling.
6. **Session start:** define what happens to pending reservations when the session starts.
7. **Venue changes:** define how to handle physical seats that become inactive or are removed after reservations have been created.
8. **Reservation API:** define endpoints and request/response contracts.
9. **Concurrency and consistency:** define transaction boundaries and consistency guarantees between PostgreSQL and Redis.
10. **Sessions integration:** define the exact API contract for validating session references and retrieving session information.

## 13. Initial Scope

The initial implementation should focus on:

- Reservation creation with multiple seats.
- Session reference validation through Sessions.
- Atomic seat reservation.
- Temporary seat holds.
- Pending reservation expiration.
- Payment event consumption.
- Reservation confirmation.
- Customer cancellation.
- Reservation lookup using code, email, and phone.
- Tenant isolation through PostgreSQL RLS.

Additional complexity should only be introduced when required by an actual business rule or integration.