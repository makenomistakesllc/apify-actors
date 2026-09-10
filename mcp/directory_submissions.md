# Directory & list submissions for the two actors

Ten concrete, currently-real submission targets (checked live Sep 2026), covering every category
the task named: Apify's own surfaces, n8n/Make template libraries, GitHub awesome-lists, MCP-
specific directories, Postman, and RapidAPI. Ranked by an internal research note's own hierarchy
(§1 move #5 + §4a): **templates first, directories are backlinks,
not a channel.** Items 1–2 (the templates) are the ones with a demonstrated revenue effect;
items 3–10 are near-free supply-side listings — do them, but don't expect them to move a needle on
their own.

Brand identity used throughout: **Make No Mistakes** (matches the Apify Store username
`make_no_mistakes` both actors are already published under). Every submission below uses that
brand name, never a personal one.

| # | Directory | URL | Requires | Anonymity-safe? | Notes |
|---|---|---|---|---|---|
| 1 | n8n Community Templates | [n8n.io/workflows](https://n8n.io/workflows) via [Creator Hub](https://n8n.io/creators) | n8n Cloud account (brand email) | **Yes** | Submit all 4 `n8n/*.json` files. Free; paid creator-affiliate tier unlocks after 3 templates. |
| 2 | Make.com Public Template Library | [make.com/en/templates](https://www.make.com/en/templates) | Make account (brand email) | **Yes** | Submit the 2 `make/*.blueprint.json` scenarios as a "Team Template," then request public review. |
| 3 | Apify Store — Integrations tags | Actor detail pages (Console → Actor → Publication) | Existing Apify account (already brand-owned) | **Yes** | Self-serve toggle, no new account. Tag both actors' listings as n8n/Make-compatible; add a one-line "Works in n8n and Make — see `github.com/<brand>/apify-actors-integrations`" pointer once the repo below exists. |
| 4 | `punkpeye/awesome-mcp-servers` (GitHub) | [github.com/punkpeye/awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers) | **Brand GitHub account + PR** | Yes, if brand account used | ⚠️ **Needs the brand GitHub account, not a personal one.** See PR body below. |
| 5 | `lorien/awesome-web-scraping` (GitHub) | [github.com/lorien/awesome-web-scraping](https://github.com/lorien/awesome-web-scraping) | **Brand GitHub account + PR** | Yes, if brand account used | ⚠️ Same brand-GitHub requirement. See PR body below. |
| 6 | mcp.so | [mcp.so/submit?type=server](https://mcp.so/submit?type=server) | Web form, no account | **Yes** | Fastest of the ten — no login at all. |
| 7 | mcp.directory | [mcp.directory/submit](https://mcp.directory/submit) | Web form; free tier auto-pulls metadata from a public GitHub repo URL | Yes, once a public repo exists | Needs a public GitHub URL for each actor — use the brand GitHub org's repo (same one from #4/#5) as the source link. Premium tier (paid) publishes instantly instead of queued review; skip paid, free queue is fine. |
| 8 | Smithery.ai | [smithery.ai](https://smithery.ai) via `smithery mcp publish` | Smithery account — **likely GitHub OAuth login** | Check before using — GitHub OAuth may bind to whichever GitHub account signs in | ⚠️ If Smithery only offers GitHub OAuth (verify at signup), use the **brand** GitHub account to authenticate, same as #4/#5. |
| 9 | Postman Public API Network | [learning.postman.com/docs/postman-api-network](https://learning.postman.com/docs/postman-api-network/showcase/publish/public-apis/) | Postman account (brand email) + public workspace | **Yes** | Publish `postman/apify-actors.postman_collection.json` as a public collection, then enable "Allow Collection Discovery" → "Add to API Network." |
| 10 | RapidAPI Hub | [docs.rapidapi.com/docs/add-api-getting-started](https://docs.rapidapi.com/docs/add-api-getting-started) | RapidAPI provider account (brand email) + a backend to bill through | **Yes**, but heaviest setup | RapidAPI expects *you* to be the origin server it proxies to and bills through — point the listing at the same `run-sync-get-dataset-items` endpoints with your Apify token baked into the backend config. More setup than the others; do it last. |

---

## Ready-to-paste text

### #4 — PR body for `punkpeye/awesome-mcp-servers`

> **Title:** Add Make No Mistakes — US Building Permits & Multi-ATS Job Board actors
>
> **Body:**
> Adding two Apify Actors that are callable as MCP tools via the hosted Apify MCP server
> (`https://mcp.apify.com`).
>
> - **US Building Permits Scraper** (`make_no_mistakes/us-building-permits-scraper`) — normalized
>   building/construction permits from 11 US cities' official open-data APIs, one schema, no HTML
>   scraping. [Apify Store link] · [Docs]
> - **Multi-ATS Job Board API** (`make_no_mistakes/multi-ats-job-board-api`) — resolves a company
>   name or domain to its applicant tracking system (9 platforms: Greenhouse, Ashby, Lever,
>   Workday, SmartRecruiters, Recruitee, Rippling, Workable, Personio) and returns every open job
>   on one schema. [Apify Store link] · [Docs]
>
> Both connect through Apify's MCP server — no separate install, auth via Apify API token or
> OAuth. Added under the Apify/data-access section, alphabetical order preserved. Checked for
> existing duplicates first; neither actor is currently listed.

*(Fill the two `[Apify Store link]` / `[Docs]` placeholders with the live Store URLs before
opening the PR — Console → Actor → the public Store URL.)*

### #5 — PR body for `lorien/awesome-web-scraping`

> **Title:** Add two normalized-API Apify Actors (building permits, ATS job boards)
>
> **Body:**
> Two Actors that scrape via documented public APIs (Socrata/CKAN/Carto for permits; each ATS
> vendor's own job-board API) rather than HTML, normalized onto a single schema per category:
>
> - US Building Permits Scraper — 11 US cities' permit registers → one schema.
>   `make_no_mistakes/us-building-permits-scraper` on Apify. [link]
> - Multi-ATS Job Board API — auto-detects which of 9 ATS platforms an employer uses and returns
>   every open job on one schema. `make_no_mistakes/multi-ats-job-board-api` on Apify. [link]
>
> Both are "scrape a public API, not a page" tools, which seemed like a fit for this list's
> API-based section rather than the HTML-scraping one. Happy to move them if there's a better spot.

### #6 — mcp.so submission form fields

> **Name:** US Building Permits Scraper (Apify Actor)
> **Description:** Building and construction permits from 11 US cities' official open-data APIs,
> normalized onto one schema — address, valuation, contractor name/phone, work description. Callable
> as an MCP tool via `https://mcp.apify.com?tools=make_no_mistakes/us-building-permits-scraper`.
> **Category:** Data / Web Scraping
> **Repo/Homepage:** [Apify Store URL]
>
> *(repeat as a second submission for the Multi-ATS Job Board API, swapping the description for:
> "Resolves a company name or domain to its applicant tracking system across 9 platforms and
> returns every open job on one schema. MCP: `https://mcp.apify.com?tools=make_no_mistakes/multi-ats-job-board-api`.")*

### #9 — Postman collection description (paste into the collection's public description field)

> Run either of two Apify Actors synchronously and get results back in one call — no polling, no
> separate dataset fetch. **US Building Permits Scraper** returns normalized permits from 11 US
> cities; **Multi-ATS Job Board API** resolves a company to its ATS and returns every open job.
> Both are pay-per-event (see each request's description for current pricing). Fork this
> collection, set your Apify API token as the `apify_token` collection variable, and send.

### #2 — Make.com template submission descriptions

> **Permits New Contractors to Slack:** Daily automation that pulls Austin's newest building
> permits, keeps the ones with a dialable contractor phone number, and posts each new lead to a
> Slack channel — a live contractor call list with zero manual pulling.
>
> **Jobs Watchlist to Airtable:** Daily automation that checks a company watchlist for job postings
> across 9 ATS platforms (Greenhouse, Ashby, Lever, Workday and 5 more) and keeps an Airtable base
> in sync — new postings inserted, existing ones refreshed, nothing duplicated.

### #1 — n8n template sticky-note text (paste as the template's main sticky note)

> **What it does:** [see each template's own README.md in `../n8n/` — copy the "What it does"
> section verbatim, it's already written in the right voice for this field.]
> **Setup:** [copy that template's "Setup after import" list.]

---

## What NOT to do here

Per §4a of the research doc: *"there's significant SUPPLY for what he built… the hard part is the
demand"* (swyx, on MCP directory submissions specifically). Don't spend more than the ~2–3 hours
total these ten submissions take, and don't treat a rising submission count as a success metric.
The only measurable outcomes that matter are (a) whether the n8n/Make templates get **installed**
by someone besides us, and (b) whether any of these listings show up as a **referrer** in Apify's
own Store analytics. Check both in 60 days; if a listing shows zero installs and zero referral
traffic, leave it — don't re-submit or bump it.
