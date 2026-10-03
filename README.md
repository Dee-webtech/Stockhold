# Stockhold

Stockhold is a multi-tenant inventory reservation API that stops online stores from overselling during flash sales.

## The problem

Two customers click "buy" on the last item at the same moment. Both succeed, and one order has to be cancelled. Stockhold fixes this by holding stock for a customer before the order is final.

## How it works

1. A store calls `reserve(sku, qty)`, and Stockhold holds that stock for a time limit (10 minutes by default).
2. The store then **confirms** the order, which permanently deducts the stock, or **cancels** it.
3. If neither happens in time, the hold **expires** and the stock returns to available.

The design goals are:

- NO OVERSELLING - A reservation is one atomic conditional SQL `UPDATE`, so two buyers can never take the same unit.
- SAFE RETRIES - Mutating calls need an `Idempotency-Key`, so a retried request never reserves twice.
- SAFE SCALING - Several identical instances can run at once. Expiry uses `SELECT ... FOR UPDATE SKIP LOCKED`, so no hold is released twice.
- TENANT ISOLATION - Every table carries a `tenant_id`, and API keys are stored only as SHA-256 hashes.

## Stack

| Part | Technology |
| --- | --- |
| API | Java 21, Spring Boot 4.1, Maven |
| Database | PostgreSQL 16, Flyway migrations |
| Tests | JUnit 5, Testcontainers, k6 |
| SDK | TypeScript, Vitest |
| Dashboard | React, Vite, Tailwind, TanStack Query |
| CI | GitHub Actions |

## Quick start

Requirements: JDK 21, Docker Desktop.

```bash
cp .env.example .env
docker compose up -d
cd api
export DB_PASSWORD=change-me-locally
./mvnw spring-boot:run
```

Check that it is running:

```bash
curl http://localhost:8080/actuator/health
```

Run the tests (Docker must be running, because tests start a real Postgres):

```bash
cd api
./mvnw test
```

## Roadmap

- [x] Repository, branch protection, CI
- [x] Postgres via Docker Compose, API skeleton, health endpoint
- [ ] Database schema (tenants, API keys, inventory, reservations, idempotency)
- [ ] API-key authentication and tenant isolation
- [ ] Inventory and reservation endpoints
- [ ] Confirm, cancel and expiry sweeper
- [ ] Concurrency tests (500 buyers, 100 units)
- [ ] TypeScript SDK and React dashboard
- [ ] Load test results