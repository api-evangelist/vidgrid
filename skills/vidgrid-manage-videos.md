---
name: Retrieve, update, and delete videos
description: Manage Video Resources on a VidGrid account — look them up, batch-retrieve, expand nested properties, update, and delete (trash or permanent).
api: openapi/vidgrid-openapi-original.json
operations:
  - GET /vidgrid/v2/videos/{identifier}
  - PATCH /vidgrid/v2/videos/{identifier}
  - DELETE /vidgrid/v2/videos/{identifier}
  - GET /vidgrid/v2/search/{resource}
---

# Retrieve, update, and delete VidGrid videos

Base URL: `https://api.vidgrid.com/ContentManagement`

## Auth
HTTP Basic: `Authorization: Basic {base64(key:secret)}`. Keys come from
https://app.vidgrid.com/integrations and are server-side only — never expose them
client side. See `../authentication/vidgrid-authentication.yml`.

## Steps
1. **Find videos.** `GET /vidgrid/v2/search/{resource}` with `page` and `filter[]`
   query params, or `GET /vidgrid/v2/videos/{identifier}`. For a batch, pass
   `identifiers[]` (it takes priority over the path identifier). Add `include[]`
   to expand nested properties (captions, events, metadata, thumbnails, jwts).
2. **Read the envelope.** Success is `{"data": [...]}` on GETs; errors are
   `{"message": "..."}` (see `../errors/vidgrid-problem-types.yml`).
3. **Update.** `PATCH /vidgrid/v2/videos/{identifier}` with a JSON body
   `{"properties": {...}}`.
4. **Delete.** `DELETE /vidgrid/v2/videos/{identifier}`; body `{"trash": true}`
   moves the video to trash for 30 days, `{"trash": false}` deletes permanently.
   Expect `204 No Content`.

## Rules
- Rate limit is 60 requests/minute; watch `X-RateLimit-Limit` /
  `X-RateLimit-Remaining` and back off before exhausting the window
  (`../rate-limits/vidgrid-rate-limits.yml`).
- No idempotency-key mechanism exists — do not blindly retry non-GET requests
  (`../conventions/vidgrid-conventions.yml`).
- `422` means validation failure; fix the body rather than retrying.
