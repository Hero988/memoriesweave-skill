---
name: memoriesweave
description: Create photo memory collections with AI on MemoriesWeave. Use when the user wants to upload photos, design AI layouts, add captions, manage memories, or order print products via the MemoriesWeave API.
metadata:
  version: 1.6.0
  author: Hero988
---

# MemoriesWeave API Skill

Create and manage photo memory collections programmatically using the MemoriesWeave API. You design the HTML layouts yourself — the API gives you access to photos, conversations, workspace context, and people data.

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
https://grandiose-loris-729.eu-west-1.convex.site/api/v1
```

## CRITICAL: Required Workflow — Always Gather Context First

Before creating any memory or selecting any photos, you MUST follow this workflow:

### Step 1: Get Workspace Context
Fetch the workspace description, relationship info, and general instructions. This tells you WHO the people are, what their relationship is, and how the user wants content presented.

```bash
curl "$BASE_URL/workspaces/{wsId}/context" -H "Authorization: Bearer $KEY"
```

Returns:
```json
{
  "data": {
    "description": "Photos with my relationship with Andrea...",
    "relationshipInfo": "Suliman and Andrea - boyfriend and girlfriend until 30/01/2026...",
    "generalInstructions": "Always mention Andrea and Suliman if you see them..."
  }
}
```

### Step 2: Get Registered People
Fetch the people registered in the workspace with their roles and descriptions.

```bash
curl "$BASE_URL/workspaces/{wsId}/persons" -H "Authorization: Bearer $KEY"
```

Returns:
```json
{
  "data": [
    { "id": "...", "name": "Andrea", "role": "Girlfriend now ex girlfriend best friend", "description": "Girlfriend until 30/01/2026..." },
    { "id": "...", "name": "Suliman", "role": "Boyfriend...", "description": "..." }
  ]
}
```

### Step 3: Select Photos WITH Conversation Context
When selecting photos for a memory, do NOT just look at tags. For each candidate photo, check the conversation context around it to understand what was happening when it was shared.

```bash
# Get conversations around a specific photo
curl "$BASE_URL/workspaces/{wsId}/conversations/by-photo/{photoId}" \
  -H "Authorization: Bearer $KEY"
```

This returns the WhatsApp messages before and after the photo was shared, giving you context about the moment — who was there, what they were talking about, the emotional context.

### Step 4: Choose the Right Digital Format
Before creating the memory, check available formats with their dimensions:

```bash
curl "$BASE_URL/digital-formats" -H "Authorization: Bearer $KEY"
```

**Common formats with multi-page support:**
| Format | Dimensions | Max Pages |
|--------|-----------|-----------|
| Phone Wallpaper (iPhone 16 Pro Max) | 1320x2868 | 10 |
| Phone Wallpaper (iPhone 15 Pro) | 1290x2796 | 10 |
| Desktop Wallpaper (4K UHD) | 3840x2160 | 8 |
| Social Media Post (Instagram) | 1080x1080 | 10 |
| Story/Reel (Instagram/TikTok) | 1080x1920 | 12 |
| Greeting Card | 1920x1280 | 4 |
| Photo Collage | 3000x3000 | 6 |

### Step 5: Create Memory with Format
Create the memory with the correct digital format and page count:

```bash
curl -X POST "$BASE_URL/workspaces/{wsId}/memories" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Summer Memories",
    "description": "Our summer adventures together",
    "digitalFormat": {
      "category": "phone_wallpaper",
      "widthPx": 1290,
      "heightPx": 2796,
      "outputFormat": "png",
      "pageCount": 8
    }
  }'
```

### Step 6: Design and Push HTML
You design the HTML yourself. Each page should be wrapped in a `<div data-mw-page="N">` element. The HTML must be self-contained with inline styles.

For multi-page memories, structure as:
```html
<div data-mw-page="1" style="width:1290px;height:2796px;position:relative;overflow:hidden;">
  <!-- Page 1 content with photo background, text overlays, etc. -->
</div>
<div data-mw-page="2" style="width:1290px;height:2796px;position:relative;overflow:hidden;">
  <!-- Page 2 content -->
