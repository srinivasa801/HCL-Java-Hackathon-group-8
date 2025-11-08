# Merchant & Settlement Services

This document merges the Merchant and Settlement service documentation. It describes merchant account management, settlement scheduling and processing, data models, endpoints, and integration notes for how these services interact in the e-wallet system.

Overview

- Merchant service: Register and manage merchant profiles and account metadata used for payments and settlements.
- Settlement service: Schedule and process transfers of funds owed to merchants (settlements), typically driven by accumulated net amounts and fees.

Models

- Merchant
  - `id` (UUID)
  - `name` (string)
  - `email` (string)
  - `currency` (string)
  - `merchantAccountId` (string) — external payment/settlement account identifier
  - `settlementSchedule` (string) — e.g., `DAILY`, `WEEKLY`, `MONTHLY`
  - `createdAt` (ISO-8601)

- Settlement
  - `id` (UUID)
  - `merchantId` (UUID)
  - `amount` (decimal)
  - `currency` (string)
  - `status` (string: `PENDING` | `SCHEDULED` | `PROCESSING` | `SETTLED` | `FAILED`)
  - `scheduledAt` (date/time)
  - `processedAt` (date/time, optional)
  - `reference` (string, optional)

Database design notes (high level)

- merchants table
  - id UUID PK
  - name VARCHAR
  - email VARCHAR (unique)
  - currency CHAR(3)
  - merchant_account_id VARCHAR
  - settlement_schedule VARCHAR
  - created_at TIMESTAMP

- settlements table
  - id UUID PK
  - merchant_id UUID FK -> merchants.id
  - amount NUMERIC(18,2)
  - currency CHAR(3)
  - status VARCHAR
  - scheduled_at TIMESTAMP
  - processed_at TIMESTAMP NULLABLE
  - reference VARCHAR
  - created_at TIMESTAMP

- Optional: settlement_items table to record which payments/fees are part of a settlement
  - id UUID PK
  - settlement_id UUID FK -> settlements.id
  - payment_id UUID
  - amount NUMERIC(18,2) (net amount credited to merchant for this payment)
  - fee NUMERIC(18,2)

Endpoints - Merchant

- POST `/merchants` — Register merchant
  - Request:
    ```json
    {
      "name":"ACME Store",
      "email":"merchant@example.com",
      "currency":"USD",
      "settlementSchedule":"DAILY"
    }
    ```
  - Response: `201 Created`
    ```json
    {
      "id":"UUID",
      "name":"ACME Store",
      "email":"merchant@example.com",
      "currency":"USD",
      "settlementSchedule":"DAILY"
    }
    ```

- GET `/merchants/{id}` — Merchant details
  - Response: `200 OK` (Merchant model)

- PUT `/merchants/{id}` — Update merchant (e.g., change settlement schedule)
  - Request: partial merchant fields

Endpoints - Settlement

- POST `/settlements` — Initiate/ schedule a settlement for a merchant
  - Request:
    ```json
    {
      "merchantId":"UUID",
      "amount":1000.00,
      "currency":"USD",
      "schedule":"2025-10-01",
      "reference":"settlement-20251001"
    }
    ```
  - Response: `201 Created`
    ```json
    {
      "id":"UUID",
      "merchantId":"UUID",
      "amount":1000.00,
      "currency":"USD",
      "status":"PENDING",
      "scheduledAt":"2025-10-01T00:00:00Z"
    }
    ```

- GET `/settlements/{id}` — Check settlement status
  - Response: `200 OK` (Settlement model)

- GET `/merchants/{id}/settlements` — List settlements for a merchant (filters by date/status)

Integration and flow notes

1. Payment -> Merchant relation
   - Each completed Payment contains `merchantId` and `netAmount` (amount after fees).
   - Payments should be recorded (or referenced) so settlement batches can group eligible netAmounts.

2. Settlement creation
   - Periodic job (scheduler) or manual trigger queries completed payments since last settlement for a merchant, aggregates net amounts, and creates a `Settlement` record with status `SCHEDULED` or `PENDING`.
   - A Settlement may reference multiple payments via `settlement_items` for auditability.

3. Processing settlements
   - A settlement worker/process picks up `PENDING` or `SCHEDULED` settlements, marks them `PROCESSING`, calls the external payout provider using `merchantAccountId`, and upon success updates status to `SETTLED` and sets `processedAt`.
   - On failure, mark `FAILED` and include a retry or escalation policy.

4. Accounting
   - Fees collected should be stored separately and reconciled — settlement only transfers merchant share (net amounts).
   - Emit events (e.g., `SettlementInitiated`, `SettlementCompleted`, `SettlementFailed`) to the message bus for ledger/audit services.

Example flows

- Daily scheduled settlements (typical):
  1. Scheduler runs at 00:05 daily and gathers all payments with status `COMPLETED` and not yet included in a settlement.
  2. For each merchant, sum `netAmount` and create a `Settlement` with `status: SCHEDULED`.
  3. A processing worker picks up scheduled settlements and executes payouts.

- Manual settlement trigger:
  - Admin calls POST `/settlements` with desired `merchantId` and `amount`.

Security and error handling

- All endpoints must be authenticated and authorized; only payment/settlement services or admin roles may call settlement endpoints.
- Validate currency and merchant account presence before attempting processing.
- Provide idempotency keys for settlement creation to prevent double processing.

Notes

- This document is a merged, canonical reference for merchant and settlement behaviour. Keep the `merchant` and `settlement` services aligned if you split them into separate microservices — workloads can remain separate but data models must be consistent.


