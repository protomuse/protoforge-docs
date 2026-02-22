---
name: protoforge-api
description: >
  Search music tracks, create sync licenses, generate contracts, and manage
  webhooks via the ProtoForge API. Use when building music licensing integrations
  for film, TV, advertising, or digital content.
license: MIT
compatibility: Claude Code, Cursor, Windsurf, and other AI coding assistants
metadata:
  author: protomuse
  version: "1.0"
---

# ProtoForge API Skill

## Authentication

All requests require a Bearer token in the `Authorization` header:

```
Authorization: Bearer pk_test_your_api_key
```

- **Test keys** (`pk_test_*`) — sandbox environment, sample data, no real licenses
- **Live keys** (`pk_live_*`) — production, real catalogue, binding licenses
- Keys are org-scoped — all resources belong to the authenticated organisation

## API Capabilities

Base URL: `https://api.protoforge.com/v1`

| Endpoint | Method | Description | Rate Limit |
|----------|--------|-------------|------------|
| `/v1/tracks/search` | GET | Search tracks by BPM, key, genre, mood, territory, availability | 100/min |
| `/v1/tracks/{id}` | GET | Get full track details with rights metadata | 100/min |
| `/v1/licenses` | POST | Create a sync license (track_id, usage_type, territory, duration_months required) | 30/min |
| `/v1/licenses/{id}` | GET | Get license details and status | 100/min |
| `/v1/licenses` | GET | List licenses with filtering (status, track_id) and pagination | 100/min |
| `/v1/contracts` | POST | Generate contract from license + template (license_id, template_id required) | 20/min |
| `/v1/contracts/{id}` | GET | Get contract with PDF URL and signature status | 100/min |
| `/v1/templates` | GET | List available contract templates | 100/min |
| `/v1/templates` | POST | Create Handlebars template with token schema (name, body, token_schema required) | 30/min |
| `/v1/templates/{id}` | GET | Get template details and token schema | 100/min |
| `/v1/webhooks` | POST | Subscribe to events (url, events required) | 30/min |
| `/v1/webhooks` | GET | List active webhook subscriptions | 100/min |
| `/v1/webhooks/{id}` | DELETE | Remove a webhook subscription | 30/min |
| `/v1/bulk/licenses` | POST | Upload CSV for batch license creation (multipart/form-data) | 5/min |

## Common Agent Mistakes

1. **Wrong auth header format** — Use `Authorization: Bearer pk_test_...`, not `Authorization: pk_test_...` or `X-API-Key: ...`
2. **Missing required fields on POST /v1/licenses** — All four required: `track_id`, `usage_type`, `territory` (array), `duration_months` (integer 1-120)
3. **Territory as string instead of array** — `territory` must be `["GB"]` not `"GB"`
4. **Invalid usage_type enum** — Must be exactly one of: `film`, `tv`, `advertising`, `digital`, `gaming`, `social_media`
5. **Not paginating search results** — Default `per_page` is 20, max 100. Check `pagination.total`.
6. **Using wrong ISRC format** — ISRC format is `CC-XXX-YY-NNNNN` (with hyphens)
7. **Forgetting Content-Type header** — POST requests need `Content-Type: application/json`

## Dashboard vs API

| Action | API | Dashboard |
|--------|-----|-----------|
| Search tracks | `GET /v1/tracks/search` | Visual search |
| Create license | `POST /v1/licenses` | Wizard flow |
| Generate contract | `POST /v1/contracts` | One-click |
| Manage API keys | — | Settings > API Keys |
| Invite team members | — | Settings > Team |
| View audit trail | — | Audit dashboard |
| Configure billing | — | Settings > Billing |

## Rate Limits

| Category | Limit | Applies To |
|----------|-------|-----------|
| Search | 100/min | Track search and retrieval |
| License | 30/min | License creation |
| Contract | 20/min | Contract generation |
| Bulk | 5/min | CSV batch upload |

Rate limit headers in every response: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`

## Quick Example: Search, License, Verify

```bash
# 1. Search for electronic tracks available in GB
curl -s "https://api.protoforge.com/v1/tracks/search?genre=electronic&territory=GB" \
  -H "Authorization: Bearer pk_test_your_api_key" | jq '.data[0].id'
# Returns: "trk_abc123"

# 2. Create a license
curl -s -X POST "https://api.protoforge.com/v1/licenses" \
  -H "Authorization: Bearer pk_test_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"track_id":"trk_abc123","usage_type":"film","territory":["GB"],"duration_months":12}' | jq '.id'
# Returns: "lic_xyz789"

# 3. Check license status
curl -s "https://api.protoforge.com/v1/licenses/lic_xyz789" \
  -H "Authorization: Bearer pk_test_your_api_key" | jq '.status'
# Returns: "draft"
```

## Error Format

All errors return:

```json
{
  "error": {
    "code": "validation_error",
    "message": "Human-readable description",
    "details": [{ "field": "territory", "issue": "Invalid ISO 3166-1 code: UK" }]
  }
}
```

Error codes: `bad_request`, `unauthorized`, `forbidden`, `not_found`, `conflict`, `validation_error`, `rate_limited`, `internal_error`

## Webhook Events

Subscribe to: `license.created`, `license.approved`, `license.expired`, `contract.generated`, `contract.signed`, `payment.confirmed`, `payment.failed`
