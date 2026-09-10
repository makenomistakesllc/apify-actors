# Jobs Title Alert → Email

**File:** `jobs-title-alert-to-email.json`

## What it does
Every day at 8:00 AM, runs `make_no_mistakes/multi-ats-job-board-api` with `titleIncludes` set to
a watch list of job titles (default: `staff engineer`, `principal engineer`) and
`postedWithinDays: 1`, builds an HTML digest of the matches with a **Code** node, and emails it —
but only on days something actually matched. The Code node returns an empty item list when there
are zero matches, which stops the workflow before the email node runs, so you get silence instead
of an empty "0 new jobs" email every morning.

## Required credentials
- **Apify account** — `apifyApi`.
- **SMTP** — n8n's built-in Send Email node (credential type `smtp`). Any SMTP relay works
  (Gmail app password, SendGrid, Postmark, etc.) — no OAuth required.

## Setup after import
1. Open **Run Title Alert Search**, attach your Apify credential, and edit `companies` and
   `titleIncludes` to the roles and employers you actually want to track.
2. Open **Send Digest Email**, attach your SMTP credential, and replace the placeholder
   `fromEmail` / `toEmail` addresses.
3. Activate the workflow.

## Notes on title matching
`titleIncludes` is an OR match, case-insensitive, against the job title only — it does not read
the description. `["staff engineer", "principal engineer"]` will also match "Staff Engineer,
Platform" but not "Senior Staff-Level Engineer" (no exact substring). Keep the list short and
literal; broaden with a Filter/Code node downstream if you need fuzzier matching.

## Cost per run
Same pricing as the Airtable template: $0.005/run + $0.004/job **returned**, and `titleIncludes`
filtering happens actor-side, so unmatched jobs are never written or billed. An 8-company sweep
with a narrow title filter typically returns 0–5 matches/day:
- 0 matches ≈ **$0.005/run** (no email sent) → ≈$0.15/month
- 5 matches ≈ **$0.005 + $0.02 = $0.025/run** → ≈$0.75/month

This is one of the cheapest possible uses of the actor because the title filter, not `maxItems`,
does almost all of the cost-limiting work.
