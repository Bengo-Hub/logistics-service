# Threat Model – Logistics Service (Initial Draft)

This document outlines adversarial risks and mitigations for the Logistics Service using the STRIDE framework.

## Assets
- Customer PII (names, phone numbers in `task_steps.contact`).
- Location/telemetry data (driver positions, routes, ETAs).
- Provider credentials (maps, weather, smart locks, telematics).
- Financial events (billing events, payouts, expenses).

## STRIDE Analysis

### S – Spoofing Identity
- Risk: Stolen API keys, forged JWTs, fake device identities.
- Mitigations:
  - OAuth 2.0 service accounts; short‑lived tokens; JWT signature validation and audience checks.
  - mTLS for service‑to‑service traffic.
  - Device binding and token revocation; optional device attestation for mobile.

### T – Tampering with Data
- Risk: Altered telemetry (fake locations), modified tasks or routes, webhook payload tampering.
- Mitigations:
  - HMAC‑signed webhooks; nonce and timestamp validation.
  - Immutable event logs; outbox pattern with signature over payload.
  - PostGIS spatial consistency checks (max speed sanity checks, route adherence detection).

### R – Repudiation
- Risk: Users deny actions (e.g., delivery completion).
- Mitigations:
  - Append‑only `task_events` with signer identity (user_id/device_id).
  - Proof‑of‑delivery with signatures/photos/timestamps/geo‑coordinates.

### I – Information Disclosure
- Risk: Leakage of PII, provider credentials, routes.
- Mitigations:
  - Encrypt secrets at rest (KMS); redact secrets on read.
  - Data minimization in logs; field‑level encryption for sensitive fields if required.
  - Role‑based access control and scope‑limited tokens.

### D – Denial of Service
- Risk: Request floods (API, webhooks), pathologically large route computations.
- Mitigations:
  - WAF/rate limiting; KEDA/HPA autoscaling; circuit breakers and timeouts.
  - Backpressure on routing jobs; queue length controls and shedding.

### E – Elevation of Privilege
- Risk: Lateral movement from a compromised service to others; privilege escalation.
- Mitigations:
  - Least‑privilege service accounts; network policies; mTLS.
  - Fine‑grained RBAC for dispatcher vs rider vs ops roles.

## Privacy Considerations (PII)
- Retention policies for customer data; right to access/delete (GDPR/DPA).
- Pseudonymize personal data in event streams; restrict who can query raw telemetry.

## Next Steps
- Validate assumptions with security and privacy teams.
- Add abuse case testing (fake GPS injection, webhook replay).
- Integrate threat checks into CI (static analysis, dependency scanning).

