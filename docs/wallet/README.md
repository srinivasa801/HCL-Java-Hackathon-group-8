# Wallet Service

Purpose: Manage customer wallets and balances.

Models

- Wallet
  - `id` (UUID)
  - `customerId` (UUID)
  - `currency` (string, e.g., "USD")
  - `balance` (decimal)
  - `createdAt` (ISO-8601)
  - `updatedAt` (ISO-8601)

- WalletTransaction
  - `id` (UUID)
  - `walletId` (UUID)
  - `type` (string: `CREDIT` | `DEBIT`)
  - `amount` (decimal)
  - `currency` (string)
  - `reference` (string)
  - `status` (string: `PENDING` | `COMPLETED` | `FAILED`)
  - `createdAt` (ISO-8601)

Endpoints

- POST `/wallets` — Create wallet
  - Request:
    ```json
    {
      "customerId": "UUID",
      "currency": "USD"
    }
    ```
  - Response: `201 Created`
    ```json
    {
      "id": "UUID",
      "customerId": "UUID",
      "currency": "USD",
      "balance": 0.0,
      "createdAt": "2025-01-01T12:00:00Z"
    }
    ```

- GET `/wallets/{id}` — Get wallet
  - Response: `200 OK` (Wallet model JSON)

- POST `/wallets/{id}/credit` — Add funds
  - Request:
    ```json
    {
      "amount": 100.00,
      "currency": "USD",
      "reference": "topup-123"
    }
    ```
  - Response: `200 OK`
    ```json
    {
      "transactionId": "UUID",
      "status": "COMPLETED",
      "balance": 100.00
    }
    ```

- POST `/wallets/{id}/debit` — Remove funds
  - Request:
    ```json
    {
      "amount": 40.00,
      "currency": "USD",
      "reference": "pay-456"
    }
    ```
  - Response: `200 OK`
    ```json
    {
      "transactionId": "UUID",
      "status": "COMPLETED",
      "balance": 60.00
    }
    ```

cURL example

```bash
curl -X POST http://localhost:8085/wallets \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"customerId":"<uuid>","currency":"USD"}'
```

Environment variables (example)

- `SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/wallet_db`
- `SPRING_DATASOURCE_USERNAME=postgres`
- `SPRING_DATASOURCE_PASSWORD=secret`
- `SERVER_PORT=8085`

