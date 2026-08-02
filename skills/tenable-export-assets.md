---
name: Export the asset inventory
description: Export the full Tenable Vulnerability Management asset inventory using the async export job pattern (request, poll status, download chunks).
api: openapi/tenable-vulnerability-management-openapi.json
operations: [export-assets-v2, exports-assets-export-status, exports-assets-download-chunk, exports-assets-export-cancel]
---

# Export the asset inventory

Base URL: `https://cloud.tenable.com`. Auth header on every request:
`X-ApiKeys: accessKey=<ACCESS_KEY>;secretKey=<SECRET_KEY>`.

## Steps

1. **Request the export** — `POST /assets/v2/export` (`export-assets-v2`) with a
   `chunk_size` and optional `filters` (e.g. `last_assessed`, `sources`, tag
   filters). Capture the `export_uuid`.
2. **Poll status** — `GET /assets/export/{export_uuid}/status`
   (`exports-assets-export-status`) until `status: FINISHED`; read
   `chunks_available`.
3. **Download each chunk** — `GET /assets/export/{export_uuid}/chunks/{chunk_id}`
   (`exports-assets-download-chunk`) for each id.
4. **Cancel if needed** — `POST /assets/export/{export_uuid}/cancel`
   (`exports-assets-export-cancel`) to abort an in-progress export.

## Rules

- `chunk_size` controls assets-per-chunk; smaller chunks download faster but
  create more requests (watch the rate limit).
- Poll with backoff and honor `429` / `Retry-After`.
- The export is a point-in-time snapshot; re-request for fresh data.
