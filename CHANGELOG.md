# Changelog

This file keeps track of significant changes to the Logistics Service.  
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and [SemVer](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed
- Standardized API base path to `/api/v1` (previously `/v1`)
- Standardized Swagger documentation path to `/v1/docs` (previously `/swagger/*`)
- Updated OpenAPI specification servers to use HTTPS URLs for local development
- Updated Swagger specifications to support both HTTP and HTTPS schemes
- Replaced `http-swagger` with custom Swagger handler that embeds OpenAPI spec and provides protocol-aware URL detection for HTTPS compatibility
- Swagger UI now displays standard header with Explore button and URL input field

### Added
- Authored service delivery plan (`plan.md`) covering scope, architecture, and roadmap.
- Produced ERD (`docs/erd.md`) detailing fleet, task, routing, telemetry, and integration entities with comprehensive cross-service entity alignment.
- Added repository scaffolding: README, contributing guide, code of conduct, security and support docs.
- **Service Bootstrap:** Complete Go service scaffolding with HTTP server, configuration, logging, health endpoints, and Swagger documentation.
- **Auth-Service SSO Integration:** Integrated `shared/auth-client` v0.1.0 library for production-ready JWT validation using JWKS from auth-service. All protected `/v1/{tenant}` routes require valid Bearer tokens. Swagger documentation updated with BearerAuth security definition. Uses monorepo `replace` directives with versioned dependency. See `shared/auth-client/DEPLOYMENT.md` and `shared/auth-client/TAGGING.md` for details.
- **Infrastructure:** PostgreSQL connection pool, Redis caching, NATS event bus integration, Prometheus metrics, structured logging with zap.

### Changed
- Service now uses Go workspace (`go.work`) for local development; production deployments consume `shared/auth-client` as a private Go module.

### Pending
- Ent schema implementation
- CI/CD automation
- Domain-specific handlers and business logic (fleet management, task assignment, routing)

## [2025-01-17] Sprint 0 Documentation & Organization
- Reorganized `plan.md` with numbered sections, full requirements, system analysis, and integration points.
- Updated `plan.md` to link to individual sprint files instead of duplicating task lists, improving maintainability.
- Enhanced ERD (`docs/erd.md`) with comprehensive cross-service entity alignment section:
  - Entity ownership rules (which service owns which entities)
  - Integration patterns (tenant discovery, user identity, inventory transfers, zone dispatch, financial events)
  - ID & reference conventions
  - Zone & dispatch collaboration details
- Deleted duplicate `docs/erd-alignment.md` file (content merged into `docs/erd.md`).
- Created comprehensive sprint documentation files for all 11 sprints:
  - `docs/sprints/sprint-0.md` through `docs/sprints/sprint-10.md`
  - Each sprint file includes: goals, detailed tasks, dependencies, acceptance criteria, progress log, and next sprint preview
- Added API contract skeleton: `docs/api/openapi.yaml` (tasks, routes, provider configs).
- Authored mapping provider integration guide: `docs/integrations/mapping-providers.md` (Google Maps, OSRM/OSM, Maptiler).
- Drafted initial Threat Model: `docs/threat-model.md` (STRIDE, mitigations).
- Updated `README.md` with links to new documentation.
