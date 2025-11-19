# Logistics Service

The Logistics Service coordinates dispatch, routing, telemetry, and carrier integrations for BengoBox deliveries, inventory transfers, and reverse logistics using the same multi-tenant `tenant_slug` and outlet registry shared across the platform.

## Capabilities

- Fleet and rider management with compliance checks.
- Task lifecycle (create → assign → execute → complete) across delivery, pickup, transfer, and returns.
- Dispatch engine with rules, marketplace tendering, and route optimisation.
- Real-time telemetry ingestion, ETA forecasting, geofence alerts, and anomaly detection.
- Proof-of-delivery capture, incident management, and SLA monitoring.
- Billing hooks for rider earnings, carrier invoices, and surcharge handling.

## Technology

- Go 1.22+, Ent ORM, PostgreSQL (with PostGIS), Redis.
- Routing integrations (OSRM/Mapbox/Google) and geospatial indexing delivered via callback events (no polling).
- REST APIs via `chi`, optional ConnectRPC streams (WebSocket/gRPC).
- Outbox pattern to NATS/Kafka for downstream services.
- Observability: zap logs, Prometheus metrics, OpenTelemetry traces, Grafana dashboards.

## Local Development

```shell
cp config/example.env .env
make deps
docker compose up -d postgres redis osrm
go generate ./internal/ent
go run ./cmd/server
```

The service binds to `http://localhost:4103` by default (configurable via `LOGISTICS_HTTP_PORT`).

## Structure

- `cmd/` – main binaries (`server`, `migrate`, `worker`, `dispatcher`).
- `internal/app` – bootstrap and dependency wiring.
- `internal/ent` – Ent schemas and generated clients.
- `internal/modules` – fleet, dispatch, tasks, telemetry, billing, integrations.
- `docs/` – ERD, ADRs, integration playbooks, incident runbooks.
  - [ERD overview](./docs/erd.md)
  - [Cross‑service ERD alignment](./docs/erd-alignment.md)
  - [API contract (OpenAPI)](./docs/api/openapi.yaml)
  - [Mapping & routing providers](./docs/integrations/mapping-providers.md)
  - [Threat model](./docs/threat-model.md)
  - [Sprint 0 progress](./docs/sprints/sprint-0.md)

## Integrations

- **Food Delivery Backend:** task creation, ETA updates, proof-of-delivery, escalation signals.
- **Inventory Service:** transfer orders, pick wave coordination, warehouse pickups.
- **POS Service:** curbside pickup readiness, in-store dispatch requests.
- **Notifications Service:** customer and rider alerts, SLA breach notifications.
- **Treasury App:** payouts, tariffs, marketplace billing delivered via webhook callbacks (no polling).
- **Auth Service:** rider SSO, device sessions, token validation; tenant/outlet discovery callbacks hydrated here on first login.

Refer to `plan.md` and `docs/erd.md` for the current design blueprint.

## Current Status

- In planning/design phase; schema modelling in progress.
- Major milestones tracked in `CHANGELOG.md`.

