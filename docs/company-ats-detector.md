# Company ATS Detector: which job board does a company use? (Greenhouse, Lever, Ashby, Workday, SmartRecruiters, Workable, Recruitee, Personio, Rippling)

> Mirrored from the Actor's own README, build 0.1.3 (Apify Store page is the live source of truth).

**Which ATS does a company use?** Paste a domain, a company name or a careers-page URL and this
Actor tells you which applicant tracking system that employer's job board runs on, where the board
lives, and how many roles are open on it right now.

Stripe is on **Greenhouse** under `stripe`. Notion is on **Ashby** under `notion`. Match Group is on
**Lever** under `matchgroup`. NVIDIA is on **Workday** at `nvidia.wd5.myworkdayjobs.com`. You cannot
scrape any of those boards until you know which platform the employer is on and what its board slug
is, and there is no public directory that will tell you. That lookup is what this Actor does.

Every resolved company comes back with the platform slug, the human board URL, the **public
zero-auth API endpoint** you can curl yourself, the board slug, and the live open-job count. Nine
platforms are probed. It is free: you pay only your own Apify platform usage, and no API keys,
proxies, logins or CAPTCHAs are involved on any of them.

A company with no public board comes back as a row with `resolved: false`, never as a missing row.

---

## For AI agents and MCP clients

Resolves company domains or names to the applicant tracking system behind their careers page, and
returns the public no-auth API endpoint that serves that board's postings.

Minimal input that returns something useful:

```json
{ "companies": ["stripe.com", "Notion", "nvidia.com"] }
```

You get one dataset item per input, in input order, with `input`, `company_domain`, `ats`,
`board_url`, `api_url`, `board_slug`, `job_count`, `detected_via`, `resolved`, `error` and
`checked_at`. `ats` is one of `greenhouse`, `ashby`, `lever`, `smartrecruiters`, `recruitee`,
`rippling`, `personio`, `workday`, `workable`, or `null` when nothing public was found.

Cost: free. There is no pay-per-event pricing on this Actor, so a run costs only the Apify compute
it uses. A measured 20-company run took 33 seconds and 0.0046 compute units, about a tenth of a
cent.

Typical questions it answers: which ATS does this company use; what is the API endpoint for this
company's job board; which of these 200 companies are on Greenhouse; which of my target accounts
are hiring at all right now, and how heavily.

