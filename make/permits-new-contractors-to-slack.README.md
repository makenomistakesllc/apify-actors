# Permits New Contractors → Slack (Make blueprint)

**File:** `permits-new-contractors-to-slack.blueprint.json`
**Public Make template:** ID 19724 — **live, approved Sep 16 2026** (ticket #2307299)
**Public URL:** https://www.make.com/en/integration/19724-permits-new-contractors-to-slack
(logged-in users are region-redirected to `https://us2.make.com/templates/19724`)
**Editor:** `us2.make.com/2954832/templates/16863/edit`

## What it does
Three-module scenario: an HTTP call that runs `make_no_mistakes/us-building-permits-scraper`
synchronously, a built-in Iterator that fans the returned array out one item at a time, and an
HTTP call to Slack's `chat.postMessage` endpoint per item.

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
| `method: "POST"` (upper case) | `method: "post"` (lower case) |
| — | `mapper.stopOnHttpError`, built-in pagination, `parameters.proxyKeychain` |

`parseResponse` and `shareCookies` carry over unchanged.

**Note:** the v4 "Body content type" select does not reliably round-trip through a template save,
so both modules also send an explicit `Content-Type: application/json` header. Keep it.

## Wizard fields (what the template asks the user for)
Marked in the template editor with **Use in Wizard** + **Use as default value**, each with help
text:

| Module | Field | Help text shown to the user |
|---|---|---|
| 1 – Run Permits Actor | `Headers → Authorization → Value` | Your Apify API token. Get it at https://console.apify.com/settings/integrations (Personal API tokens). Replace `YOUR-APIFY-API-TOKEN`, keeping the word `Bearer` and the space in front of it. |
| 3 – Post to Slack | `Headers → Authorization → Value` | Your Slack bot token. Create a Slack app at https://api.slack.com/apps, add the `chat:write` scope under OAuth & Permissions, install it to your workspace, and invite the bot to the target channel. Replace `xoxb-YOUR-SLACK-BOT-TOKEN`, keeping the `Bearer ` prefix. |

The template description (Set up a template → Template description) repeats both token links and
adds the setup steps, the Slack channel ID step, and the "edit `cities` / `daysBack`" note.

## Required credentials
- **Apify API token** — Apify Console → Settings → Integrations → Personal API tokens. Goes in
  module 1's `Authorization` header (`Bearer <token>`).
- **Slack Bot Token** with `chat:write`, bot invited to the target channel — goes in module 3's
  `Authorization` header. Set the real channel ID in module 3's `channel` field
  (`REPLACE_WITH_SLACK_CHANNEL_ID`).

No Make connection or keychain is required: authentication type is **No authentication** and both
tokens are plain header values, so nothing is stored in the Make account and nothing is stripped
when the template is published.

## Setup after import
1. Import the blueprint into a new scenario (or start from the public template).
2. Module 1: paste your Apify token into the Authorization header; edit `cities` (16 US cities
   supported) and `daysBack` if you want a different scope.
3. Add a **Filter** on the connector between module 2 (Iterator) and module 3: condition
   `contractor_phone` is not empty.
4. (Optional) Add a Make **Data Store** lookup before module 3 keyed on `contractor_phone` +
   `permit_number` so an already-posted permit does not post again.
5. Module 3: paste your Slack Bot Token and set the real channel ID.
6. Click the scenario's clock icon and set the schedule to **daily** — Make stores this outside
   the blueprint JSON.

## Cost per run
$0.02/run start + $0.005/permit returned. Austin's ~165 permits/day (before the phone filter)
costs **≈$0.02 + $0.83 = $0.85/run**, ≈$26/month daily. Make's own operation cost is separate and
scales with the number of Iterator cycles.
