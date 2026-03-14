---
name: memoriesweave
description: Creates and manages photo memory collections on MemoriesWeave via its REST API. Use when the user mentions MemoriesWeave, memory collections, photo wallpapers, photo books, or wants to create/edit digital memories or print products using the MemoriesWeave API. Do not use for general photo editing, image generation, or non-MemoriesWeave tasks.
---

# MemoriesWeave API

Provides programmatic access to MemoriesWeave — a platform for creating photo memory collections with custom HTML layouts. The agent designs HTML layouts and pushes them to memories. The API provides photos, conversations, workspace context, and people data.

## How to call the API

**ALWAYS use `curl` via the Bash tool.** Do not use WebFetch, fetch(), or any other HTTP client — they fail due to redirects and auth header stripping.

```bash
API="https://grandiose-loris-729.eu-west-1.convex.site/api/v1"
KEY="<user-provided-api-key>"

curl -s "$API/ENDPOINT" -H "Authorization: Bearer $KEY"
```

The user provides their API key (format: `mw_sk_<32hex>`) when requesting this skill.

## Workflow: Creating or editing a memory

Follow these steps in order. Do not skip steps.

### Step 1: Lock the memory FIRST

The VERY FIRST API call must be locking the memory. Do this before gathering context, before reading HTML, before anything else. This shows a loading animation on the user's screen immediately.

```bash
curl -s -X POST "$API/memories/{memoryId}/lock" -H "Authorization: Bearer $KEY"
```

If creating a new memory (no memory ID yet), lock it immediately after creation.

### Step 2: Gather context

```bash
# Get workspace list
curl -s "$API/workspaces" -H "Authorization: Bearer $KEY"

# Get workspace context (who the people are, relationship info, instructions)
curl -s "$API/workspaces/{wsId}/context" -H "Authorization: Bearer $KEY"

# Get registered people (names, roles, descriptions)
curl -s "$API/workspaces/{wsId}/persons" -H "Authorization: Bearer $KEY"
```

### Step 3: Save a "before" snapshot

```bash
curl -s -X POST "$API/memories/{memoryId}/snapshots" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"label": "Before edit — description of planned change"}'
```

### Step 4: Select photos

Use tag filtering as a starting point, then verify with conversation context.

```bash
# Filter by tag (e.g. couple, selfie, portrait, traveling)
curl -s "$API/workspaces/{wsId}/photos?tag=couple&limit=50" -H "Authorization: Bearer $KEY"

# If user provides a photo ID, fetch it directly
curl -s "$API/photos/{photoId}" -H "Authorization: Bearer $KEY"

# Get conversation context around a photo (what was happening when it was shared)
curl -s "$API/workspaces/{wsId}/conversations/by-photo/{photoId}" -H "Authorization: Bearer $KEY"
```

**Tags are AI-generated and may be inaccurate.** Use them as a first filter, then verify with conversation context. Do not rely on tags alone.

**Pagination:** If a search doesn't return enough results, check `meta.hasMore` in the response. If `true`, use the `cursor` value to fetch the next batch:

```bash
# First request
curl -s "$API/workspaces/{wsId}/photos?tag=couple&limit=50" -H "Authorization: Bearer $KEY"
# Response includes: "meta": { "cursor": "abc123", "hasMore": true }

# Get next batch using cursor
curl -s "$API/workspaces/{wsId}/photos?tag=couple&limit=50&cursor=abc123" -H "Authorization: Bearer $KEY"
```

Keep paginating until you find what you need or `hasMore` is `false`.

### Step 5: Choose the correct format

```bash
curl -s "$API/digital-formats" -H "Authorization: Bearer $KEY"
```

Common formats: Phone Wallpaper (1170x2532 iPhone 13, 1320x2868 iPhone 16 Pro Max), Desktop (3840x2160), Social Post (1080x1080).

### Step 5b: VERIFY every photo before using it

Before including ANY photo in your design, you MUST verify it exists and get its actual data by calling:

