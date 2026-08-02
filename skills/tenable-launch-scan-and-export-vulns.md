---
name: Launch a scan and export its vulnerabilities
description: Create and launch a Tenable Vulnerability Management scan, wait for it to finish, then export the resulting vulnerabilities via the async export job pattern.
api: openapi/tenable-vulnerability-management-openapi.json
operations: [scans-create, scans-launch, scans-get-latest-status, exports-vulns-request-export, exports-vulns-export-status, exports-vulns-download-chunk]
---

# Launch a scan and export its vulnerabilities

Base URL: `https://cloud.tenable.com`. Auth: send header
`X-ApiKeys: accessKey=<ACCESS_KEY>;secretKey=<SECRET_KEY>` on every request.

## Steps

1. **Create a scan** — `POST /scans` (`scans-create`) with a template/policy
   `uuid`, `settings.name`, and `settings.text_targets`. Capture the returned
   `scan.id`.
2. **Launch it** — `POST /scans/{scan_id}/launch` (`scans-launch`). Returns a
   `scan_uuid` for the running instance.
3. **Poll until complete** — `GET /scans/{scan_id}/latest-status`
   (`scans-get-latest-status`) with backoff; wait for status `completed`.
   Honor `429 Too Many Requests` / `Retry-After` — do not tight-loop.
4. **Request a vulnerabilities export** — `POST /vulns/export`
   (`exports-vulns-request-export`) with filters (e.g. `since`, `severity`,
   `num_assets`). Capture `export_uuid`.
5. **Poll export status** — `GET /vulns/export/{export_uuid}/status`
   (`exports-vulns-export-status`) until `status: FINISHED`; read the
   `chunks_available` array.
6. **Download each chunk** — `GET /vulns/export/{export_uuid}/chunks/{chunk_id}`
   (`exports-vulns-download-chunk`) for every id in `chunks_available`.

## Rules

- Prefer the export job pattern over `/workbenches/*` for bulk data.
- No idempotency keys — re-POSTing `/scans` creates a new scan; store `scan.id`.
- Errors are plain JSON (not RFC 9457): 401 = bad keys, 403 = insufficient role,
  429 = rate limited (back off with `Retry-After`).
