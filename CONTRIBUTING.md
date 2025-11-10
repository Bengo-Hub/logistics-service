# Contributing Guide

Thank you for considering a contribution to the Logistics Service! This document outlines the preferred workflow.

## Setup

1. Install Go 1.22+, Docker, and make.
2. Provision PostgreSQL (with PostGIS extension), Redis, and routing engine (OSRM/Mapbox emulator).
3. Configure environment variables using `config/example.env` (coming soon).
4. After updating Ent schema files, run `go generate ./internal/ent`.

## Development Process

1. Branch from `main` (e.g., `feature/dispatch-rules`).
2. Implement changes with clean, focused commits.
3. Run:
   ```shell
   go fmt ./...
   golangci-lint run
   go test ./...
   ```
4. Update documentation (`plan.md`, `docs/erd.md`, README) if you modify behaviour or data structures.
5. Open a pull request describing the changes, testing done, and integration impact.

## Coding Guidelines

- Use clean architecture boundaries (handlers → services → repositories).
- Prefer dependency injection for external services (routing engines, notifications).
- Provide table-driven unit tests; integration tests should leverage Testcontainers.
- Include tracing spans and structured logs for operational visibility.

## Commits & Reviews

- Use descriptive messages (e.g., `dispatch: add marketplace tendering`).
- Reference related tasks/issues where applicable.
- Expect reviewers to ask for ADRs when introducing significant architectural changes.

## Reporting Issues

- Provide reproduction steps, expected behaviour, logs (with IDs), and environment info.
- Tag severity: `bug`, `enhancement`, `ops`, `security`.
- Sensitive/security issues should follow `SECURITY.md`.

## Communication

- Slack: `#bengobox-logistics`.
- Dispatch stand-up: weekdays 09:30 EAT.
- Incident channel: `#logistics-incident`.

We appreciate your help building reliable logistics infrastructure!

