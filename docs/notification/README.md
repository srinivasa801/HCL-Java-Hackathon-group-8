# Notification Service

Purpose: Send notifications (email/SMS) — mocked for dev.

Models

- Notification
  - `id` (UUID)
  - `to` (string)
  - `type` (string: `EMAIL` | `SMS`)
  - `subject` (string)
  - `body` (string)
  - `status` (string: `SENT` | `FAILED`)

Endpoints

- POST `/notifications` — Send notification
  - Request:
    ```json
    {
      "to":"user@example.com",
      "type":"EMAIL",
      "subject":"Payment received",
      "body":"Your payment of $100 was successful."
    }
    ```
  - Response: `202 Accepted`
    ```json
    {
      "id":"UUID",
      "status":"SENT"
    }
    ```

