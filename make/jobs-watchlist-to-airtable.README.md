# Jobs Watchlist → Airtable (Make blueprint)

**File:** `jobs-watchlist-to-airtable.blueprint.json`
**Public Make template:** ID 19725 — **live, approved Sep 16 2026** (ticket #2307299)
**Public URL:** https://www.make.com/en/integration/19725-jobs-watchlist-to-airtable
(logged-in users are region-redirected to `https://us2.make.com/templates/19725`)
**Editor:** `us2.make.com/2954832/templates/16864/edit`

## What it does
Three-module scenario: an HTTP call that runs `make_no_mistakes/multi-ats-job-board-api`
synchronously against a company watchlist with `postedWithinDays: 1`, a built-in Iterator that
fans the returned array out one job at a time, and an HTTP `PATCH` to Airtable's REST API using
its `performUpsert` option (merge key: `ats` + `job_id`) so re-runs update existing rows instead
of duplicating them.

## Module versions (updated Sep 2026)
Both HTTP modules are the **current HTTP app, version 4 — "Make a request"**. In blueprint JSON
that is:

```json
{ "module": "http:MakeRequest", "version": 4 }
```

The old v3 module (`http:ActionSendData`, version 3) is now shown in Make as **"HTTP (legacy)"**
and is no longer used here. v4 keeps `url`, `method` and `headers` in `mapper`, but the rest of
the shape changed:

| v3 (`http:ActionSendData`) | v4 (`http:MakeRequest`) |
|---|---|
| no auth concept (raw headers only) | `parameters.authenticationType`: `noAuth` / `apiKey` / `basicAuth` / `oAuth` |
| `bodyType: "raw"` + `contentType` + `data` | `mapper.inputMethod: "jsonString"` + `mapper.jsonStringBodyContent` (or a data structure) |
| `followRedirect`, `followAllRedirects` | `mapper.allowRedirects` |
| `gzip` | `mapper.requestCompressedContent` |
| `rejectUnauthorized`, `ca`, `useMtls` | `parameters.tlsType` (`""` / `tls` / `mTls`) |
| `method: "PATCH"` (upper case) | `method: "patch"` (lower case) |
| — | `mapper.stopOnHttpError`, built-in pagination, `parameters.proxyKeychain` |

`parseResponse` and `shareCookies` carry over unchanged.

**Note:** the v4 "Body content type" select does not reliably round-trip through a template save,
so both modules also send an explicit `Content-Type: application/json` header. Airtable rejects
the upsert body without it — keep it.

## Wizard fields (what the template asks the user for)
Marked in the template editor with **Use in Wizard** + **Use as default value**, each with help
text:

| Module | Field | Help text shown to the user |
|---|---|---|
| 1 – Run ATS Watchlist Actor | `Headers → Authorization → Value` | Your Apify API token. Get it at https://console.apify.com/settings/integrations (Personal API tokens). Replace `YOUR-APIFY-API-TOKEN`, keeping the word `Bearer` and the space in front of it. |
| 3 – Upsert Job to Airtable | `Headers → Authorization → Value` | Your Airtable personal access token. Create one at https://airtable.com/create/tokens with the `data.records:write` scope on your base. Replace `YOUR-AIRTABLE-TOKEN`, keeping the `Bearer ` prefix. |

The template description (Set up a template → Template description) repeats both token links and
adds the base ID / table name step, the required column list, and the "edit `companies` /
`postedWithinDays`" note.

## Required credentials
- **Apify API token** — goes in module 1's `Authorization` header (`Bearer <token>`).
- **Airtable Personal Access Token** with `data.records:write` on the target base — goes in
  module 3's `Authorization` header. Set the real base ID and table name in module 3's URL
  (`REPLACE_WITH_BASE_ID`, `REPLACE_WITH_TABLE_NAME`), and make sure that table has columns
  matching the mapped fields (company, ats, job_id, title, department, location_raw, is_remote,
  employment_type, salary_min, salary_max, salary_currency, posted_at, apply_url).

No Make connection or keychain is required: authentication type is **No authentication** and both
tokens are plain header values, so nothing is stored in the Make account and nothing is stripped
when the template is published.

## Setup after import
1. Import the blueprint into a new scenario (or start from the public template).
2. Module 1: paste your Apify token, edit the `companies` array to your real watchlist.
3. Module 3: paste your Airtable token, fill in the base ID and table name in the URL. The method
   is already `PATCH` and the body already carries `performUpsert.fieldsToMergeOn: ["ats",
   "job_id"]` — Airtable's API requires both for upsert-by-key; don't drop either.
4. Click the scenario's clock icon and set the schedule to **daily**.

## Cost per run
$0.005/run start + $0.004/job returned. A `postedWithinDays: 1` sweep of 5 companies typically
returns a handful to a few dozen new jobs:
- 20 new jobs/day ≈ **$0.005 + $0.08 = $0.085/run** → ≈$2.55/month
- 100 new jobs/day ≈ **$0.405/run** → ≈$12/month

Make's own operation cost scales with Iterator cycles — check your plan's operations quota for a
large watchlist.