**Then fetch the jobs:** this Actor identifies the board.
[multi-ats-job-board-api](https://apify.com/make_no_mistakes/multi-ats-job-board-api) returns every
posting on it, normalized across all nine platforms, at $0.004 per job.

---

## What it detects

Nine platforms, each through the public job-board API its vendor publishes for aggregators:

| ATS | Board URL shape | Public API endpoint returned in `api_url` |
|---|---|---|
| `greenhouse` | `job-boards.greenhouse.io/{slug}` | `boards-api.greenhouse.io/v1/boards/{slug}/jobs` |
| `ashby` | `jobs.ashbyhq.com/{slug}` | `api.ashbyhq.com/posting-api/job-board/{slug}` |
| `lever` | `jobs.lever.co/{slug}` | `api.lever.co/v0/postings/{slug}?mode=json` |
| `smartrecruiters` | `jobs.smartrecruiters.com/{slug}` | `api.smartrecruiters.com/v1/companies/{slug}/postings` |
| `recruitee` | `{slug}.recruitee.com` | `{slug}.recruitee.com/api/offers/` |
| `rippling` | `ats.rippling.com/{slug}/jobs` | `api.rippling.com/platform/api/ats/v1/board/{slug}/jobs` |
| `personio` | `{slug}.jobs.personio.de` | `{slug}.jobs.personio.de/xml` (XML, not JSON) |
| `workday` | `{tenant}.wd{N}.myworkdayjobs.com/{site}` | `{tenant}.wd{N}.myworkdayjobs.com/wday/cxs/{tenant}/{site}/jobs` (POST) |
| `workable` | `apply.workable.com/{slug}` | `apply.workable.com/api/v1/widget/accounts/{slug}?details=true` |

Every `api_url` is a plain GET with no key, no token and no header, except Workday, which is a POST
with an empty JSON body, and Personio, which answers XML.

---

## Need the actual job postings on one schema?

This Actor answers **where the board is**. Its paid sibling answers **what is on it**.

> ### [multi-ats-job-board-api](https://apify.com/make_no_mistakes/multi-ats-job-board-api) - $0.004 per job
>
> Same nine platforms, same automatic company-to-ATS resolution, but it returns every open posting
> on one normalized schema: title, department, team, city, state, country, remote flag, employment
> type, salary range with currency and period, posted and updated dates, full description, the real
> apply URL, and the untouched platform record under `raw`. Filters for posted-within-days, title
> keywords, location and remote-only. One run across a 40-company watchlist, one CSV out.
>
> Detect here for free, then hand the same company list straight to it.

---

## Input

```json
{
  "companies": [
    "stripe.com",
    "Notion",
    "https://jobs.lever.co/matchgroup",
    "https://www.figma.com/careers",
    "nvidia.com"
  ],
  "includeJobCount": true,
  "maxCompanies": 100
}
```

| Field | Type | Default | Notes |
|---|---|---|---|
| `companies` | array of string | `["stripe.com","Notion","https://jobs.lever.co/matchgroup"]` | Domains, company names, board slugs, board URLs or careers-page URLs. Mix them freely. |
| `includeJobCount` | boolean | `true` | Count the open roles on each detected board. For a domain or a name this is free: the request that identifies the board is the same request that counts it. Turning it off only saves a request when the input is already a board URL. |
| `maxCompanies` | integer | `100` | Hard cap on inputs checked per run. `0` means no cap. Anything past the cap is skipped, listed in `RUN_SUMMARY.companies_not_checked` and logged at ERROR level, never quietly reported as "no board found". |

All four input forms work and can be mixed in one run:

* **domain** - `stripe.com`, `vandebron.nl`
* **company name** - `Notion`, `Match Group`
* **careers-page URL** - `https://www.figma.com/careers`
* **board URL or slug** - `https://jobs.lever.co/matchgroup`, `ramp`

---

## Output fields

One row per input, in input order, always. Every row carries every field, so the dataset exports to
CSV or a database without ragged columns.

| Field | Type | Description |
|---|---|---|
| `input` | string | The company string exactly as you passed it, so a row always joins back to your list. |
| `company_domain` | string or null | The employer's own domain when the input carried one. Null for a bare company name, and null for an ATS board URL, because that host belongs to the ATS vendor rather than the employer. Never guessed. |
| `ats` | string or null | `greenhouse`, `ashby`, `lever`, `smartrecruiters`, `recruitee`, `rippling`, `personio`, `workday`, `workable`, or null. |
| `board_url` | string or null | The human careers board a candidate would open. |
| `api_url` | string or null | The public zero-auth endpoint serving that board's postings. See the table above. |
| `board_slug` | string or null | The employer's account name on that platform. This is the token every other job-board scraper makes you supply by hand. |
| `job_count` | integer or null | Open roles at `checked_at`. Null when `includeJobCount` is off, and when a board URL was accepted without a confirming request. |
| `detected_via` | string or null | `board_url`, `slug_probe` or `careers_page_link`. Null when unresolved. |
| `resolved` | boolean | True when a board was found and confirmed against the platform's own API. |
| `error` | string or null | Set only when something went wrong, such as a timeout or an unreachable careers page. An ordinary miss is `resolved: false` with `error: null`. |
| `checked_at` | string | UTC ISO timestamp of the check. |

A resolved row:

```json
{
  "input": "stripe.com",
  "company_domain": "stripe.com",
  "ats": "greenhouse",
  "board_url": "https://job-boards.greenhouse.io/stripe",
  "api_url": "https://boards-api.greenhouse.io/v1/boards/stripe/jobs",
  "board_slug": "stripe",
  "job_count": 676,
  "detected_via": "slug_probe",
  "resolved": true,
  "error": null,
  "checked_at": "2026-09-22T20:52:04+00:00"
}
```

An honest miss, which is still a row:

```json
{
  "input": "doordash.com",
  "company_domain": "doordash.com",
  "ats": null,
  "board_url": null,
  "api_url": null,
  "board_slug": null,
  "job_count": null,
  "detected_via": null,
  "resolved": false,
  "error": null,
  "checked_at": "2026-09-22T20:52:04+00:00"
}
```

A `RUN_SUMMARY` record is written to the default key-value store with the row count, the resolved
count, the hit rate, rows per platform, how each company was detected, the unresolved list, any
inputs dropped by `maxCompanies`, and any per-input errors.

---

## How detection works

Three steps, cheapest first. Each one stops the moment it has an answer, and nothing is reported
until the platform's own API has confirmed the board exists.

1. **Board URL pattern.** If the input is already an ATS board URL, the platform and slug are read
   straight off it. `jobs.lever.co/matchgroup`, `boards.greenhouse.io/stripe`,
   `acme.recruitee.com`, `nvidia.wd5.myworkdayjobs.com/NVIDIAExternalCareerSite` and the rest are
   all recognised. `detected_via: board_url`.
2. **Direct board probes.** A company name or domain is reduced to a short ordered list of
   plausible board slugs: the domain label, the name lowercased and de-spaced, the hyphenated
   variant, and the variant with `Inc`, `GmbH`, `Group` and similar suffixes dropped. Those are
   probed against the platforms in three tiers, cheapest first: the six single-GET platforms, then
   Personio and Workable, then Workday. **One request per platform probe**, and the probe response
   is where `job_count` comes from, so the count is free. `detected_via: slug_probe`.
3. **Careers-page link patterns.** Only for companies step 2 missed, and only when the input gave
   us a domain. Up to three pages of the company's own site (`/careers`, `/jobs`, `/`) are fetched
   as plain HTML and scanned for a link to any of the nine platforms. Anything found is confirmed
   against that platform's API before it is reported. This is what catches an employer whose board
   slug looks nothing like its domain, for example `ouraring.com` linking a Greenhouse board under
   `oura`. `detected_via: careers_page_link`.

**Measured hit rate: 18 of 20 (90%)** on a mixed 20-company test run on 2026-09-22 covering all
nine platforms: stripe.com, notion.so, figma.com, nvidia.com, wandelbots.com, adverity.com,
vandebron.nl, rippling.com, gong.io, Vercel, a Lever board URL, ouraring.com, openai.com,
anthropic.com, databricks.com, cloudflare.com, salesforce.com, doordash.com, scale.com and a
nonsense domain. The two misses were **doordash.com** and the nonsense domain. DoorDash runs a
custom careers site that is not on any of the nine public job-board APIs, so there is nothing to
find, and saying so is the correct answer rather than a failure.

The resolution code is shared with the paid Actor
[multi-ats-job-board-api](https://apify.com/make_no_mistakes/multi-ats-job-board-api): the modules
under `src/discovery.py`, `src/http.py`, `src/normalize.py` and `src/platforms/` are copied from it
verbatim, with a header naming the origin file, and `ops/sync_vendored.py` fails loudly if the two
ever drift. A detector that disagrees with the scraper would be worse than no detector.

---

## Limits

**It only knows these nine platforms.** An employer running a custom careers site, or an ATS with
no public job-board API (Taleo, iCIMS, Jobvite, BambooHR and many more), comes back unresolved.
That is an answer, not an error.

**Workday resolution is fragile.** Workday is the only one of the nine with no documented public
job-board API and no directory of tenants. Every employer lives on its own host (`wd1` through
`wd12` in practice) under a site name the employer chose (`External`, `NVIDIAExternalCareerSite`,
`External_Career_Site`). Both are discovered by reading the tenant's `robots.txt` and falling back
to probing common site names. That is observed behaviour, not contract. If Workday changes it,
Workday companies start coming back unresolved while the other eight are unaffected.

**Personio and Workable rate-limit by IP.** Both answer HTTP 429 when requests arrive close
together. Calls to them are paced and retried, and they are probed after the cheaper platforms for
exactly this reason, but a large list that leans on those two can see an individual company come
back unresolved. Re-run it, or pass the board URL.

**A slug guess can belong to a different company.** `scale.com` resolves to a small Personio board
owned by a different company called Scale; Scale AI's real board is on Greenhouse under `scaleai`.
When a guessed slug is a real board there is no signal that distinguishes the two. If `job_count`
looks absurd for the employer, pass the board URL instead.

**The careers-page fallback reads HTML only.** No browser, no JavaScript. A careers page that
renders its board client-side, or that sits behind a bot wall returning 403, yields nothing. It is
a fallback that recovers some misses, not a guarantee.

**Job counts move.** A count is a snapshot at `checked_at`, not a promise about the next run.
Boards change hour to hour.

**Some employers run two boards at once.** Mid-migration boards are common. The larger board wins,
and the other one is not reported here. The paid Actor lists the alternates in `other_matches`.

---

## Use it from Python, curl, n8n, Make and MCP

### curl

```bash
curl -X POST "https://api.apify.com/v2/acts/make_no_mistakes~company-ats-detector/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"companies": ["stripe.com", "Notion", "nvidia.com"]}'
```

Then curl the endpoint it hands back, with no key at all:

```bash
curl -s "https://boards-api.greenhouse.io/v1/boards/stripe/jobs" | head -c 400
```

### Python

```python
from apify_client import ApifyClient

client = ApifyClient("<YOUR_APIFY_TOKEN>")
run = client.actor("make_no_mistakes/company-ats-detector").call(
    run_input={"companies": ["stripe.com", "Notion", "nvidia.com"], "includeJobCount": True}
)

for row in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(row["input"], "->", row["ats"], row["board_slug"], row["job_count"], row["api_url"])
```

### JavaScript

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: '<YOUR_APIFY_TOKEN>' });
const run = await client.actor('make_no_mistakes/company-ats-detector').call({
    companies: ['stripe.com', 'Notion', 'nvidia.com'],
});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.table(items.map(({ input, ats, board_slug, job_count }) => ({ input, ats, board_slug, job_count })));
```

### n8n, Make, Postman and MCP

Ready-made n8n workflows and Make blueprints, a Postman collection and MCP client config are
maintained at [github.com/makenomistakesllc/apify-actors](https://github.com/makenomistakesllc/apify-actors)
(public, MIT). Pull the template that matches your tool and point it at the input schema above.

In n8n and Make, use the generic Apify "Run an Actor" step with actor
`make_no_mistakes/company-ats-detector` and the JSON input above, then map `ats`, `board_slug` and
`api_url` onto your CRM or sheet.

For MCP clients, expose it through the Apify MCP server:

```json
{
  "mcpServers": {
    "apify": {
      "command": "npx",
      "args": ["-y", "@apify/actors-mcp-server", "--actors", "make_no_mistakes/company-ats-detector"],
      "env": { "APIFY_TOKEN": "<YOUR_APIFY_TOKEN>" }
    }
  }
}
```

---

## Pricing

**Free.** There is no pay-per-event pricing attached to this Actor. You pay only for the Apify
platform usage your own run consumes. The measured 20-company run described above took 33 seconds
at 512 MB and used **0.0046 compute units**, about a tenth of a cent of platform usage. There is no
monetization code in it, and nothing here charges you per company or per row.

When you want the postings themselves, that is
[multi-ats-job-board-api](https://apify.com/make_no_mistakes/multi-ats-job-board-api) at $0.004 per
job.

---

## Data source and attribution

Every check reads a public endpoint that the ATS vendor publishes for exactly this purpose, or the
company's own public careers page. No authentication, no proxies, no browser, no HTML scraping of
job content.

**The employer is the system of record.** This Actor reports which platform an employer's board is
on and how many roles are on it. It does not copy, infer or enrich posting content. No personal
data is collected: these are job boards, not candidates.
