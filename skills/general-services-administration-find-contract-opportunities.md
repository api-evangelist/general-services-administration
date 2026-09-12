---
name: find-contract-opportunities
description: Search published federal contract opportunities on SAM.gov and follow one through to its award record, respecting the mandatory date window and the retired notice types.
api: general-services-administration:samgov-get-opportunities-api
operations:
  - 'GET /opportunities/v2/search'
  - getAwardDetails
  - searchAssistanceListings
---

# Find federal contract opportunities

Base: `https://api.sam.gov/opportunities/v2/search`. Key: a SAM.gov public API key
from the Account Details page.

## Steps

1. **Both date bounds are mandatory and the window is capped at one year.**
   `postedFrom` and `postedTo` are required, formatted `MM/dd/yyyy`, and the range
   between them may not exceed 12 months. To sweep several years, walk it a year
   at a time — there is no cursor.

2. **Filter by notice type with `ptype`.** Valid codes: `u` Justification (J&A),
   `p` Pre-solicitation, `a` Award Notice, `r` Sources Sought, `s` Special Notice,
   `o` Solicitation, `g` Sale of Surplus Property, `k` Combined Synopsis/Solicitation,
   `i` Intent to Bundle Requirements. **`f` and `l` are retired** — use `u` instead
   of the old fair-opportunity code.

3. **Do not build on `deptname` or `subtier`.** Both are marked deprecated in the
   request parameter table. Filter on the Federal Hierarchy organization fields the
   response carries instead.

4. **Pagination is `limit` + `offset`** on this API — not `page`/`size`, which is
   what the Entity Management API uses on the same host.

5. **Follow an award through.** `getAwardDetails` — `GET /contract-awards/v1/search`
   on `https://api.sam.gov` returns the award record; `downloadAwardData`
   (`GET /contract-awards/v1/download`) gives you the bulk form. For grants rather
   than contracts, `searchAssistanceListings` —
   `GET /assistance-listings/v1/search`.

6. **Join to the vendor.** Award records carry the awardee `ueiSAM`; hand that to
   the `check-vendor-eligibility` skill rather than matching on company name.

## Rules that will bite you

- This API only ever returns the **latest active version** of an opportunity. Version
  history lives in the SAM.gov Data Services section, not here.
- Active notices refresh daily; archived notices refresh weekly.
- The published change log for this API has had **no entry since 2021-06-11** while
  the API moved to v2 and retired two notice types. Treat it as unmaintained and
  verify behaviour against the live response.
