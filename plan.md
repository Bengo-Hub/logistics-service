## Logistics Service Delivery Plan

### Vision & Mission
- Orchestrate all mid-mile and last-mile logistics for BengoBox properties (food delivery, ecommerce, POS backoffice, inventory transfers) using a single, multi-tenant platform that shares the core `tenant_slug` and outlet registry with every Go microservice.
- Provide real-time visibility over riders/drivers, fleets, tasks, and service-level agreements while integrating tightly with inventory, POS, delivery apps, and treasury.
- Support a wide spectrum of flows: on-demand delivery, scheduled routes, batch consolidation, courier marketplace, third-party carrier integrations, and reverse logistics (returns, waste recovery).
- Offer subscription-aware features (advanced routing, marketplace integrations) aligned with tenant licensing.

### Technical Foundations
- **Language & Runtime:** Go 1.22+ (clean architecture, gofmt, golangci-lint).
- **Data Stores:** PostgreSQL primary (routing plans, tasks, telemetry), Redis for geospatial cache, task queues, rate limiting.
- **ORM & Migrations:** Ent schema definitions, `ent/migrate` for migrations.
- **Messaging:** Outbox pattern -> NATS JetStream/Kafka for events: `logistics.task.created`, `logistics.route.completed`, `logistics.rider.offline`.
- **API Layer:** REST (chi), ConnectRPC/gRPC (stream updates). Swagger/OpenAPI for docs.
- **Geo & Routing:** Integration with Mapbox/Google, OSRM, or custom routing engine. Use geospatial extensions (PostGIS) when needed.
- **Observability:** Prometheus metrics, Grafana dashboards, OpenTelemetry traces, structured logging with zap.
- **Testing:** Unit tests, Testcontainers for Postgres/Redis, routing simulation tests, k6 load tests.
- **Deployment:** Docker/Helm, ArgoCD; multi-env config (dev/stage/prod) with feature flags.

### Core Capability Areas
1. **Tenant & Fleet Management**
   - Tenants, fleets, rider/driver profiles, vehicle types, capacity, compliance documents, all keyed by shared `tenant_slug` and outlet IDs supplied by upstream services. Missing metadata triggers a tenant/outlet discovery webhook back to the originating service (usually auth or food-delivery) before persistence.
   - Integration with auth-service for single sign-on and treasury for licensing.
2. **Task Lifecycle Management**
   - Work order intake from food-delivery backend (rider onboarding, order assignments), inventory, POS, returns—each payload references the shared tenant/outlet metadata to avoid duplication. Status machine (created → assigned → en-route → completed → failed).
   - Multi-leg tasks (pickup → hub → drop-off), SLA timers, escalation rules.
3. **Routing & Dispatch**
   - On-demand dispatch (nearest rider), scheduled batch routes, zone-based dispatch, marketplace/wave assignment.
   - Route optimization (TSP, VRP), capacity-aware scheduling, recurring route templates.
   - Manual override with dispatcher console.
4. **Real-time Tracking & Telemetry**
   - Rider location streaming (via mobile SDK, IoT trackers), geofencing, route adherence checks.
   - Customer/dispatcher dashboards with ETA updates, deviation alerts.
5. **Carrier & Marketplace Integrations**
   - Support third-party carriers (Bolt, Uber, DHL) via connector adapters, tendering, status sync.
   - API for partner fulfilment networks.
6. **Proof of Delivery & Compliance**
   - Capture delivery confirmation (codes, signatures, photos, NFC), temperature logs.
   - Returns/exchange workflows, audit trail for disputes.
7. **Billing & Finance Hooks**
   - Trigger payout calculation (rider earnings, carrier invoices) via treasury app.
   - Track cost per task, surcharges, fines for late/missed deliveries.
8. **Analytics & Operations Intelligence**
   - Routing metrics (on-time %, distance, utilisation), rider performance, heat maps.
   - Operational alerts (idle fleet, SLA breach), scenario planning.
9. **Subscription & Feature Gating**
   - Enforce plan-based access to advanced routing, marketplace connectors, analytics modules.
   - Track usage (tasks, routed km) for billing/overage events.
10. **Reverse & Internal Logistics**
    - Returns management (customer returns, expired goods), waste/logistics pickups, inter-warehouse shuttles.
    - Integration with inventory transfers for pick-up/drop-off scheduling.

### External Integrations
- **Inventory Service:** Pull transfer requests, update stock movements on completion, manage pick waves.
- **POS Service:** Receive order readiness notifications, update driver assignment, handle curbside pickup.
- **Food Delivery Backend / Delivery App:** Provide rider assignments, live ETAs, order status updates. All rider onboarding payloads originate in the UI of services that integrate with this service, hit the individual service backend, and are persisted here under the same tenant/outlet identifiers. Discovery webhooks ensure tenant/outlet data is created before rider data is accepted.
- **Treasury Service:** Payout processing, surge/bonus adjustments, subscription usage.
- **Notifications Service:** Multi-channel alerts (driver assignment, delayed arrival, SLA breach, customer updates).
- **Auth Service:** Central SSO, driver/rider access management, device session control.
- **Mapping APIs:** Mapbox/Google for geocoding, OSRM for routing, Maptiler fallback.
- **Third-party Carriers:** External API connectors (Bolt, Uber, DHL). Extensible integration framework.

