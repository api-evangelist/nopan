---
name: Accept an account or wallet payment with Nopan
description: Authenticate, initiate a payment, complete SCA, finalize, and capture funds using the Nopan API.
api: openapi/nopan-openapi-original.yml
operations: [getAccessToken, initiatePayment, finalizePayment, capturePayment, getPaymentStatus]
---

# Accept a payment with Nopan

Nopan is a specialized PSP for European account and wallet payments (Wero, BLIK, Bizum, Satispay, IRIS). Use this skill to take a one-off payment end to end.

## Prerequisites
- An organization account (`client_id`) issued at onboarding.
- A valid mTLS client certificate — every request runs over mutual TLS.
- Requests are signed with JWS.

## Steps

1. **Get an access token** — `getAccessToken` (`POST /auth/token`).
   Send `grant_type=client_credentials`, your `client_id`, and `scope=payments:process`
   (default is `payments:read`). Use the returned Bearer JWT `access_token` on all
   subsequent calls until `expires_in` elapses.

2. **Initiate the payment** — `initiatePayment` (`POST /payments/initiate`).
   Supply the `processingAccountId`, amount, and payment method. Send a unique
   `Idempotency-Key` header (UUIDv4). The response returns a redirect URL for the
   payer's SCA and a `transactionId`. State becomes `PENDING` awaiting SCA.

3. **Finalize** — `finalizePayment` (`POST /payments/finalize`) after the payer
   completes SCA, to confirm the two-step payment.

4. **Capture funds** — `capturePayment` (`POST /payments/capture`) to capture the
   finalized payment. Reuse a fresh `Idempotency-Key` per mutating call.

5. **Check status** — `getPaymentStatus` (`GET /payments/{transactionId}/status`)
   or `getPaymentEvents` for the full timeline.

## Rules
- Idempotency: send `Idempotency-Key` on every mutating call; duplicates within 24h
  return the original result (see `conventions/nopan-conventions.yml`).
- Errors arrive in the status envelope (`statusInfo.reasonCode` + `message` + `callId`).
  Handle `4061` (token expired → re-auth), `5001` (already exists), `429`/`3000`
  (rate limited → back off). Full list: `errors/nopan-error-codes.yml`.
- Test first against the sandbox via `mockPayment` (`POST /mock/set`).
