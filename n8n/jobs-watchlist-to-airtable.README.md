# Jobs Watchlist → Airtable

**File:** `jobs-watchlist-to-airtable.json`

## What it does
Every day at 6:00 AM, runs `make_no_mistakes/multi-ats-job-board-api` against a fixed company
watchlist with `postedWithinDays: 1`, and upserts each returned job into an Airtable table, keyed
on `ats` + `job_id` so re-running never creates duplicate rows — a job already logged gets its
fields refreshed instead.

Default watchlist in the template: `stripe, openai, ramp, anthropic, databricks`. Edit the
`companies` array inside **Run ATS Watchlist**'s Input JSON — names, domains, board slugs or full
board URLs all work; see the actor's README for how resolution works.

## Required credentials
- **Apify account** — `apifyApi`.
- **Airtable personal access token** — n8n's built-in Airtable node, credential type
  `airtableTokenApi`. Create a base with a table (default name `Jobs`) that has at least columns
  matching the mapped fields: company, ats, job_id, title, department, location_raw, is_remote,
  employment_type, salary_min, salary_max, salary_currency, posted_at, apply_url.

## Setup after import
1. Open **Run ATS Watchlist**, attach your Apify credential, confirm the Input JSON and edit the
   watchlist.
2. Open **Upsert to Airtable**, attach your Airtable credential, and set `base` / `table` to your
   own (replace `REPLACE_WITH_BASE_ID` / `REPLACE_WITH_TABLE_ID`). Confirm `matchingColumns` is
   set to `["ats", "job_id"]` — that pair is the actor's natural unique key for a job.
3. Activate the workflow.

## Cost per run
Pricing is $0.005/run start + $0.004/job returned. A `postedWithinDays: 1` sweep of 5 companies
typically returns a handful to a few dozen new jobs, not the whole board:
- 20 new jobs/day ≈ **$0.005 + $0.08 = $0.085/run** → ≈$2.55/month
- 100 new jobs/day (larger watchlist or a hiring surge) ≈ **$0.405/run** → ≈$12/month

`maxItems` is set to 1000 as a safety ceiling — with `postedWithinDays: 1` you will almost never
approach it unless the watchlist is very large.