</div>
```

Push the HTML to the memory:
```bash
curl -X PATCH "$BASE_URL/memories/{memoryId}" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"customHtml": "<your HTML here>"}'
```

### Step 7: ALWAYS Save a Snapshot After Creating or Changing a Memory

**This is MANDATORY.** Every time you create a new memory and push HTML, or modify an existing memory, you MUST save a snapshot so the user can restore previous versions.

**After creating a new memory:**
```bash
curl -X POST "$BASE_URL/memories/{memoryId}/snapshots" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"label": "Initial design — 10-page iPhone wallpapers"}'
```

**Before modifying an existing memory:**
```bash
# 1. Save a "before" snapshot
curl -X POST "$BASE_URL/memories/{memoryId}/snapshots" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"label": "Before update — original 10-page design"}'

# 2. Make your changes
curl -X PATCH "$BASE_URL/memories/{memoryId}" ...

# 3. Save an "after" snapshot
curl -X POST "$BASE_URL/memories/{memoryId}/snapshots" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"label": "After update — added new pages"}'
```

**List snapshots:**
```bash
curl "$BASE_URL/memories/{memoryId}/snapshots" -H "Authorization: Bearer $KEY"
```

**Restore a previous version:**
```bash
curl -X POST "$BASE_URL/memories/{memoryId}/snapshots/{snapshotId}/restore" \
  -H "Authorization: Bearer $KEY"
```

## Resizing a Memory

When resizing page dimensions, you MUST update BOTH the HTML and the memory's `digitalFormat` so the website's resize dialog reflects the correct size.

```bash
# 1. Resize the HTML (replace width/height in all data-mw-page divs, scale fonts/padding proportionally)
# 2. Push both the resized HTML AND the new digitalFormat in a single PATCH:
curl -X PATCH "$BASE_URL/memories/{memoryId}" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "customHtml": "<resized HTML>",
    "digitalFormat": {
      "category": "phone_wallpaper",
      "widthPx": 1170,
      "heightPx": 2532,
      "outputFormat": "png"
    }
  }'
```

If the target size isn't a standard preset (e.g. a custom resolution), use `"category": "custom"`.

## CRITICAL: Visual Verification — Always Check Your Work

Before telling the user you're done, you MUST take a screenshot to verify your design looks correct. Also take a screenshot BEFORE starting edits to understand what you're working with.

### Take a screenshot of any page:
```bash
curl "$APP_URL/api/screenshot?memoryId={memoryId}&page={pageNum}" \
  -H "Authorization: Bearer $KEY"
```

**App URL:** `https://memoriesweave.com` (or the deployed Next.js URL)

Returns:
```json
{
  "data": {
    "screenshotUrl": "https://...",
    "page": 1,
    "totalPages": 10,
    "width": 1320,
    "height": 2868
  }
}
```

The `screenshotUrl` is a JPEG image you can view to verify the design.

### Required verification workflow:

1. **Before editing:** Take a screenshot of the page(s) you're about to change — understand what you're looking at
2. **After editing:** Take a screenshot of the changed page(s) — verify it looks correct
3. **Fix issues:** If something looks wrong (text cut off, photo not loading, layout broken), fix it and screenshot again
4. **Only then** tell the user you're done

This ensures quality and catches rendering issues before the user sees them.

## Photo Selection Best Practices

### Using the Tag System (AI-generated — use as a guide, not gospel)

All photos have AI-generated tags (e.g., `"couple"`, `"selfie"`, `"traveling"`, `"andrea"`). You can filter by tag via the API:

```bash
curl "$BASE_URL/workspaces/{wsId}/photos?tag=couple&limit=50" \
  -H "Authorization: Bearer $KEY"
```

**IMPORTANT:** Tags are AI-generated and may be inaccurate. Use them as a **first filter** to narrow down candidates, but always:
- **Verify by checking conversation context** around each candidate photo
- **Don't assume a "couple" tag means the right couple** — it could be other people
- **Don't exclude photos missing expected tags** — a great couple photo might only be tagged "selfie" or "joyful" without "couple"
- **Combine tag search with date range** for best results: `?tag=selfie&dateFrom=...&dateTo=...`

Useful tags for finding relationship photos: `couple`, `selfie`, `portrait`, `romantic`, `joyful`, `traveling`, `vacation`, `celebrating`

### Using Specific Photo IDs

If the user provides a specific photo ID (e.g. `jh7dr9t5x4921j296wp1dm92ex829b89`), use it directly — no need to search. The user can copy a photo ID from the website by hovering over a photo and clicking the copy button, or by opening the photo lightbox where the ID is displayed.

```bash
# Get a specific photo by ID
curl "$BASE_URL/photos/{photoId}" -H "Authorization: Bearer $KEY"
```

