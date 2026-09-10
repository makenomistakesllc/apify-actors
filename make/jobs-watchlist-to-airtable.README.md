# Jobs Watchlist → Airtable (Make blueprint)

**File:** `jobs-watchlist-to-airtable.blueprint.json`

## What it does
Three-module scenario: an HTTP call that runs `make_no_mistakes/multi-ats-job-board-api`
synchronously against a company watchlist with `postedWithinDays: 1`, a built-in Iterator that
fans the returned array out one job at a time, and an HTTP `PATCH` to Airtable's REST API using
its `performUpsert` option (merge key: `ats` + `job_id`) so re-runs update existing rows instead
of duplicating them.

## Why HTTP modules instead of Make's native Apify/Airtable apps
Same reasoning as the Slack blueprint: this uses the generic HTTP module for both calls so the
file imports and runs with two pasted tokens and no OAuth setup, and so the request bodies here
could be verified against the actor directly. Make's native "Apify" app (Run an Actor / Get
Dataset Items) and Airtable app (Create/Update/Search a Record) both exist and work fine if you'd
rather swap them in after import for a nicer UI — the Iterator module stays either way.

## Required credentials (paste into the HTTP modules — no Make app connection needed)
- **Apify API token** — goes in module 1's Authorization header (`Bearer <token>`).
- **Airtable Personal Access Token** with `data.records:write` scope on the target base — goes in
  module 3's Authorization header. Set the real base ID and table name in module 3's URL
  (`REPLACE_WITH_BASE_ID`, `REPLACE_WITH_TABLE_NAME`), and make sure that table has columns
  matching the mapped fields (company, ats, job_id, title, department, location_raw, is_remote,
  employment_type, salary_min, salary_max, salary_currency, posted_at, apply_url).

## Setup after import
1. Import the blueprint into a new scenario.
2. Module 1: paste your Apify token, edit the `companies` array to your real watchlist.
3. Module 3: paste your Airtable token, fill in the base ID and table name in the URL. The method
   is already set to `PATCH` and the body already carries `performUpsert.fieldsToMergeOn: ["ats",
   "job_id"]` — Airtable's API requires both for upsert-by-key to work; don't drop either.
4. Click the scenario's clock icon and set the schedule to **daily**.

## Cost per run
Same actor pricing as the n8n version: $0.005/run start + $0.004/job returned. A
`postedWithinDays: 1` sweep of 5 companies typically returns a handful to a few dozen new jobs:
- 20 new jobs/day ≈ **$0.005 + $0.08 = $0.085/run** → ≈$2.55/month
- 100 new jobs/day ≈ **$0.405/run** → ≈$12/month

Make's own operation cost scales with Iterator cycles as in the Slack blueprint — check your plan's
operations quota for a large watchlist.
