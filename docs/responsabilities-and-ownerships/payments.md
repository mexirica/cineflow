# Payments Service

## 1. Purpose

The Payments service is responsible for processing reservation payments, integrating with Stripe, tracking payment attempts, and communicating payment outcomes to other services.

It provides a reliable record of payment activity for each reservation.

## 2. Responsibilities

Payments owns:

- Payment creation and lifecycle.
- Payment attempt creation and tracking.
- Stripe Checkout integration.
- Stripe webhook processing.
- Payment status management.
- Payment attempt expiration.
- Payment history.
- Payment-related events published through RabbitMQ.
- Administrative read-only payment queries.

Payments does not own:

- Reservation creation or lifecycle.
- Seat availability or reservation expiration rules.
- Ticket pricing or reservation total calculation.
- Customer accounts or authentication.
- Email or SMS notifications.
- Refund processing in the initial scope.

## 3. Multi-tenancy

The platform uses the Pool multi-tenancy model.

- Payments has its own shared PostgreSQL database.
- Each payment and payment attempt belongs to a tenant through `tenant_id`.
- PostgreSQL Row-Level Security (RLS) enforces tenant isolation.
- Every operation must execute within the appropriate tenant context.
- Payments must never access another service's database directly.

The tenant is an independent cinema.

## 4. Payment Provider

Payments integrates with **Stripe**, initially using its sandbox/test mode.

The integration must exercise the actual Stripe payment flow without processing real money.

### Payment method

The initial version supports:

- Credit and debit cards.
- EUR (`EUR`) as the only supported currency.
- Stripe Checkout as the payment interface.

Other payment methods and currencies are outside the initial scope.

### Card data

Cineflow will not collect or store sensitive card details.

Stripe Checkout hosts the payment page and handles card information directly. Payments stores only the identifiers and information necessary to track the payment operation.

## 5. Payment Amount

Reservations is responsible for calculating the reservation's total amount.

It sends the calculated amount and currency to Payments when requesting payment creation.

Payments:

- Does not calculate ticket prices.
- Does not query another service for pricing.
- Does not recalculate the reservation total.
- Stores the amount and currency received for each payment attempt.

The initial supported currency is EUR.

## 6. Payment Model

The service uses two main records:

### Payment

Represents the payment associated with a reservation.

A reservation can have only one Payment record.

It groups the attempts associated with that reservation and provides the overall payment context.

### PaymentAttempt

Represents an individual attempt to process a payment.

Each attempt has its own identifier, status, amount, and Stripe references.

The history of attempts is retained, including failed and expired attempts.

### Initial data model

**Payment**

- `id`
- `tenant_id`
- `reservation_id`
- `created_at`
- `updated_at`

**PaymentAttempt**

- `id`
- `payment_id`
- `tenant_id`
- `amount`
- `currency`
- `status`
- `failure_reason`
- `stripe_checkout_session_id`
- `stripe_payment_intent_id`
- `checkout_url`
- `created_at`
- `completed_at`
- `expires_at`

These are initial fields; the final schema may include additional Stripe references and operational metadata.

## 7. Payment Attempts and Business Rules

A reservation may have multiple payment attempts, subject to the following rules:

- Only one payment can be approved for a reservation.
- Once a payment is approved, new attempts are blocked.
- A declined attempt does not immediately cancel the reservation.
- After three declined attempts, the reservation is cancelled.
- An expired attempt does not count as a declined attempt.
- After an expired attempt, another attempt may be created if the reservation is still valid.
- Once the reservation expires, no further payment attempts are allowed.

The three-decline limit applies to the reservation's payment history.

## 8. Expiration

The reservation and payment attempt use the same expiration period: **15 minutes**.

- Reservations creates a pending reservation with a 15-minute expiration.
- Payments creates the payment attempt with the corresponding expiration.
- Stripe Checkout must be configured with the same expiration deadline.
- An expired attempt cannot be reused.
- Payments publishes `PaymentFailed` with reason `expired`.
- Reservations expires the reservation and releases its seats.

The expiration handling must be idempotent. Duplicate webhook deliveries, expiration processing, or message deliveries must not produce duplicate business effects.

Payments must not accept a successful payment result for a reservation that has already expired. Handling a late Stripe approval, including any required refund, remains an open integration decision.

## 9. Payment Lifecycle

Payment attempts have the following relevant outcomes:

- `pending`: payment is awaiting the Stripe result.
- `approved`: payment was successfully approved.
- `failed`: payment was declined.
- `expired`: the payment attempt reached its expiration deadline.

The final state transition rules must prevent conflicting outcomes, such as approving an attempt that has already expired.

An approved payment is terminal for the reservation's payment process: no further attempts are allowed.

## 10. Integration with Reservations

Payments communicates with Reservations through two mechanisms:

- Synchronous internal API calls for validation.
- RabbitMQ events for payment outcomes.

### 10.1 Starting a payment

Reservations calls the internal Payments API with:

- `reservation_id`
- `tenant_id`
- `amount`
- `currency`

Payments then:

1. Validates the request.
2. Calls Reservations synchronously to verify that the reservation exists, belongs to the tenant, is pending, and has not expired.
3. Verifies that the reservation has not already reached the maximum number of declined attempts.
4. Checks that no payment has already been approved.
5. Creates the Payment or retrieves the existing Payment record.
6. Creates a PaymentAttempt.
7. Creates a Stripe Checkout Session.
8. Stores the Stripe references and expiration.
9. Returns the Checkout URL to Reservations.

Reservations makes the Checkout URL available to the customer.

The API must support idempotency to avoid creating duplicate attempts when a request is repeated.

### 10.2 Reservation validation

Payments must validate the reservation before creating every new attempt.

The validation is performed through the internal Reservations API, not through direct database access.

A payment attempt must not be created if the reservation is invalid, cancelled, expired, or otherwise ineligible for payment.

## 11. Stripe Webhooks

Stripe communicates payment results through webhooks.

Payments is responsible for:

- Receiving Stripe webhook requests.
- Verifying webhook signatures.
- Processing webhook events idempotently.
- Updating the corresponding PaymentAttempt.
- Publishing the appropriate RabbitMQ event.

The browser redirect after Checkout is not proof of payment success.

**The Stripe webhook is the authoritative notification of the payment result.**

Webhook processing must handle duplicate and out-of-order deliveries safely.

## 12. RabbitMQ Events

Payments publishes payment outcomes through RabbitMQ.

Only two event types are included in the initial scope.

### PaymentApproved

Published when a payment attempt is successfully approved.

Reservations consumes this event and confirms the corresponding reservation.

### PaymentFailed

Published when a payment attempt fails or expires.

The event includes a reason that distinguishes the outcomes:

- `declined`
- `expired`

Reservations consumes this event and applies the corresponding reservation rules.

No `PaymentAttemptCreated` event will be published in the initial version.

### Event reliability

Event publishing must be reliable.

The implementation should use an Outbox pattern so that payment state changes and event publication intent are persisted consistently.

Consumers must be idempotent.

## 13. Administrative API

Payments provides an internal administrative read-only API for authorized cinema staff.

It allows staff to:

- Search payments by reservation.
- View payment attempts.
- View payment and