---
name: Request and retrieve captions for a video
description: Request a machine or professional caption for a VidGrid video, wait on webhook events, and fetch the finished transcript.
api: openapi/vidgrid-openapi-original.json
operations:
  - POST /vidgrid/v2/captions
  - GET /vidgrid/v2/captions/{identifier}
  - DELETE /vidgrid/v2/captions/{identifier}
---

# Request and retrieve VidGrid captions

Base URL: `https://api.vidgrid.com/ContentManagement`

## Auth
HTTP Basic: `Authorization: Basic {base64(key:secret)}` — see
`../authentication/vidgrid-authentication.yml`.

## Steps
1. **Request a caption.** `POST /vidgrid/v2/captions` with
   `{"video_identifier": "...", "type": "machine"}` — `type` must be `machine`
   or `professional`.
2. **Wait for readiness.** Prefer webhooks over polling: `CAPTION_REQUESTED`,
   `CAPTION_READY`, and `CAPTION_FAILED` events fire to endpoints configured at
   https://app.vidgrid.com/webhooks (`../asyncapi/vidgrid-webhooks.yml`).
3. **Fetch the caption.** `GET /vidgrid/v2/captions/{identifier}` returns a
   Caption Resource with `language`, `status`, `type`, `video_identifier`, and
   the `transcript`.
4. **Remove if needed.** `DELETE /vidgrid/v2/captions/{identifier}` returns
   `204 No Content`.

## Rules
- Errors arrive as `{"message": "..."}`; `422` means the video identifier or
  caption type is invalid (`../errors/vidgrid-problem-types.yml`).
- Respect the 60 requests/minute account limit — poll sparingly if webhooks are
  unavailable (`../rate-limits/vidgrid-rate-limits.yml`).
