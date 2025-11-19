# Sprint 9 – Analytics & AI Constraints (Weeks 18-19)

**Status**: Not Started  
**Start Date**: TBD  
**Target Completion**: TBD

## Goals

- Build dashboards for on-time %, distance, utilization, SLA alerting, and capacity planning.
- Create simulation UX for what-if scenarios.
- Train lateness/dwell models integrating driver/incident/weather features into constraints engine.
- Optimize route performance with matrix/ETA caching.

## Detailed Tasks

### 10.1 Dashboards & Reporting
- [ ] KPI endpoints:
  - `GET /v1/{tenant}/analytics/kpis` → returns: on-time %, average distance, utilization %, SLA breach count
  - Aggregate from `tasks`, `route_metrics`, `task_events`
- [ ] On-time % calculation:
  - Query: tasks where `completed_at <= sla_due_at` / total completed tasks
  - Time range: daily/weekly/monthly
- [ ] Utilization %:
  - Active driver hours / total driver hours (from `rider_shifts`)
  - Capacity utilization: tasks assigned / max capacity
- [ ] SLA alerting:
  - Real-time: tasks approaching SLA (15min before due) → alert dispatcher
  - Historical: breach rate trends → identify problem routes/drivers
- [ ] Capacity planning:
  - `GET /v1/{tenant}/analytics/capacity` → forecast demand vs available drivers
  - Use historical data to predict peak hours/days

### 10.2 Simulation UX
- [ ] Simulation endpoint:
  - `POST /v1/{tenant}/routes/simulate` → input: task list, constraints, optimization mode
  - Output: optimized route plan, estimated distance/duration/cost, driver assignments
  - Does not create actual tasks (dry-run)
- [ ] What-if scenarios:
  - Compare: current route vs optimized route
  - Parameters: add/remove tasks, change driver, adjust time windows
  - Return: savings (distance, time, cost)

### 10.3 AI-Driven Constraints Engine
- [ ] Lateness model:
  - Train ML model: predict task completion time based on features
  - Features: driver scorecard, historical lateness, route complexity, weather, traffic incidents
  - Output: probability of lateness, expected delay minutes
- [ ] Dwell time model:
  - Predict: time spent at pickup/dropoff location
  - Features: location type (residential/commercial), time of day, driver experience
- [ ] Constraints integration:
  - Use model predictions to adjust ETA calculations (Sprint 3)
  - Penalize routes with high lateness probability
  - Prefer drivers with better scorecards for time-sensitive tasks
- [ ] Model training pipeline:
  - Background job: retrain models weekly using historical data
  - Store model versions in `ml_models` table (if exists) or S3
  - A/B testing: compare model predictions vs actuals

### 10.4 Route Performance Tuning
- [ ] Distance matrix caching:
  - Cache: origin → destination → (distance, duration)
  - Store in Redis with TTL (1 hour for traffic-aware, 24 hours for static)
  - Reduce API calls to routing providers
- [ ] ETA caching:
  - Cache: route → ETA (with traffic conditions)
  - Invalidate on: traffic incident, weather change, route update
- [ ] Performance metrics:
  - `GET /v1/{tenant}/analytics/route-performance` → average deviation, cache hit rate, API call count
- [ ] Optimization:
  - Batch route planning: optimize multiple tasks together (VRP)
  - Use cached matrices to reduce computation time

## Dependencies

- Sprint 2-4 complete (tasks, routes, telemetry exist for analytics)
- ML framework (optional): TensorFlow/PyTorch or simple regression models
- Redis for caching

## Acceptance Criteria

- [ ] Dashboards display accurate KPIs
- [ ] Simulation returns optimized routes without creating tasks
- [ ] Lateness/dwell models predict with reasonable accuracy (±15%)
- [ ] Constraints engine uses model predictions in routing
- [ ] Distance matrix caching reduces API calls by >50%
- [ ] Route performance metrics are tracked

## Next Sprint Preview

Sprint 10 will focus on launch readiness: runbooks, SLOs/alerts, on-call procedures, documentation, tenant onboarding playbooks, and migration/backfill scripts.

