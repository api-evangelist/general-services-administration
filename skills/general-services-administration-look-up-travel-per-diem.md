---
name: look-up-travel-per-diem
description: Get the correct federal lodging and M&IE reimbursement rate for a destination and fiscal year, without averaging seasonal rates or forgetting the first/last-day rule.
api: general-services-administration:per-diem-api
operations:
  - 'GET /v2/rates/city/{city}/state/{state}/year/{year}'
  - 'GET /v2/rates/zip/{zip}/year/{year}'
  - 'GET /v2/rates/state/{state}/year/{year}'
  - 'GET /v2/rates/conus/lodging/{year}'
  - 'GET /v2/rates/conus/zipcodes/{year}'
---

# Look up federal travel per diem

Base: `https://api.gsa.gov/travel/perdiem`. Key: any api.data.gov key, passed as
`X-Api-Key` or `?api_key=`. `DEMO_KEY` works for exploration at 30 requests/hour.

> The Per Diem OpenAPI declares **no operationId** on any of its five operations,
> so everything below is addressed by method and path. That is not an omission in
> this skill; it is what GSA published.

## Steps

1. **Establish the fiscal year first.** GSA fiscal years run **1 October – 30 September**.
   FY2024 means October 2023 through September 2024. A calendar year is the wrong
   input and will silently return the wrong rate.

2. **Pick the lookup by what you actually know.**
   - City + state → `GET /v2/rates/city/{city}/state/{state}/year/{year}`
   - Only a ZIP → `GET /v2/rates/zip/{zip}/year/{year}`
   - Comparing destinations in a state → `GET /v2/rates/state/{state}/year/{year}`
   - Which destination a ZIP belongs to → `GET /v2/rates/conus/zipcodes/{year}`
   - The whole CONUS table → `GET /v2/rates/conus/lodging/{year}` (large; page it)

3. **Report the month, not the year.** Non-Standard Areas have **seasonal** lodging
   rates that change month to month. Returning an average is a wrong answer that
   looks like a right one — quote the rate for the month of travel.

4. **Apply the 75% rule.** Only 75% of the M&IE rate may be claimed on the first and
   last day of travel.

5. **Empty result means the standard rate.** If a city lookup returns nothing, the
   destination is not individually listed and the CONUS standard rate applies.

## Gap to know about

The M&IE breakdown endpoint, `GET /v2/rates/conus/mie/{year}`, is documented on
<https://open.gsa.gov/api/perdiem/> and used in GSA's own curl examples but is
**absent from the published OpenAPI**. A client generated from the spec will not
have it. GSA-TTS's own MCP server exposes it as `perdiem_get_mie_rates`; see
`mcp/general-services-administration-tool-crosswalk.yml`.
