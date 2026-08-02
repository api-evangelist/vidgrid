---
name: Record or upload a video via an API token
description: Mint a record, upload, or direct-upload token, launch the VidGrid recorder/uploader, and pick up the resulting video when webhook events fire.
api: openapi/vidgrid-openapi-original.json
operations:
  - POST /vidgrid/v2/tokens
  - GET /vidgrid/v2/videos/{identifier}
  - POST /vidgrid/v2/webhooks
---

# Record or upload a video via a VidGrid token

Base URL: `https://api.vidgrid.com/ContentManagement`

## Auth
HTTP Basic: `Authorization: Basic {base64(key:secret)}` — server-side only; the
token you mint is what reaches the client.

## Steps
1. **Mint a token.** `POST /vidgrid/v2/tokens` with
   `{"type": "record" | "upload" | "direct_upload", "video": {...}, "recorder": {...}, "webhook_extras": {...}}`.
   - `video` presets properties on videos created with the token.
   - `recorder` configures recorder behavior when launched with the token.
   - `webhook_extras` attaches custom data that comes back in the `extras`
     field of webhook events.
2. **Hand the token to the client.** Record tokens launch the VidGrid Screen
   Recorder; upload tokens open the web uploader; direct-upload tokens include
   `formAttributes`/`formInputs` for a direct file POST
   (`../components/vidgrid-components.yml`).
3. **Wait for `VIDEO_UPLOADED` → `VIDEO_READY`** (or `VIDEO_FAILED`) webhook
   events, configured at https://app.vidgrid.com/webhooks. To re-fire an event
   for a video, `POST /vidgrid/v2/webhooks` with
   `{"video_identifier": "...", "type": "...", "payload": {...}}`.
4. **Fetch the video.** `GET /vidgrid/v2/videos/{identifier}` with `include[]`
   for metadata, thumbnails, or captions.

## Rules
- Tokens expire (`expires` on the token resource) — mint fresh tokens per
  session rather than caching long-lived ones.
- The webhook payload `token` field is deprecated — key off `event_type` +
  `data` (`../asyncapi/vidgrid-webhooks.yml`).
- 60 requests/minute account limit; no idempotency keys — avoid blind retries
  of `POST /vidgrid/v2/tokens`.
