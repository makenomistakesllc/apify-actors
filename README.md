# Apify Actors — Integration Kit

Integration templates and reference material for three [Apify](https://apify.com) Actors published
under the **Make No Mistakes** Store account. This repo does not contain the Actors' source code —
it contains the templates, configs and docs that make them easy to plug into workflow tools,
AI agents, and API clients.

## The Actors

### [US Building Permits Scraper](https://apify.com/make_no_mistakes/us-building-permits-scraper)
Normalized building and construction permit data from 16 US cities' official open-data APIs
(Socrata, ArcGIS REST, CKAN, Carto): Austin, San Antonio, New York City, Los Angeles, Chicago,
Philadelphia, Boston, Seattle, San Francisco, Cincinnati, Baton Rouge, Washington DC, Denver,
Nashville, Raleigh and Louisville. One call, one schema: permit number, type, work description,
status, dates, address, valuation, owner name, and contractor name/phone where the city publishes
it (Austin and Raleigh are the only two that publish phone numbers). No HTML scraping, no proxies,
no logins.

### [Multi-ATS Job Board API](https://apify.com/make_no_mistakes/multi-ats-job-board-api)
Give it a company name or domain and it resolves which applicant tracking system that company
uses — across Greenhouse, Ashby, Lever, Workday, SmartRecruiters, Recruitee, Rippling, Workable
and Personio — then returns every open job on one normalized schema. No need to already know the
company's board token or ATS platform; that lookup is what the Actor does.

### [Company ATS Detector](https://apify.com/make_no_mistakes/company-ats-detector)
The free companion to the Multi-ATS Job Board API. Give it a company domain, name or careers-page
URL and it tells you which of the same nine ATS platforms (Greenhouse, Ashby, Lever, Workday,
SmartRecruiters, Recruitee, Rippling, Workable, Personio) that company's job board runs on, the
board URL, the public no-auth API endpoint, and the current open-job count. It runs the same
discovery code as the Multi-ATS Job Board API but stops at "which platform and how many jobs" rather
than returning full postings, so it costs nothing beyond your own Apify platform usage. Free, no
pay-per-event pricing.

Full docs for all three, including input schemas and pricing, are mirrored in [`docs/`](docs/) and
live on their Apify Store pages linked above.

## What's in this repo

| Path | What it is |
|---|---|
| [`n8n/`](n8n/) | Importable n8n workflow templates (JSON), each with its own README. |
| [`make/`](make/) | Importable Make.com blueprints, each with its own README. |
| [`mcp/`](mcp/) | How to call all three Actors through Apify's hosted MCP server: config examples for Claude Desktop, Cursor, VS Code, Claude Code and a local stdio fallback. |
| [`postman/`](postman/) | A Postman collection with all three Actors' `run-sync-get-dataset-items` calls, pre-filled with working example request bodies. |
| [`docs/`](docs/) | Mirrored copies of all three Actors' full Apify Store READMEs (input schemas, examples, pricing). |

## How to use each template

**n8n**: In n8n, go to **Workflows → Import from File** and select a `.json` file from `n8n/`.
Each workflow needs an Apify API token (n8n credential type `apifyApi`) and any downstream
credentials the template uses (Slack, Airtable, Google Sheets, SMTP) — see that template's own
README in the same folder for the exact setup steps and field mapping.

**Make.com**: In Make, go to **Scenarios → Create a new scenario → Import Blueprint** and select a
`.blueprint.json` file from `make/`. These blueprints call Apify's REST API directly via Make's
generic HTTP module (rather than a native Apify app module), so all you need is an Apify API token
pasted into the HTTP module's header — see each blueprint's README for details.

**MCP (AI agents / Claude, Cursor, etc.)**: See [`mcp/README.md`](mcp/README.md) for the hosted
MCP server URL, scoping all three Actors as callable tools, and config snippets for every major
client.

**Postman**: Import [`postman/apify-actors.postman_collection.json`](postman/apify-actors.postman_collection.json),
set an `apify_token` collection variable to your own Apify API token, and run any of the three
requests.

Every template calls the Actors under your own Apify account and API token — nothing here embeds
or requires ours. The US Building Permits Scraper and Multi-ATS Job Board API are pay-per-event;
the Company ATS Detector is free. See the Store pages for current pricing.

## License

MIT — see [`LICENSE`](LICENSE).
