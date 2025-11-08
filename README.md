# E-Wallet Microservices System (Spring Boot + OAuth2)

Overview

- Microservices: Wallet, Payment, Merchant, Fee, Notification, Audit, Settlement.
- Tech: Java 17+, Spring Boot 3, OAuth2 Resource Server (JWT), PostgreSQL, Kafka/RabbitMQ (optional), Docker.
- Each service exposes REST endpoints and Swagger/OpenAPI docs at `/swagger-ui/index.html`.

Quick start (local)

1. Install Java 17, Maven, MYSQL, Docker(Optional).
2. Create mysql for each service.
3. Build each service:

   mvn clean package

4. Run the JARs or use Docker Compose:

   docker-compose up --build

Authentication

- All endpoints should be secured with `Authorization: Bearer <access_token>`.
- Configure `spring.security.oauth2.resourceserver.jwt.jwk-set-uri` to your OIDC provider (e.g., Keycloak).

Services (each under its own folder)

- `wallet/README.md` — Wallet Service
- `payment/README.md` — Payment Service
- `merchant/README.md` — Merchant Service
- `notification/README.md` — Notification Service
- `audit/README.md` — Audit Service


- Service endpoints

- `Base URL` — /api/v1/wallets
- `Base URL` — /api/v1/payments
- `Base URL` — /api/v1/merchants
- `Base URL` — /api/v1/notifications
- `Base URL` — /api/v1/customers


Docker notes

- Each service should include a `Dockerfile` for container builds. Example:

  docker build -t wallet-service .
  docker run -p 8085:8080 wallet-service

License

- HCL Hackathon Use Only
