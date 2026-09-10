# Permits New Contractors → Slack

**File:** `permits-new-contractors-to-slack.json`

## What it does
Every day at 7:30 AM, pulls the last 1 day of Austin permits (the only city in the actor with
contractor phone numbers), keeps only rows with a non-empty `contractor_phone`, drops anything
already posted on a previous run using n8n's built-in **Remove Duplicates** node (compared on
`contractor_phone` + `permit_number`, no external sheet needed), and posts one Slack message per
genuinely new contractor lead.

This is the shape of a small internal lead-gen tool: a live list of dialable contractor names and
numbers tied to a specific permit, delivered where the person who calls them already looks.

## Required credentials
- **Apify account** — same as above, `apifyApi`.
- **Slack** — n8n's built-in Slack node (OAuth2 or Bot Token). The bot needs
  `chat:write` scope and must be invited to the target channel.

## Setup after import
1. Open **Run Austin Permits**, attach your Apify credential, confirm the Input JSON.
2. Open **Dedupe New Contractors** — this node's "seen before" memory is scoped to the workflow
   and lives inside n8n itself. **The first run after import will post every matching contractor
   as if new** (there's no history yet); that's expected and only happens once.
3. Open **Post New Contractor**, attach your Slack credential, and pick the real channel (replace
   the placeholder `REPLACE_WITH_CHANNEL_ID`).
4. Activate the workflow.

## Why Austin only
Austin is the one city in `us-building-permits-scraper` that publishes contractor phone numbers
(92% coverage). Every other supported city gives a contractor name only — useful for matching
against a license board, not for calling directly. Swap the `cities` array and drop the
`contractor_phone` filter if you want a name-only version for another metro.

## Cost per run
Austin issues roughly ~1,150 permits/week (~165/day). At $0.02 + $0.005/permit, a full day's pull
before filtering costs **≈$0.02 + $0.83 = $0.85/run**, ≈$26/month daily. Filtering happens inside
n8n after the actor returns data, so the actor cost doesn't shrink with the phone/dedupe filters —
only the number of Slack messages does. Lower `maxItems` (default 1000, well above one day's
volume) if you want a hard cap.