### General Selection Rules

1. **Always check conversation context** for candidate photos before selecting them
2. **Use tags as a starting point** — filter by `couple`, `selfie`, person names, then verify
3. **Prefer portrait orientation** for phone wallpapers (height > width)
4. **Avoid** photos tagged: food, screenshot, meme, document, sticker (unless specifically requested)
5. **Use medium URLs** for displaying in designs (optimized for web)
6. **Use original URLs** for high-resolution outputs
7. **Check the dateTaken** field to ensure photos are from the requested time period

## Photo URLs

Each photo has three URL variants:
- `urls.thumbnail` — 300px wide WebP (for previews)
- `urls.medium` — 800px wide WebP (for web display)
- `urls.original` — Full resolution original (for print/export)

## Complete Endpoint Reference

### Account
| Method | Path | Description |
|--------|------|-------------|
| GET | `/me` | User info + plan |
| GET | `/me/credits` | Credit balance |
| GET | `/me/usage` | Usage summary |

### Workspaces
| Method | Path | Description |
|--------|------|-------------|
| GET | `/workspaces` | List workspaces |
| GET | `/workspaces/:id` | Workspace details |
| GET | `/workspaces/:id/context` | Workspace AI context (description, relationship, instructions) |
| GET | `/workspaces/:id/persons` | Registered people |

### Digital Formats
| Method | Path | Description |
|--------|------|-------------|
| GET | `/digital-formats` | All format presets with dimensions |

### Photos
| Method | Path | Description |
|--------|------|-------------|
| GET | `/workspaces/:wsId/photos` | List photos (paginated, filterable) |
| GET | `/photos/:id` | Photo details + URLs |
| POST | `/workspaces/:wsId/photos` | Upload a photo |
| PATCH | `/photos/:id` | Update metadata |
| DELETE | `/photos/:id` | Trash a photo |

Query params for listing: `cursor`, `limit` (max 100), `status`, `source`, `tag`, `dateFrom`, `dateTo`

### Memories
| Method | Path | Description |
|--------|------|-------------|
| GET | `/workspaces/:wsId/memories` | List memories |
| GET | `/memories/:id` | Memory details |
| POST | `/workspaces/:wsId/memories` | Create memory (with optional digitalFormat) |
| PATCH | `/memories/:id` | Update title, description, or customHtml |
| DELETE | `/memories/:id` | Delete memory |
| GET | `/memories/:id/html` | Get rendered HTML |
| GET | `/memories/:id/snapshots` | Version history |
| POST | `/memories/:id/snapshots` | Create snapshot (ALWAYS do this after create/before edit) |
| POST | `/memories/:id/snapshots/:snapId/restore` | Restore a previous version |

### Conversations
| Method | Path | Description |
|--------|------|-------------|
| GET | `/workspaces/:wsId/conversations` | List chat chunks |
| GET | `/workspaces/:wsId/conversations/search?start=TS&end=TS` | Search by date range |
| GET | `/workspaces/:wsId/conversations/by-photo/:photoId` | Messages around a photo |

### Sharing
| Method | Path | Description |
|--------|------|-------------|
| POST | `/memories/:id/shares` | Create share link |
| GET | `/memories/:id/shares` | List shares |
| DELETE | `/shares/:id` | Deactivate share |

### Products (Physical/Print)
| Method | Path | Description |
|--------|------|-------------|
| GET | `/products` | Print product catalog |
| GET | `/products/:id` | Product details |

## Pricing

- **Free:** All GET endpoints, creating/updating/deleting resources, pushing HTML
- **Credits:** AI operations only (design chat, content generation, photo captioning)
- 1 credit = $1 USD, same pool as website

## Rate Limits

| Plan | General RPM | AI Ops/min |
|------|------------|-----------|
| Free | 10 | 2 |
| Starter | 30 | 5 |
| Plus | 60 | 10 |
| Pro | 120 | 20 |

## Error Codes

| Code | Status | Description |
|------|--------|-------------|
| `bad_request` | 400 | Invalid parameters |
| `unauthorized` | 401 | Missing/invalid API key |
| `not_found` | 404 | Resource not found |
| `rate_limited` | 429 | Too many requests |
| `internal_error` | 500 | Server error |

## Full API Reference

- OpenAPI spec: https://memoriesweave.com/api/openapi.json
- Interactive docs: https://memoriesweave.com/docs/api
