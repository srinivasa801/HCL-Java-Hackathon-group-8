# Audit Service

Purpose: Centralized logging of transaction and system events.

Models

- AuditEvent
  - `id` (UUID)
  - `source` (string)
  - `type` (string)
  - `payload` (JSON object)
  - `timestamp` (ISO-8601)

Endpoints

- POST `/audit/events` — Log an event
  - Request:
    ```json
    {
      "source":"payment-service",
      "type":"PAYMENT_COMPLETED",
      "payload":{ "paymentId":"UUID", "amount":100.00 }
    }
    ```
  - Response: `201 Created` with event `id`.

- GET `/audit/events` — Query events (filters supported)
  - Response: `200 OK` (array of AuditEvent)