```bash
curl -s "$API/photos/{photoId}" -H "Authorization: Bearer $KEY"
```

**NEVER fabricate, guess, or construct photo URLs.** Every photo URL must come from an actual API response (`urls.original`, `urls.medium`, or `urls.thumbnail`). If a photo search returns no results for a month, report that to the user — do not invent URLs.

**Verification checklist for every photo:**
1. You received the photo ID from the API (not made up)
2. You called `GET /photos/{photoId}` and got a valid response
3. The `dateTaken` field matches the month you're placing it on
4. The `tags` or photo content matches what you expect (couple/selfie/andrea etc.)
5. The `urls.original` URL is what you're putting in the HTML

If the API returns empty results for a time period, try different search strategies:
- Search by `tag=couple`, `tag=selfie`, `tag=andrea` without date range
- Then filter results by `dateTaken` in your code
- Try broader date ranges
- If truly no photos exist for a month, tell the user

### Step 6: Design HTML and push

Design self-contained HTML with inline styles. For multi-page memories, wrap each page in `<div data-mw-page="N">`.

```bash
curl -s -X PATCH "$API/memories/{memoryId}" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"customHtml": "<div data-mw-page=\"1\" style=\"width:1170px;height:2532px;...\">...</div>"}'
```

When resizing, include `digitalFormat` in the same PATCH:

```bash
curl -s -X PATCH "$API/memories/{memoryId}" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"customHtml": "...", "digitalFormat": {"category": "phone_wallpaper", "widthPx": 1170, "heightPx": 2532, "outputFormat": "png"}}'
```

### Step 7: Verify ALL images load, then screenshot

Before screenshotting, verify that every image URL in your HTML actually loads. Extract all image URLs from your HTML and spot-check at least 3 by fetching them:

```bash
curl -sI "https://pub-....r2.dev/photos/.../photo.jpg" | head -1
# Should return: HTTP/2 200 (not 404 or 403)
```

If any return 404, you used a fabricated or wrong URL — go back to Step 5b and fix it.

Then take screenshots to verify the design:

```bash
curl -s "https://www.memoriesweave.com/api/screenshot?memoryId={memoryId}&page=1" \
  -H "Authorization: Bearer $KEY"
```

Returns `{ "data": { "screenshotUrl": "https://..." } }`. Download and view the screenshot. Check that:
- All photos are visible (no broken image icons)
- Photos show the correct people/content
- No duplicate-looking photos on the same page
- Text is readable and correctly positioned

### Step 8: Save an "after" snapshot

```bash
curl -s -X POST "$API/memories/{memoryId}/snapshots" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"label": "After edit — description of what changed"}'
```

### Step 9: Unlock the memory

This removes the loading animation from the user's screen.

```bash
curl -s -X POST "$API/memories/{memoryId}/unlock" -H "Authorization: Bearer $KEY"
```

**Always unlock when done**, even if an error occurred.

## Product-specific rules

**Wall calendars** MUST always have exactly **14 pages**: cover + 12 months (Jan-Dec) + back page. Each month page needs a correct calendar grid with proper day-of-week starts. Dimensions: `3772x5250px`. Never use duplicate photos across pages. Every photo on relationship months must show the people (not food, scenery, or screenshots).

**Phone wallpapers** support up to 10 pages. Each page is a standalone wallpaper at the device's resolution.

## Photo selection rules

