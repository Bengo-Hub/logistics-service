# Sprint 8 – Reverse Logistics & Compliance (Weeks 16-17)

**Status**: Not Started  
**Start Date**: TBD  
**Target Completion**: TBD

## Goals

- Implement returns/exchanges workflows with hub flows.
- Upgrade PoD with photos/NFC support.
- Add data retention policies, GDPR export/delete endpoints, and compliance reporting.
- Build safety features (no-go zones, HAZMAT skills) with approval workflows.

## Detailed Tasks

### 9.1 Returns & Exchanges
- [ ] Return request creation:
  - `return_requests` table: `tenant_id`, `original_task_id`, `reason_code` (damaged/wrong_item/customer_request), `status`, `requested_at`, `approved_by`, `approved_at`
  - API: `POST /v1/{tenant}/returns` → create return request (from customer or staff)
- [ ] Return task generation:
  - When return approved, create reverse task: `task_type='return'`, link via `original_task_id`
  - Assign to fleet member (same or different)
  - Route: pickup from customer → return to warehouse/outlet
- [ ] Exchange workflow:
  - Similar to return, but includes replacement item in `task_metadata`
  - Create two tasks: return old item, deliver new item
- [ ] Hub flows:
  - `waste_pickups` table: scheduled waste/expired goods collection
  - Link to inventory transfers (inventory service owns transfer, logistics handles movement)

### 9.2 PoD Upgrades
- [ ] Photo PoD:
  - `POST /v1/{tenant}/tasks/{id}/proof/photo` → upload delivery photo
  - Store in S3, URL in `proof_of_delivery.photo_url`
- [ ] NFC PoD:
  - `POST /v1/{tenant}/tasks/{id}/proof/nfc` → NFC tag scan at delivery
  - Validate: NFC tag ID matches customer device or delivery location
  - Store in `proof_of_delivery.nfc_tag_id`
- [ ] Enhanced PoD API:
  - `POST /v1/{tenant}/tasks/{id}/complete` → accepts signature/photo/NFC/OTP
  - All PoD types stored in `proof_of_delivery` table

### 9.3 Data Retention & GDPR
- [ ] Retention policies:
  - Config per tenant: `data_retention_days` (default: 90 days for telemetry, 7 years for financial)
  - Background job: archive or delete old data based on policy
- [ ] GDPR export:
  - `POST /v1/{tenant}/gdpr/export?user_id={id}` → export all data for user (tasks, telemetry, PoD)
  - Returns: JSON/CSV file with all user-related records
  - Store export job in `export_jobs` table
- [ ] GDPR delete:
  - `DELETE /v1/{tenant}/gdpr/delete?user_id={id}` → delete all user data (anonymize or hard delete)
  - Cascade: delete tasks, telemetry, PoD (or anonymize if required for audit)
  - Audit log: record deletion in `audit_logs` table
- [ ] Compliance reporting:
  - `GET /v1/{tenant}/compliance/report` → generate compliance report (SLA breaches, incidents, driver violations)
  - Export to PDF/CSV for regulatory submission

### 9.4 Safety Features
- [ ] No-go zones:
  - `geo_fences` with `fence_type='restricted'` → zones where deliveries are prohibited
  - Validation: reject task assignment if destination in no-go zone
  - Override: require admin approval to deliver to restricted zone
- [ ] HAZMAT skills:
  - `fleet_members.hazmat_certified` → boolean flag
  - `tasks.requires_hazmat` → boolean flag
  - Validation: only assign HAZMAT tasks to certified drivers
- [ ] Approval workflows:
  - `task_approvals` table: `task_id`, `approval_type` (restricted_zone/hazmat/override), `requested_by`, `approved_by`, `approved_at`, `reason`
  - API: `POST /v1/{tenant}/tasks/{id}/approvals` → request approval
  - `POST /v1/{tenant}/approvals/{id}/approve` → approve (requires admin role)

## Dependencies

- Sprint 2 complete (tasks exist for returns)
- Sprint 4 complete (PoD exists for upgrades)
- S3-compatible storage for photo PoD
- Treasury service (optional) for financial retention policies

## Acceptance Criteria

- [ ] Return requests create reverse tasks successfully
- [ ] Photo and NFC PoD are captured and stored
- [ ] GDPR export returns all user data
- [ ] GDPR delete removes/anonymizes user data
- [ ] Compliance reports are generated accurately
- [ ] No-go zones prevent unauthorized deliveries
- [ ] HAZMAT tasks require certified drivers
- [ ] Approval workflows enforce safety rules

## Next Sprint Preview

Sprint 9 will add analytics and AI-driven constraints (dashboards, simulation, lateness/dwell models, route performance tuning).

