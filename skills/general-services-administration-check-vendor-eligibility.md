---
name: check-vendor-eligibility
description: Confirm whether a vendor is registered and not excluded before you award, pay or contract with them, using SAM.gov Entity Management and Exclusions.
api: general-services-administration:samgov-entity-management-api
operations:
  - getEntityManagementDataUsingGET_4
  - getExclusionsDataUsingGET_2
  - getFileFromS3UsingGET
---

# Check vendor eligibility (SAM.gov)

Two questions, two APIs, one identifier. The identifier is the **UEI** (12-character
Unique Entity Identifier). DUNS numbers were removed from the SAM.gov data
dictionary on 2022-11-25 and will not work.

## Before you start

- Get an API key from the **Account Details** page on `sam.gov` (or `alpha.sam.gov`
  for the prodlike environment). This is *not* an api.data.gov key.
- Send it as the `X-Api-Key` **header**, not in the URL, for anything non-public.
- Know your daily ceiling before you loop: a non-federal user with no SAM.gov role
  gets **10 requests per day**. See `rate-limits/`.

## Steps

1. **Is the entity registered?**
   `getEntityManagementDataUsingGET_4` — `GET /entity-information/v4/entities`
   on `https://api.sam.gov`. Filter with `ueiSAM`. Use `includeSections` to ask
   only for what you need (`entityRegistration,coreData`) — the full record is
   large and there is no field-expansion mechanism to trim it afterwards.

2. **Is the entity excluded (debarred/suspended)?**
   `getExclusionsDataUsingGET_2` — `GET /entity-information/v4/exclusions`.
   Query by the same `ueiSAM`. An empty result set is the answer you want;
   treat it as a negative finding, not as an error.

3. **Bulk instead of per-vendor.** If you are screening more than a handful,
   stop looping. `getFileFromS3UsingGET` — `GET /entity-information/v4/download-entities`
   returns an extract rather than a page, and the Extracts Download API
   (`downloadUsingGET`, `GET /extracts/v1/extracts`) exists precisely so that a
   daily 1,000-request ceiling is not the thing that limits you.

## Rules that will bite you

- **Pagination is `page` + `size`, zero-based.** Not `limit`/`offset`, which is what
  the Get Opportunities API uses. GSA is not consistent across its own estate.
- **Sensitive and FOUO data are a different authorization, not a different endpoint
  shape.** They need a SAM.gov **System Account**: HTTP Basic
  `base64(username:password)` in `Authorization`, the API key in `x-api-key`, and
  both `Accept` and `Content-Type` set to `application/json`.
- **The contract will not help you.** This spec declares 14 operations and **zero**
  component schemas, so generated clients give you untyped blobs. Read the
  response shape from <https://open.gsa.gov/api/entity-api/> and its Data Dictionary.
- **Errors**: `{"error":{"code":"API_KEY_MISSING"|"OVER_RATE_LIMIT"|...}}`. Branch on
  `error.code`, never on `error.message` — api.data.gov says the message text changes.
