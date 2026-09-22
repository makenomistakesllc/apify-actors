# Permits Weekly → Google Sheets

**File:** `permits-weekly-to-google-sheets.json`

## What it does
Every Monday at 7:00 AM (workflow timezone), runs `make_no_mistakes/us-building-permits-scraper`
for a chosen list of cities over the trailing 7 days and appends one row per permit to a Google
Sheet. This is the low-effort "have a live permit feed in a spreadsheet" version of the actor —
no code, no database.

Default cities in the template: `austin, san-antonio, new-york, chicago, boston`. Edit the `cities`
array inside the **Run Permits Actor** node's Input JSON to change coverage — see the actor's
README for all 16 supported city keys.

## Required credentials
- **Apify account** (API token) — n8n community node `@apify/n8n-nodes-apify`, credential type
  `apifyApi`. Get the token from Apify Console → Settings → API & Integrations.
- **Google Sheets OAuth2** — n8n's built-in Google Sheets node. Connect your Google account and
  point `documentId` / `sheetName` at your own spreadsheet (create one with a header row matching
  the columns mapped in the node: city, permit_number, permit_type, permit_class,
  work_description, status, issued_date, address, zip, contractor_name, contractor_phone,
  valuation, owner_name, source_url).

## Setup after import
1. Install the community node `@apify/n8n-nodes-apify` if not already installed (Settings →
   Community Nodes).
2. Open **Run Permits Actor**, reselect the Actor from the picker if prompted, confirm the Input
   JSON, and attach your Apify credential.
3. Open **Append to Google Sheets**, attach your Google Sheets credential, and set `documentId`
   and `sheetName` to your own spreadsheet.
4. Activate the workflow.

## Cost per run
5 cities × 7 days is typically a few hundred to ~1,500 permits depending on the week (see the
actor's per-city permits/week table). At $0.02 per run start + $0.008/permit:
- 500 permits ≈ **$0.02 + $4.00 = $4.02/run** → ~$17/month at 1 run/week
- 1,500 permits ≈ **$0.02 + $12.00 = $12.02/run** → ~$52/month at 1 run/week

Set `maxItems` (default 5000 in this template) lower if you want a hard ceiling on spend
regardless of volume.
