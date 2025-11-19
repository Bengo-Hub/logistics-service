# Mapping & Routing Providers – Integration Notes

This guide describes how the Logistics Service integrates with mapping, routing, and road‑condition providers to deliver accurate geocoding, routing, ETAs, and incident‑aware alternative route suggestions.

## Goals
- Reliable geocoding and routing with traffic‑aware ETAs.
- Ability to switch providers at runtime (Google Maps, OSRM/OSM, Maptiler).
- Optional overlays for traffic incidents (accidents, closures, hazards) and speed limits to inform driver safety and ETA models.
- All credentials encrypted at rest (KMS‑backed) and redacted in API responses.

## Providers

### Google Maps Platform
- Services used:
  - Geocoding / Places API (address → coordinates).
  - Routes / Directions API (routing with traffic model; distance matrix for ETAs).
  - Traffic Incidents (where licensed) for accidents, closures, hazards.
  - Roads Speed Limits (where licensed) to contextualize speeding events.
- Authentication: API key or OAuth 2.0 (service account) stored as `provider_credentials`.
- Rate limiting: enforce QPS caps; exponential backoff on HTTP 429/5xx.
- Privacy: do not log full PII; hash or truncate addresses in logs.

### OSRM + OpenStreetMap (OSM)
- Open‑source routing engine backed by OSM data.
- Pros: no per‑request charges; full control, on‑prem deployment.
- Cons: traffic is typically static (no live congestion); pair with incident feeds if needed.
- Deployment: Kubernetes `osrm-backend` with prebuilt planet or regional extracts; update tiles regularly.

### Maptiler / Mapbox
- Tile and routing services with generous free tiers and commercial SLAs.
- Useful for fallback routing/tiling and map rendering in dispatcher UI.

## Configuration

`/v1/{tenant}/integrations/providers` manages per‑tenant settings:

```json
[
  {
    "code": "google_maps",
    "enabled": true,
    "settings": {
      "apiKeyRef": "secret:google_maps_api_key",
      "trafficModel": "best_guess",
      "region": "KE"
    }
  },
  {
    "code": "osrm",
    "enabled": true,
    "settings": {
      "baseUrl": "https://osrm.example.com",
      "profile": "car"
    }
  }
]
```

Secrets (API keys, client secrets) are stored in `provider_credentials` and never returned to clients. Reads from `/providers` redact secret values.

## Error Handling
- 4xx: validation and quota errors → return `400/429` with safe messages; include `errorCode`.
- 5xx: transient provider outage → retry with backoff; fail over to fallback provider if configured.
- Telemetry: export provider latency, success rate, and error rate to Prometheus; alert on sustained degradation.

## Alternative Route Suggestions
- Ingest incident feeds (Google Traffic Incidents where licensed) into `incidents` table.
- On route update, re‑compute candidate alternatives if:
  - Affected segment intersects an active incident, or
  - Forecasted weather severity for a segment exceeds threshold.
- Expose `/v1/{tenant}/routes/{id}/alternatives` with suggested options and reasons; record driver accept/decline.

## Security & Compliance
- All requests to external providers originate from server‑side; never expose raw keys to clients.
- Sign all outbound webhooks to partners; verify inbound signatures.
- Apply data residency policies when choosing provider regions/endpoints.

