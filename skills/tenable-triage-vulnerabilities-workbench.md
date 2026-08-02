---
name: Triage vulnerabilities from the workbench
description: Query the Tenable Vulnerability Management workbench to list current vulnerabilities, drill into a specific plugin, and pull the affected-asset detail for triage.
api: openapi/tenable-vulnerability-management-openapi.json
operations: [workbenches-vulnerabilities, workbenches-vulnerability-info, workbenches-vulnerability-output, workbenches-assets-vulnerabilities]
---

# Triage vulnerabilities from the workbench

Base URL: `https://cloud.tenable.com`. Auth header on every request:
`X-ApiKeys: accessKey=<ACCESS_KEY>;secretKey=<SECRET_KEY>`.

Use the workbench for interactive triage of recent findings (dashboard-style
queries). For bulk extraction use the export skill instead.

## Steps

1. **List vulnerabilities** — `GET /workbenches/vulnerabilities`
   (`workbenches-vulnerabilities`) with `date_range` and `filter.*` params
   (e.g. `filter.0.filter=severity`, `filter.0.quality=eq`,
   `filter.0.value=critical`). Note each `plugin_id`.
2. **Get plugin detail** — `GET /workbenches/vulnerabilities/{plugin_id}/info`
   (`workbenches-vulnerability-info`) for description, CVSS/VPR, solution, and
   affected counts.
3. **Get plugin output** — `GET /workbenches/vulnerabilities/{plugin_id}/outputs`
   (`workbenches-vulnerability-output`) for the raw evidence per host.
4. **Scope by asset** — `GET /workbenches/assets/vulnerabilities`
   (`workbenches-assets-vulnerabilities`) to rank the assets carrying the most
   (or most severe) findings.

## Rules

- Workbench queries cover a rolling window (`date_range`, default ~30 days);
  widen `date_range` for older data.
- Filters use the indexed `filter.N.filter/quality/value` triplet form.
- Honor `429` / `Retry-After`; errors are plain JSON (not RFC 9457).