### Data Model Highlights
- `fleets`, `fleet_members`, `vehicles`, `vehicle_capacity_profiles`.
- `logistics_tasks`, `task_steps`, `task_events`, `task_metadata`.
- `routes`, `route_stops`, `route_assignments`, `route_metrics`.
- `rider_locations`, `rider_status`, `rider_shift_logs`.
- `proof_of_delivery`, `delivery_assets` (photos, signatures).
- `carrier_connections`, `carrier_jobs`, `carrier_callbacks`.
- `tariff_profiles`, `earnings_statements`, `billing_events`.
- `subscription_entitlements`, `usage_records`.
- `integration_settings`, `webhook_subscriptions`.

### System Architecture & Components
- **Gateway/API:** Authenticated endpoints for internal services, dispatcher UI, mobile rider apps.
- **Dispatch Engine:** service for matching tasks to riders/carriers (Rule-based + optimization + ML future).
- **Routing Service:** wraps external routing APIs, caches results, simulates what-if scenarios.
- **Telemetry Stream:** processes location updates, triggers geofence events, calculates ETA.
- **Workflows/Saga Orchestrator:** handles long-running processes (multi-leg transfer, return flow) ensuring idempotency.
- **Notification Adapter:** central point to push events to notifications service (customer ETA, delays).
- **Integration Layer:** connectors for carriers, mapping, IoT devices. Configurable per tenant.
- **Callback Infrastructure:** all external notifications (task status, tenant discovery, carrier updates) are delivered via signed webhooks with retries—polling is avoided.

### API Strategy
- REST endpoints: `/v1/{tenant}/tasks`, `/routes`, `/fleet`, `/carriers`, `/proof`. Use filters (status, SLA) and pagination.
- Bulk endpoints for import/export (CSV/Excel) to manage route plans, rider rosters.
- Webhooks: `logistics.task.assigned`, `logistics.task.completed`, `logistics.rider.offline`, `logistics.carrier.failure`.
- Streaming updates via WebSocket/SSE/gRPC for dispatcher dashboards and customer apps.
- Admin console & GraphQL (future) for advanced analytics queries.

### Cross-Cutting Concerns
- **Security:** Validate JWTs from auth-service, enforce service-specific scopes (`logistics:dispatch`, `logistics:read`). Support OAuth2 client credentials for service integrations.
- **Multi-tenancy:** `tenant_id` in all tables, row-level security, optional schema isolation for enterprise tenants; outlet/location identifiers mirror those in POS and inventory services to avoid divergent hierarchies. Tenant/outlet discovery webhooks from upstream services (auth, food delivery, POS) hydrate records on first contact.
- **Resilience:** Retry/backoff for carrier APIs, failover to manual dispatch, auto-escalation for SLA risk.
- **Offline Support:** Buffer tasks and telemetry when mobile connection lost; reconcile on reconnect.
- **Compliance:** GDPR (delete/export data), local transport regulations (driver documents, route retention), proof-of-delivery retention policy.
- **Observability:** KPIs (tasks/hour, on-time %, average pickup delay), error budgets, alerting thresholds.

### Delivery Roadmap (Indicative Sprints)
1. **Sprint 0 – Platform Foundations**: repo setup, logging, health checks, ent base schema (tenants, fleet members), CI/CD.
2. **Sprint 1 – Fleet & Rider Management**: CRUD for fleets/riders/vehicles, document storage, integration with auth for SSO.
3. **Sprint 2 – Task Lifecycle MVP**: create/assign/complete tasks, status events, webhooks, auditing.
4. **Sprint 3 – Routing & Dispatch Engine**: basic nearest-driver dispatch, geocoding, ETA calculations, manual override UI scaffolding.
5. **Sprint 4 – Telemetry Streams**: real-time location ingestion (WebSocket/gRPC), geofence alerts, map dashboards.
6. **Sprint 5 – Integration with Inventory/Delivery**: consume inventory transfer requests, update food delivery orders, expose state to POS/delivery apps.
7. **Sprint 6 – Carrier Marketplace Connectors**: generic connector framework, Bolt/Uber stub integrations, failure handling.
8. **Sprint 7 – Billing & Subscription Enforcement**: usage metrics, treasury hooks, plan gating, payout export.
9. **Sprint 8 – Reverse Logistics & Compliance**: returns flows, proof-of-delivery enhancements, regulatory data retention, audit logs.
10. **Sprint 9 – Analytics & Hardening**: dashboards, SLA alerting, high availability, load testing, security review.
11. **Sprint 10 – Launch & Support**: runbooks, on-call readiness, documentation, backlog triage.

### Future Backlog & Enhancements
- Machine learning for demand forecasting, dispatch optimization, and dynamic pricing.
- Incentive engine for riders (gamification, bonuses).
- Drone/robot delivery connectors, EV fleet energy planning.
- IoT integrations (temperature sensors, door sensors) with alerts.
- Carbon footprint tracking, sustainability metrics.
- Marketplace API for external partners to plug-in tasks or capacity dynamically.

### Immediate Next Steps
- Confirm ERD alignment with inventory/POS (shared concepts: warehouse, order IDs).
- Define contract-first API specs for food-delivery and POS interactions.
- Validate mapping provider contracts, start reference implementation (Mapbox + OSRM fallback).
- Produce threat model (location spoofing, assignment fraud) and commence Sprint 0 after stakeholder approval.

