---
name: audit-federal-websites
description: Pull the daily Site Scanning results for a federal domain and read them alongside the DAP analytics for the same site.
api: general-services-administration:site-scanning-api
operations:
  - WebsiteController_getResults
  - WebsiteController_getResultByUrl
  - AnalysisController_getResults
---

# Audit federal websites (Site Scanning + DAP)

Base: `https://api.gsa.gov/technology/site-scanning/v1`. Key: any api.data.gov key.
This API is fully public and works with `DEMO_KEY` for exploration.

## Steps

1. **Scan results for a domain.** `WebsiteController_getResults` —
   `GET /websites?target_url_domain=gsa.gov`. Every scan column is also a filter:
   `final_url_domain`, `final_url_live`, `target_url_redirects`,
   `target_url_agency_owner`, `target_url_bureau_owner`, `scan_status`,
   `dap_detected_final_url`.

2. **One site in detail.** `WebsiteController_getResultByUrl` —
   `GET /websites/{url}`.

3. **The analysis view.** `AnalysisController_getResults` — `GET /analysis`, same
   filter set, the derived rather than the raw form.

4. **Pagination here is `page` + `limit`.** (Entity Management uses `page`+`size`;
   Regulations.gov uses `page[number]`+`page[size]`. Do not carry one convention
   across GSA APIs.)

5. **Bulk instead of paging.** The whole corpus is published as a flat file:
   `https://api.gsa.gov/technology/site-scanning/data/site-scanning-latest.json`
   (and `.csv`, and a `site-scanning-live-filtered-latest` variant). For anything
   analytical, take the file.

6. **Cross-read traffic.** `GET /analytics/dap/v2/domain/{domain}/reports/site/data`
   on `https://api.gsa.gov` returns the Digital Analytics Program numbers for the
   same domain, so "is this site live and compliant" and "does anyone use it" can
   be answered together. `dap_detected_final_url` in the scan result tells you
   whether the DAP tag is even installed.

## Notes

- This API's own OpenAPI is served live and keyed:
  `https://api.gsa.gov/technology/site-scanning/v1/api-json?api_key=DEMO_KEY`.
  It is the only GSA API that serves its contract from the API host itself.
- The spec declares **no `servers` block**; the base above is from the documentation.
