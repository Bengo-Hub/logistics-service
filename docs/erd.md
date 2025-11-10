# Logistics Service – Entity Relationship Overview

The logistics service coordinates mid-mile and last-mile fulfilment, fleet operations, task orchestration, and integrations with carriers.  
Ent schemas model the domain and power migrations.

> **Conventions**
> - UUID primary keys.
> - `tenant_id` on all operational tables for isolation.
> - Timestamps are `TIMESTAMPTZ`.
> - Geospatial data stored using PostGIS types (`GEOGRAPHY(Point, 4326)` where applicable).

---

## Tenant & Fleet

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `fleets` | `id`, `tenant_id`, `tenant_slug`, `name`, `type`, `status`, `metadata`, `created_at`, `updated_at` | Fleet grouping (internal riders, third-party carriers) keyed by shared tenant slug. |
| `fleet_members` | `id`, `tenant_id`, `fleet_id`, `user_id`, `driver_code`, `status`, `vehicle_id`, `joined_at`, `suspended_at`, `metadata` | Rider/driver membership. Links to identities from `auth-service` via `user_id`; onboarding data originates from food-delivery backend using shared tenant/outlet references. |
| `vehicles` | `id`, `tenant_id`, `fleet_id`, `vehicle_type`, `make`, `model`, `license_plate`, `capacity_json`, `status`, `compliance_status`, `metadata` | Vehicle registry. |
| `vehicle_documents` | `id`, `vehicle_id`, `document_type`, `file_url`, `issued_at`, `expires_at`, `status`, `verified_by`, `verified_at` | Compliance documents. |
| `rider_shifts` | `id`, `tenant_id`, `fleet_member_id`, `shift_start`, `shift_end`, `status`, `created_at`, `updated_at` | Shift planning and attendance. |

## Task Lifecycle

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `tasks` | `id`, `tenant_id`, `external_reference`, `source_service`, `task_type`, `priority`, `status`, `sla_due_at`, `requested_pickup_at`, `requested_dropoff_at`, `metadata`, `created_at`, `updated_at` | Canonical work orders (deliveries, pickups, returns, transfers). |
| `task_steps` | `id`, `task_id`, `step_type`, `sequence`, `location_name`, `address_json`, `geo_point`, `contact_name`, `contact_phone`, `requires_signature`, `requires_photo`, `metadata` | Ordered steps (pickup/drop-off hubs). |
| `task_events` | `id`, `task_id`, `event_type`, `actor_id`, `actor_type`, `payload`, `occurred_at` | State transitions (created, assigned, in-progress, completed, failed). |
| `task_assignments` | `id`, `task_id`, `fleet_member_id`, `assignment_status`, `assigned_at`, `accepted_at`, `declined_at`, `completed_at`, `reason_code`, `metadata` | Mapping tasks to riders/drivers. |
| `task_documents` | `id`, `task_id`, `document_type`, `file_url`, `captured_at`, `captured_by`, `metadata` | Attachments (handover forms, manifests). |

## Routing & Dispatch

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `dispatch_rules` | `id`, `tenant_id`, `name`, `rule_type`, `config_json`, `priority`, `is_active`, `created_at`, `updated_at` | Configurable dispatch/assignment policies. |
| `dispatch_batches` | `id`, `tenant_id`, `batch_code`, `rule_id`, `status`, `requested_at`, `processed_at`, `metadata` | Batch execution of dispatch engine. |
| `routes` | `id`, `tenant_id`, `task_id`, `route_plan_json`, `distance_meters`, `duration_seconds`, `generated_at`, `routing_provider`, `metadata` | Planned route metadata (via OSRM/Mapbox). |
| `route_segments` | `id`, `route_id`, `segment_index`, `polyline`, `eta_start`, `eta_end`, `stop_id` | Granular route steps. |
| `route_metrics` | `id`, `route_id`, `actual_distance_meters`, `actual_duration_seconds`, `deviation_percent`, `updated_at` | Post-completion analytics. |

## Real-Time Telemetry

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `telemetry_streams` | `id`, `tenant_id`, `fleet_member_id`, `device_id`, `started_at`, `ended_at`, `status`, `metadata` | Session-level telemetry bundling. |
| `telemetry_points` | `id`, `stream_id`, `captured_at`, `geo_point`, `speed_kph`, `bearing_deg`, `accuracy_m`, `altitude_m`, `battery_pct`, `metadata` | Individual location samples (compressed for storage). |
| `geo_fence_events` | `id`, `tenant_id`, `fleet_member_id`, `fence_id`, `event_type`, `occurred_at`, `geo_point`, `task_id`, `metadata` | Entry/exit of virtual zones. |
| `telemetry_alerts` | `id`, `tenant_id`, `alert_type`, `severity`, `detected_at`, `resolved_at`, `payload`, `task_id`, `fleet_member_id` | Anomalies (speeding, offline, route deviation). |

## Proof of Delivery & Compliance

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `proof_of_delivery` | `id`, `task_id`, `fleet_member_id`, `signature_url`, `photo_url`, `otp_code`, `captured_at`, `metadata` | Delivery confirmation artifacts. |
| `customer_feedback` | `id`, `task_id`, `rating`, `comments`, `reported_at`, `metadata` | Customer experience capture. |
| `delivery_incidents` | `id`, `task_id`, `incident_type`, `description`, `reported_by`, `reported_at`, `resolution_status`, `resolved_at`, `metadata` | SLA breaches, damages, escalations. |