1. **Always check conversation context** around candidate photos before selecting them.
2. **If user provides a photo ID**, use it directly via `GET /photos/{photoId}`.
3. **Prefer** tags: `couple`, `selfie`, `portrait`, `andrea`, `suliman`, `romantic`, `joyful`, `traveling`.
4. **Avoid** tags: `food`, `screenshot`, `meme`, `document`, `sticker`, `illustration` (unless requested).
5. **Prefer portrait orientation** (height > width) for phone wallpapers.
6. Use `urls.original` for high-resolution output, `urls.medium` for web display.
7. **No duplicate photos across ANY pages** — not just by URL, but by content. Photos taken within seconds of each other (burst shots, slightly different angles of the same scene) count as duplicates. Check the `fileName` timestamps — if two photos have timestamps within 10 seconds, they are from the same moment.
8. **Cover and back pages must use unique photos** not used on any month page.
9. **Each month's photos must be from that actual month.** Check the `dateTaken` timestamp. Do not use April photos on the August page.
10. **Use the earliest occurrence of a photo.** Photos in WhatsApp are sometimes resent months later. The `dateTaken` field reflects when the photo was originally taken, NOT when it was shared in the chat. When filtering by date range, a photo taken in July may appear in a December conversation chunk because it was resent. Always use `dateTaken` to determine which month a photo belongs to — if `dateTaken` says July, it goes on the July page, never December, even if it was found in a December conversation search.

## Photo URLs

Each photo has three variants:
- `urls.thumbnail` — 300px wide WebP
- `urls.medium` — 800px wide WebP
- `urls.original` — Full resolution

## Endpoint reference

### Account
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/me` | User info and plan |
| GET | `/me/credits` | Credit balance |
| GET | `/me/usage` | Usage summary |

### Workspaces
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/workspaces` | List workspaces |
| GET | `/workspaces/:id` | Workspace details |
| GET | `/workspaces/:id/context` | AI context (description, relationship, instructions) |
| GET | `/workspaces/:id/persons` | Registered people |

### Photos
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/workspaces/:wsId/photos` | List photos. Params: `cursor`, `limit`, `tag`, `dateFrom`, `dateTo` |
| GET | `/photos/:id` | Single photo details and URLs |
| PATCH | `/photos/:id` | Update caption or tags |

### Memories
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/workspaces/:wsId/memories` | List memories |
| GET | `/memories/:id` | Memory details |
| POST | `/workspaces/:wsId/memories` | Create memory. Body: `title`, `description`, optional `digitalFormat` |
| PATCH | `/memories/:id` | Update `title`, `description`, `customHtml`, or `digitalFormat` |
| DELETE | `/memories/:id` | Delete memory |
| GET | `/memories/:id/html` | Get rendered HTML |
| POST | `/memories/:id/lock` | Show AI working overlay on frontend |
| POST | `/memories/:id/unlock` | Remove AI working overlay |
| GET | `/memories/:id/snapshots` | List version history |
| POST | `/memories/:id/snapshots` | Create snapshot. Body: `{"label": "..."}` |
| POST | `/memories/:id/snapshots/:snapId/restore` | Restore a previous version |

### Conversations
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/workspaces/:wsId/conversations` | List chat chunks |
| GET | `/workspaces/:wsId/conversations/search?start=TS&end=TS` | Search by date range |
| GET | `/workspaces/:wsId/conversations/by-photo/:photoId` | Messages around a photo |

### Formats and products
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/digital-formats` | All format presets with dimensions |
| GET | `/products` | Print product catalog |

### Screenshot (uses website domain, not API domain)
| Method | URL | Description |
|--------|-----|-------------|
| GET | `https://www.memoriesweave.com/api/screenshot?memoryId=X&page=N` | Render page to JPEG |

## Pricing

- **Free:** All GET endpoints, CRUD operations, pushing HTML, snapshots, lock/unlock.
- **Credits:** AI operations only (design chat, content generation, photo captioning via Trigger.dev pipeline). The agent designing HTML does not consume credits.

## Rate limits

| Plan | Requests/min | AI ops/min |
|------|-------------|-----------|
| Free | 10 | 2 |
| Starter | 30 | 5 |
| Plus | 60 | 10 |
| Pro | 120 | 20 |

## Error codes

| Code | Status | Meaning |
|------|--------|---------|
| `bad_request` | 400 | Invalid parameters |
| `unauthorized` | 401 | Missing or invalid API key |
| `not_found` | 404 | Resource not found |
| `rate_limited` | 429 | Too many requests |
| `internal_error` | 500 | Server error |
