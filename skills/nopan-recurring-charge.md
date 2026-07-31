---
name: Charge a stored credential (recurring / MIT) with Nopan
description: Use a tokenized stored credential to trigger a recurring or merchant-initiated payment, then reconcile via reports.
api: openapi/nopan-openapi-original.yml
operations: [getAccessToken, chargePayment, getPaymentStatus, searchReports, downloadReport]
---

# Charge a stored credential with Nopan

Nopan offers PSP-agnostic tokenization for recurring and merchant-initiated
transactions (MIT). Once a payer's credential is stored, charge it without another
SCA interaction where the scheme allows.

## Steps

1. **Get an access token** — `getAccessToken` (`POST /auth/token`) with
   `scope=payments:process`.

2. **Charge the token** — `chargePayment` (`POST /payments/charge`). Provide the
   stored-credential reference and amount. Send a unique `Idempotency-Key` (UUIDv4)
   so retries never double-charge.

3. **Confirm outcome** — `getPaymentStatus` (`GET /payments/{transactionId}/status`).
   Terminal states: `APPROVED`, `DECLINED`, `CANCELED`. Listen for the
   `PAYMENT_CAPTURE` webhook instead of polling where possible
   (`asyncapi/nopan-webhooks.yml`).

4. **Reconcile** — `searchReports` (`GET /reports`) by date range, then
   `downloadReport` (`GET /reports/download`) for settlement/fee detail.

## Rules
- A declined charge returns `status: DECLINED` with a `statusInfo.reasonCode`;
  map codes via `errors/nopan-error-codes.yml`.
- Provider-side failures surface as `8001`/`8003`/`8004` (retry / not supported /
  provider error) — treat as transient except `8003`.
- Always include the `Idempotency-Key`; keys are retained 24h.