## Carrier & Marketplace Integrations

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `carrier_partners` | `id`, `tenant_id`, `name`, `provider_code`, `api_credentials_json`, `status`, `metadata`, `created_at`, `updated_at` | Third-party courier integrations. |
| `carrier_jobs` | `id`, `tenant_id`, `carrier_id`, `task_id`, `carrier_reference`, `status`, `cost_amount`, `currency`, `assigned_at`, `completed_at`, `metadata` | Track external carrier jobs. |
| `carrier_callbacks` | `id`, `carrier_job_id`, `event_type`, `payload`, `received_at`, `processed_at`, `status`, `error_message` | Webhook ingestion from carriers. |
| `marketplace_offers` | `id`, `tenant_id`, `task_id`, `carrier_id`, `offer_status`, `price_amount`, `expires_at`, `metadata` | Tendering / marketplace bidding flows. |

## Billing & Earnings

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `earnings_statements` | `id`, `tenant_id`, `fleet_member_id`, `period_start`, `period_end`, `gross_amount`, `net_amount`, `bonus_amount`, `deduction_amount`, `status`, `generated_at`, `metadata` | Rider/driver payouts exported to treasury. |
| `tariff_profiles` | `id`, `tenant_id`, `name`, `base_fare`, `per_km_rate`, `per_minute_rate`, `surcharge_json`, `is_active`, `created_at`, `updated_at` | Tariff rules. |
| `tariff_applications` | `id`, `tariff_profile_id`, `task_id`, `calculated_amount`, `calculated_at`, `metadata` | Fare calculation history. |
| `billing_events` | `id`, `tenant_id`, `task_id`, `event_type`, `amount`, `currency`, `occurred_at`, `metadata` | Events forwarded to `treasury-app` (payout, surcharge, penalty). |

## Reverse & Internal Logistics

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `return_requests` | `id`, `tenant_id`, `original_task_id`, `reason_code`, `status`, `requested_at`, `approved_by`, `approved_at`, `metadata` | Reverse logistics tasks triggered by customers/staff. |
| `waste_pickups` | `id`, `tenant_id`, `warehouse_id`, `scheduled_at`, `completed_at`, `status`, `metadata` | Waste/expired goods collection (links to inventory transfers). |
| `transfer_links` | `id`, `tenant_id`, `logistics_task_id`, `inventory_transfer_id`, `status`, `sync_state`, `metadata` | Bridge between logistics tasks and inventory transfer orders. |

## Integrations & Eventing

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `integration_settings` | `id`, `tenant_id`, `tenant_slug`, `service_code`, `config_json`, `status`, `last_sync_at`, `metadata` | Configuration for food delivery backend, inventory, POS, and third-party carriers. |
| `webhook_subscriptions` | `id`, `tenant_id`, `event_key`, `target_url`, `secret`, `status`, `last_delivery_status`, `retry_count` | Outbound event subscriptions. |
| `outbox_events` | `id`, `tenant_id`, `aggregate_type`, `aggregate_id`, `event_type`, `payload`, `status`, `attempts`, `last_attempt_at`, `created_at` | Reliable event publishing to NATS/Kafka.
| `tenant_sync_events` | `id`, `tenant_id`, `tenant_slug`, `source_service`, `payload`, `synced_at`, `status` | Records inbound tenant/outlet discovery hooks to ensure metadata exists before persisting riders, tasks, or routes. |
| `import_jobs` | `id`, `tenant_id`, `job_type`, `source_url`, `status`, `requested_by`, `started_at`, `completed_at`, `error_message` | Bulk fleet/task imports. |
| `export_jobs` | `id`, `tenant_id`, `job_type`, `parameters`, `status`, `requested_by`, `result_url`, `completed_at` | Data exports for analytics or treasury. |

## Telemetry Configuration

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `geo_fences` | `id`, `tenant_id`, `name`, `fence_type`, `geometry`, `metadata`, `created_at`, `updated_at` | Delivery zones, depots, restricted areas. |
| `device_configs` | `id`, `tenant_id`, `device_type`, `config_json`, `version`, `effective_from`, `effective_to` | Mobile/IoT configuration delivered to riders. |
| `telemetry_rules` | `id`, `tenant_id`, `rule_name`, `rule_type`, `thresholds_json`, `is_active`, `created_at`, `updated_at` | Alert thresholds (offline detection, speeding). |

## Relationships & Integrations

- `tasks` are the source for `task_steps`, `task_events`, `task_assignments`, and `proof_of_delivery`.
- Each `fleet_member` references an identity from `auth-service` (`user_id`) and syncs rider KYC state with the food-delivery backend.
- `tasks.external_reference` ties back to upstream systems (food delivery orders, inventory transfers, POS tickets) using the shared `tenant_slug` and outlet identifiers to guarantee consistency; tenant/outlet records are hydrated via discovery callbacks before tasks are stored.
- `transfer_links` unify logistics movement with `inventory-service` transfer orders to avoid duplicated state.
- `billing_events` and `earnings_statements` integrate with `treasury-app` for payouts and invoicing.
- Outbound events inform:
  - Food delivery backend (`logistics.task.assigned`, `logistics.route.completed`).
  - Inventory service (`logistics.transfer.shipped`).
  - Notifications service (`logistics.driver.arriving`, `logistics.sla.breach`).
  - POS service (curbside pickup readiness).
- Tenant/outlet discovery is captured in `tenant_sync_events`, providing a webhook-driven handshake that avoids polling and ensures metadata consistency before tasks are stored.
- Telemetry alerts can trigger notifications or dispatcher dashboards in near real time.

## Seed & Reference Data

- Demo fleets (`urban_internal`, `third_party`) seeded for Urban Café pilot.
- Default dispatch rules: nearest driver, batch route, and marketplace fallback.
- Sample tariff profiles (standard delivery, express, bulk) for testing treasury integrations.

---

Maintain this document alongside schema updates. After changing Ent schema definitions, run `go generate ./internal/ent` and refresh the ERD to keep integrators aligned.

