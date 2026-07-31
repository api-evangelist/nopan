---
name: Refund or cancel a Nopan payment
description: Cancel an uncaptured payment or refund a captured one, and confirm via status/events and webhooks.
api: openapi/nopan-openapi-original.yml
operations: [getAccessToken, cancelPayment, refundPayment, getPaymentStatus, getPaymentEvents]
---

# Refund or cancel a Nopan payment

## Decide the operation
- Payment **initiated but not yet captured** → cancel it.
- Payment **already captured** → refund it.

## Steps

1. **Get an access token** — `getAccessToken` (`POST /auth/token`) with
   `scope=payments:process`.

2a. **Cancel** — `cancelPayment` (`POST /payments/cancel`) for an initiated,
   not-yet-captured payment. Include a unique `Idempotency-Key`.

2b. **Refund** — `refundPayment` (`POST /payments/refund`) for a captured payment.
   Supports full or partial amounts. Include a unique `Idempotency-Key`.

3. **Confirm** — `getPaymentStatus` (`GET /payments/{transactionId}/status`) and
   `getPaymentEvents` (`GET /payments/{transactionId}/events`) for the transition
   timeline. Watch the `PAYMENT_REFUND` webhook for the confirmed amount.

## Rules
- Invalid transitions return `5002` (Payment transition not allowed, 422); a
  locked/processing payment returns `5003` (423) — retry shortly.
- `5000` (Payment not found, 404) means the `transactionId` is wrong.
- Errors and remediation: `errors/nopan-error-codes.yml`.
