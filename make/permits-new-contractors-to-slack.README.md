# Permits New Contractors → Slack (Make blueprint)

**File:** `permits-new-contractors-to-slack.blueprint.json`

## What it does
Three-module scenario: an HTTP call that runs `make_no_mistakes/us-building-permits-scraper`
synchronously for Austin's last 1 day of permits, a built-in Iterator that fans the returned array
out one item at a time, and an HTTP call to Slack's `chat.postMessage` endpoint per item.

## Why HTTP modules instead of Make's native Apify/Slack apps
Make does have native "Apify" and "Slack" apps (confirmed current as of Sep 2026 — Apify: Run an
Actor / Get Dataset Items / Watch Actor Runs; Slack: Create a Message and friends). This blueprint
uses the generic, universal **HTTP** module for both calls instead, because:
- It imports and runs correctly with nothing but two pasted API tokens — no OAuth dance, no
  picking the right native-app module version.
- It was possible to test and verify the exact request/response shape against the actor (below)
  without a live Make account to check the native app's current field names against.

If you'd rather use the native apps for a nicer point-and-click UI: after import, replace module 1
with Apify's "Run an Actor and Get Dataset Items" action (connect with your Apify API token), and
replace module 3 with Slack's "Create a Message" action (connect via Slack OAuth). The Iterator
stays either way.

## Required credentials (paste into the HTTP modules — no Make app connection needed)
- **Apify API token** — Apify Console → Settings → API & Integrations. Goes in module 1's
  Authorization header (`Bearer <token>`).
- **Slack Bot Token** with `chat:write` scope, bot invited to the target channel — goes in module
  3's Authorization header. Set the real channel ID in module 3's `channel` field
  (`REPLACE_WITH_SLACK_CHANNEL_ID`).

## Setup after import
1. Import the blueprint into a new scenario.
2. Module 1: paste your Apify token into the Authorization header.
3. Add a **Filter** on the connector between module 2 (Iterator) and module 3 (Slack HTTP call):
   condition `contractor_phone` is not empty. Without this filter you'll get a Slack message for
   every permit, not just the ones with a callable contractor.
4. (Optional, recommended) Add a Make **Data Store** lookup before module 3 keyed on
   `contractor_phone` + `permit_number`, so a permit already posted doesn't post again on the next
   run — the n8n version of this template does this automatically with a built-in dedupe node;
   Make requires wiring a Data Store explicitly (Tools → Data Store, then a "Search Records" +
   "Add Record" pair around the Slack call).
5. Module 3: paste your Slack Bot Token and set the real channel ID.
6. Click the scenario's clock icon and set the schedule to **daily** — Make stores this outside
   the blueprint JSON, so it isn't preset by the import.

## Cost per run
Same actor pricing as the n8n version: $0.02/run start + $0.005/permit returned. Austin's ~165
permits/day (before the phone filter) costs **≈$0.02 + $0.83 = $0.85/run**, ≈$26/month daily.
Make's own operation cost is separate and scales with the number of Iterator cycles (one operation
per item per module downstream) — check your Make plan's operations quota if the daily volume is
large.
