# Payment Service

Purpose: Handle payments; debit payer wallet, credit merchant, apply fees, emit events.

Models

- Payment
  - `id` (UUID)
  - `payerWalletId` (UUID)
  - `merchantId` (UUID)
  - `amount` (decimal)
  - `currency` (string)
  - `fee` (decimal)
  - `netAmount` (decimal)
  - `status` (string: `PENDING` | `COMPLETED` | `FAILED`)
  - `createdAt` (ISO-8601)

- Refund
  - `id` (UUID)
  - `paymentId` (UUID)
  - `amount` (decimal)
  - `status` (string)

Fee integration (included from Fee Service)

Purpose: Calculate dynamic fees by region/currency/rules. The Payment service should call the Fee service to obtain the fee amount before performing wallet debits/merchant credits.

Fee models (for reference)

- FeeConfig
  - `id` (UUID)
  - `region` (string)
  - `currency` (string)
  - `percentage` (decimal)
  - `fixed` (decimal)
  - `effectiveFrom` (date)

Fee endpoints (from Fee Service)

- GET `/fees` — List fee configs
  - Response: `200 OK` (array of FeeConfig)

- GET `/fees/calculate?amount=XX&currency=USD&region=EU` — Calculate fee
  - Response: `200 OK`
    ```json
    {
      "amount":100.00,
      "fee":2.50,
      "netAmount":97.50,
      "currency":"USD"
    }
    ```

Integration notes (how Payment uses the Fee service)

1. When a payment request arrives, Payment service should:
   - Validate request and authorization.
   - Call Fee Service `/fees/calculate` with the `amount`, `currency`, and optional `region` or merchant-specific context.
   - Use returned `fee` and `netAmount` to:
     - Debit the payer wallet for the full `amount`.
     - Credit the merchant with `netAmount`.
     - Persist the `fee` in the Payment record and emit an event for accounting/fee distribution.

2. Example flow (simplified):
   - Client POSTs to Payment service `/payments` with:
     ```json
     {
       "payerWalletId":"<UUID>",
       "merchantId":"<UUID>",
       "amount":100.00,
       "currency":"USD",
       "reference":"order-789",
       "region":"EU"
     }
     ```
   - Payment service calls Fee Service:
     GET `/fees/calculate?amount=100.00&currency=USD&region=EU`
     Response:
     ```json
     {
       "amount":100.00,
       "fee":2.50,
       "netAmount":97.50,
       "currency":"USD"
     }
     ```
   - Payment service proceeds to:
     - Debit payer wallet: -100.00
     - Credit merchant account: +97.50
     - Record fee: 2.50 (for accounting/settlement)

3. Error and edge cases to handle
   - Fee service unavailable: fail-fast with a clear error or use a fallback/default fee configuration.
   - Currency mismatch: validate currencies returned by Fee service match the payment currency.
   - Rounding: apply consistent rounding rules (e.g., 2 decimal places) across services.

Endpoints

- POST `/payments` — Make a payment
  - Request:
    ```json
    {
      "payerWalletId":"UUID",
      "merchantId":"UUID",
      "amount":100.00,
      "currency":"USD",
      "reference":"order-789"
    }
    ```
  - Response: `201 Created`
    ```json
    {
      "id":"UUID",
      "status":"COMPLETED",
      "amount":100.00,
      "fee":2.50,
      "netAmount":97.50
    }
    ```

- POST `/refunds` — Refund a payment
  - Request:
    ```json
    {
      "paymentId":"UUID",
      "amount":50.00,
      "reason":"customer_request"
    }
    ```
  - Response: `200 OK`
    ```json
    {
      "refundId":"UUID",
      "status":"COMPLETED"
    }
    ```

- GET `/payments/{id}` — View payment
  - Response: `200 OK` (Payment model)

Flow notes

- Payment orchestrates: Wallet debit, Fee calculation (Fee Service), Merchant credit, Audit logging, Notification emit, PaymentCompleted event via Kafka/RabbitMQ.

cURL example (including fee call)

```bash
# 1) Calculate fee (Fee Service)
curl "http://localhost:8083/fees/calculate?amount=100.00&currency=USD&region=EU" \
  -H "Authorization: Bearer <token>"

# 2) Create payment (Payment Service)
curl -X POST http://localhost:8080/payments \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"payerWalletId":"<uuid>","merchantId":"<uuid>","amount":100.00,"currency":"USD","reference":"order-789","region":"EU"}'
```
