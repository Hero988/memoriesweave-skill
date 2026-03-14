---
name: memoriesweave
description: Create photo memory collections with AI on MemoriesWeave. Use when the user wants to upload photos, design AI layouts, add captions, manage memories, or order print products via the MemoriesWeave API.
metadata:
  version: 1.0.0
  author: Hero988
---

# MemoriesWeave API Skill

Create and manage photo memory collections programmatically using the MemoriesWeave API.

## Setup

1. Create an account at https://memoriesweave.com
2. Go to Settings > API Keys (https://memoriesweave.com/settings/api)
3. Create a new API key and copy it
4. Set the environment variable: `export MEMORIESWEAVE_API_KEY=mw_sk_your_key_here`

## Authentication

All requests require a Bearer token:

```
Authorization: Bearer mw_sk_your_key_here
```

## Base URL

```
https://grandiose-loris-729.convex.site/api/v1
```

## Common Workflows

### 1. List Your Photos

```bash
# Get workspaces
curl $BASE_URL/workspaces -H "Authorization: Bearer $MEMORIESWEAVE_API_KEY"

# List photos in a workspace
curl "$BASE_URL/workspaces/{wsId}/photos?limit=25" \
  -H "Authorization: Bearer $MEMORIESWEAVE_API_KEY"
```

### 2. Create a Memory

```bash
# Create a new memory
curl -X POST "$BASE_URL/workspaces/{wsId}/memories" \
  -H "Authorization: Bearer $MEMORIESWEAVE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"title": "Summer 2025", "description": "Beach vacation photos"}'

# Send a design chat message (AI generates HTML layout)
curl -X POST "$BASE_URL/memories/{memoryId}/design" \
  -H "Authorization: Bearer $MEMORIESWEAVE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"message": "Create a warm, sunny photo book with 12 pages"}'
# Returns: { "data": { "jobId": "...", "pollUrl": "/api/v1/jobs/..." } }

# Poll for completion
curl "$BASE_URL/jobs/{jobId}" -H "Authorization: Bearer $MEMORIESWEAVE_API_KEY"
```

### 3. Full Generation (Photo Select + Content Fill)

```bash
curl -X POST "$BASE_URL/memories/{memoryId}/generate" \
  -H "Authorization: Bearer $MEMORIESWEAVE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Our best family moments from 2025"}'
```

### 4. Get Memory HTML

```bash
curl "$BASE_URL/memories/{memoryId}/html" \
  -H "Authorization: Bearer $MEMORIESWEAVE_API_KEY"
```

## Pricing

- **Free operations:** All GET endpoints, creating/updating/deleting resources
- **Credit operations:** AI design, content generation, photo captioning
- Credits shared with website usage (1 credit = $1 USD)
- Estimated costs: Design chat ~$0.15, Content fill ~$0.10, Photo caption ~$0.005

## Rate Limits

| Plan | General RPM | AI Operations/min |
|------|------------|-------------------|
| Free | 10 | 2 |
| Starter | 30 | 5 |
| Plus | 60 | 10 |
| Pro | 120 | 20 |

Rate limit headers included in every response: `X-RateLimit-Remaining`, `X-RateLimit-Reset`, `X-RateLimit-Limit`.

## Async Operations

AI operations return immediately with a job ID. Poll `GET /jobs/{jobId}` for status:

```json
{
  "data": {
    "status": "completed",
    "result": { ... },
    "actualCredits": 0.12
  }
}
```

Statuses: `pending` → `running` → `completed` | `failed` | `cancelled`

## Error Codes

| Code | Status | Description |
|------|--------|-------------|
| `bad_request` | 400 | Invalid request parameters |
| `unauthorized` | 401 | Missing/invalid API key |
| `forbidden` | 403 | Access denied |
| `not_found` | 404 | Resource not found |
| `rate_limited` | 429 | Too many requests |
| `internal_error` | 500 | Server error |

## Full API Reference

OpenAPI spec: https://memoriesweave.com/api/openapi.json
Interactive docs: https://memoriesweave.com/docs/api
