---
name: protoforge-api
description: ProtoForge sync licensing API — search tracks, create licenses, generate contracts, manage templates and webhooks programmatically
license: MIT
compatibility:
  - Claude Code
  - Cursor
  - Windsurf
metadata:
  version: "1.0"
  base_url: https://api.protoforge.com/v1
  auth_type: Bearer token
  docs_url: https://docs.protoforge.app
  category: music-rights
  tags:
    - sync-licensing
    - music-rights
    - contracts
    - api
---

# ProtoForge API Skill

ProtoForge automates sync licensing — the process of licensing music for use in film, TV, advertising, and digital content. This skill covers the REST API for searching tracks, creating licenses, generating contracts, managing templates, and handling webhooks.

## Authentication

All requests require a Bearer token in the Authorization header.

```http
Authorization: Bearer pk_test_your_api_key
```

**Key prefixes:**
- `pk_test_` — Test mode. No real contracts created. Isolated from production data.
- `pk_live_` — Production mode. Creates real contracts and triggers signature flows.

Keys are org-scoped. A key for Organization A cannot access Organization B's data.

Get API keys from **Settings > API Keys** in the ProtoForge dashboard.

## API Endpoints

| Method | Path | Description | Rate Limit |
|--------|------|-------------|------------|
| GET | `/v1/tracks/search` | Search tracks by keyword, BPM, key, genre, mood, territory | 100/min |
| GET | `/v1/tracks/{id}` | Get a single track with full rights metadata | 100/min |
| POST | `/v1/licenses` | Create a new sync license | 30/min |
| GET | `/v1/licenses` | List licenses with filtering and pagination | 100/min |
| GET | `/v1/licenses/{id}` | Get a single license by ID | 100/min |
| POST | `/v1/contracts` | Generate a contract from a license and template | 20/min |
| GET | `/v1/contracts/{id}` | Get a contract with status, PDF URL, confidence score | 100/min |
| GET | `/v1/templates` | List available contract templates | 100/min |
| POST | `/v1/templates` | Create a new contract template | 30/min |
| GET | `/v1/templates/{id}` | Get a template with Handlebars body and token schema | 100/min |
| POST | `/v1/webhooks` | Create a webhook subscription | 30/min |
| GET | `/v1/webhooks` | List registered webhooks | 100/min |
| DELETE | `/v1/webhooks/{id}` | Delete a webhook subscription | 30/min |
| POST | `/v1/bulk/licenses` | Bulk create licenses via CSV upload | 5/min |

## Common Agent Mistakes

### 1. Wrong authentication format

```http
# WRONG — missing "Bearer" prefix
Authorization: pk_test_abc123

# CORRECT
Authorization: Bearer pk_test_abc123
```

### 2. Missing required fields on license creation

The `POST /v1/licenses` endpoint requires all of these fields:

- `track_id` (string) — The track to license
- `usage_type` (string) — One of: `film`, `tv`, `advertising`, `digital`, `gaming`, `social_media`
- `territory` (array of strings) — ISO 3166-1 alpha-2 territory codes
- `duration_months` (integer) — License duration in months (1-120)

Optional fields: `exclusive` (boolean, default false), `notes` (string, max 1000 chars).

### 3. Wrong enum values

Status filters use snake_case enum values:

```text
# WRONG
?status=PendingSignature
?status=pending-signature

# CORRECT
?status=draft
?status=active
?status=expired
?status=revoked
```

Valid license statuses: `draft`, `active`, `expired`, `revoked`.

Valid usage types: `film`, `tv`, `advertising`, `digital`, `gaming`, `social_media`.

### 4. Not paginating results

List endpoints return a maximum of 100 items per page. Always check pagination and request additional pages if needed.

```json
{
  "pagination": {
    "page": 1,
    "per_page": 25,
    "total": 250
  }
}
```

Use `?page=2&per_page=25` to fetch the next page.

### 5. Confusing licenses and contracts

Licenses and contracts are separate resources. Create a license first (`POST /v1/licenses`), then generate a contract from it (`POST /v1/contracts` with `license_id` and `template_id`).

## Dashboard vs API

Some operations are only available in the dashboard UI:

| Operation | Dashboard | API |
|-----------|-----------|-----|
| Search tracks | Yes | Yes |
| Create licenses | Yes | Yes |
| Generate contracts | Yes | Yes |
| Manage templates | Yes | Yes |
| Manage webhooks | Yes | Yes |
| Bulk license import | Yes | Yes |
| Manage API keys | Yes | No |
| Configure org settings | Yes | No |
| Manage user roles | Yes | No |

## Rate Limits

Rate limits are enforced per API key:

| Category | Limit | Applies to |
|----------|-------|-----------|
| Search & read | 100 requests/min | All GET endpoints |
| Write | 30 requests/min | POST/PUT for licenses, templates, webhooks |
| Contract generation | 20 requests/min | POST /v1/contracts |
| Bulk operations | 5 requests/min | POST /v1/bulk/licenses |

Rate limit headers are included in every response:

- `X-RateLimit-Limit` — Maximum requests allowed in the window
- `X-RateLimit-Remaining` — Requests remaining in the current window
- `X-RateLimit-Reset` — Unix timestamp when the window resets

When rate limited, the API returns HTTP 429 with the `rate_limited` error code.

## Quick Example: Search, License, Contract

A typical agent workflow: find a track, create a license, and generate a contract.

```typescript
import { ProtoForge } from "@protoforge/sdk";

const pf = new ProtoForge({ apiKey: "pk_test_your_api_key" });

// Step 1: Search for a track
const results = await pf.tracks.search({ query: "sunset boulevard" });
const track = results.data[0];

// Step 2: Create a license
const license = await pf.licenses.create({
  track_id: track.id,
  usage_type: "film",
  territory: ["US", "GB"],
  duration_months: 24,
  exclusive: false,
  notes: "Feature film theatrical release",
});

// Step 3: Generate a contract from the license
const contract = await pf.contracts.generate({
  license_id: license.id,
  template_id: "tmpl_sync_standard_v1",
});

console.log(contract.status); // "draft"
console.log(contract.confidence_score); // 95
```

## Error Response Format

All errors follow this structure:

```json
{
  "error": {
    "code": "validation_error",
    "message": "Rights splits must sum to 100%",
    "details": [
      { "field": "splits", "issue": "Sum is 95%, expected 100%" }
    ]
  }
}
```

| HTTP Code | Error Code | Description |
|-----------|-----------|-------------|
| 400 | `bad_request` | Malformed request body or invalid parameters |
| 401 | `unauthorized` | Missing or invalid API key |
| 403 | `forbidden` | Valid key but insufficient permissions |
| 404 | `not_found` | Resource does not exist or is not in your organization |
| 409 | `conflict` | Resource state conflict (e.g., contract already signed) |
| 422 | `validation_error` | Request understood but failed validation rules |
| 429 | `rate_limited` | Too many requests |
| 500 | `internal_error` | Server error — retry with exponential backoff |
