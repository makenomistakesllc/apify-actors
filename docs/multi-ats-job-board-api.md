# Multi-ATS Job Board API: Scrape Greenhouse, Lever, Ashby and Workday Jobs

> Mirrored from the Actor's own README, build 0.1.8 (Apify Store page is the live source of truth).

This Actor is a job posting scraper and job board API for nine applicant tracking systems:
**Greenhouse, Ashby, Lever, Workday, SmartRecruiters, Workable, Recruitee, Personio and Rippling.**
It returns every open job from a company's careers page feed, normalized onto one schema, as JSON
or CSV.

**Give it company names or domains. It finds the ATS.** Every other job-board Actor makes you supply
the board token, so you have to already know that Stripe is on Greenhouse under `stripe`, OpenAI is
on Ashby under `openai`, and NVIDIA is on Workday under
`nvidia.wd5.myworkdayjobs.com/NVIDIAExternalCareerSite`. That lookup is the actual work. This Actor
does it, resolving 18 of 20 companies (90%) on a mixed test set. Board URLs and board slugs work too.

It is built for recruiters and sourcers watching competitor hiring, job-board and aggregator
operators, sales and market-intelligence teams that treat open roles as a buying signal, and
developers who want one job postings API instead of nine vendor integrations.

Each job comes back as one flat record: company, ATS, job ID, title, department, team, city, state,
country, remote flag, employment type, salary range, posted and updated dates, full description and
the real apply URL, with the untouched platform record under `raw`.

**Pricing: $0.004 per job returned, plus $0.005 to start a run.** `discoverOnly` mode answers "which
ATS is this company on?" for $0.001 per company, without paying for jobs.

No HTML scraping, no proxies, no logins, no CAPTCHAs and no API keys. It calls the public job-board
APIs the ATS vendors published for aggregators to consume.

---

## Job postings from 9 ATS platforms on one schema

All nine were verified live before shipping: the Greenhouse job board API, the Lever postings API,
the Ashby job postings API, the SmartRecruiters jobs API, the Recruitee, Rippling, Personio and
Workable feeds, and Workday's per-tenant careers endpoint. "Salary" is whether the platform
publishes a structured pay range at all, not whether a given employer filled it in.

| Platform | Endpoint | Salary | Department / team | Dates |
|---|---|---|---|---|
| `greenhouse` | `boards-api.greenhouse.io/v1/boards/{token}/jobs` | ✅ pay ranges when the employer opts into transparency | department only | posted **and** updated |
| `ashby` | `api.ashbyhq.com/posting-api/job-board/{org}` | ✅ full structured comp: min, max, currency, interval | department **and** team | posted |
| `lever` | `api.lever.co/v0/postings/{org}` | ✅ `salaryRange` when filled in | department **and** team | posted |
| `smartrecruiters` | `api.smartrecruiters.com/v1/companies/{co}/postings` | ❌ | department + function | posted **and** updated |
| `recruitee` | `{co}.recruitee.com/api/offers/` | ⚠️ structured but usually empty | department | posted **and** updated |
| `rippling` | `api.rippling.com/platform/api/ats/v1/board/{co}/jobs` | ⚠️ `payRangeDetails`, rarely populated | department **and** sub-team | posted |
| `personio` | `{co}.jobs.personio.de/xml` | ❌ | department + recruiting category | posted |
| `workday` | `{tenant}.wd{N}.myworkdayjobs.com/wday/cxs/{tenant}/{site}/jobs` | ❌ | ❌ neither | posted |
| `workable` | `apply.workable.com/api/v1/widget/accounts/{co}` | ❌ | department + function | posted |

### Measured field coverage

Percentage of returned jobs with a non-null value, on a live sample of 40 jobs per board,
2026-09-02. `n/a` means the platform does not publish the field at all.

| Board | Dept | Team | City | Country | Remote flag | Type | Salary | Posted | Updated | Description |
|---|---|---|---|---|---|---|---|---|---|---|
| greenhouse / `gitlab` | 100% | n/a | 55% | 80% | ✅ | n/a | **22%** | 100% | 100% | 100% |
| greenhouse / `stripe` | 100% | n/a | 80% | 45% | ✅ | n/a | 0% | 100% | 100% | 100% |
| ashby / `ramp` | 100% | 100% | 95% | 100% | ✅ | 100% | **100%** | 100% | n/a | 100% |
| ashby / `openai` | 100% | 100% | 92% | 100% | ✅ | 100% | **90%** | 100% | n/a | 100% |
| lever / `matchgroup` | 100% | 100% | 98% | 100% | ✅ | 95% | **50%** | 100% | n/a | 100% |
| lever / `palantir` | n/a | 100% | 100% | 100% | ✅ | 100% | 0% | 100% | n/a | 100% |
| smartrecruiters / `Ubisoft2` | 100% | 95% | 100% | 100% | ✅ | 100% | n/a | 100% | n/a | 100% |
| recruitee / `vandebron` | 100% | 100% | 100% | 100% | ✅ | 100% | 23% | 100% | 100% | 100% |
| rippling / `rippling` | 100% | 100% | 95% | 98% | ✅ | 100% | 0% | 100% | n/a | 100% |
| personio / `wandelbots` | 100% | 100% | 67% | n/a | ✅ | 100% | n/a | 100% | n/a | 0% |
| workday / `nvidia` | n/a | n/a | 100% | 100% | ✅ | 100% | n/a | 100% | n/a | 100% |
| workday / `salesforce` | n/a | n/a | 100% | 100% | ✅ | 100% | n/a | 100% | n/a | 100% |
| workable / `adverity` | 100% | 64% | 86% | 100% | ✅ | 86% | n/a | 100% | n/a | 100% |

The honest summary: **if you want salary data, Ashby is the platform that has it.** Ramp publishes
a range on 100% of its postings and OpenAI on 90%. Greenhouse has ranges only where the employer
turned on pay transparency (GitLab 22%, Stripe 0%). Lever is a coin flip. The other six publish no
usable pay data at all, and this Actor returns `null` rather than guessing a number out of the
description text.

---

## Find which ATS a company uses: automatic board discovery

Discovery is the point of this Actor, so here is exactly what it does and how well it works.

Each input is reduced to a short list of plausible board slugs: the domain label, the company name
lowercased and de-spaced, the hyphenated variant, the variant with `Inc` / `GmbH` / `Group` and
friends removed. Those are probed across the platforms in three tiers, cheapest first, stopping at
the first tier that finds a board of meaningful size:

1. Greenhouse, Ashby, Lever, SmartRecruiters, Recruitee, Rippling: one cheap GET each, all
   candidates in parallel.
2. Personio and Workable: both rate-limit by IP, so they are paced and only reached when tier 1
   comes up empty (or turns up only a handful of jobs).
3. Workday: needs per-tenant host and site discovery, so it goes last.

A board URL short-circuits the whole thing: paste `https://jobs.lever.co/matchgroup` or
`https://nvidia.wd5.myworkdayjobs.com/NVIDIAExternalCareerSite` and the platform and slug are read
straight off it.

Every resolution is cached in the run's key-value store under `ATS_DISCOVERY_CACHE`, keyed by the
raw input string, so an unchanged watchlist does not re-probe.

**Measured hit rate: 18 of 20 (90%)** on a mixed test set: Stripe, OpenAI, Anthropic, Databricks,
Ramp, Notion, Linear, Datadog, NVIDIA, Vercel, Figma, Airtable, Brex, Cloudflare, Match Group,
Vandebron, Adverity, scale.com. **Misses: DoorDash and Retool**, both of which run career sites that
are not on any of the nine public job-board APIs, so there is nothing to find.

To ask only "which ATS is this company on?" and skip the jobs, use `discoverOnly`, charged at $0.001
per company:

```json
{ "companies": ["stripe.com", "notion.so", "doordash.com"], "discoverOnly": true }
```

You get one row per company: `company_input`, `company_slug`, `ats`, `board_url`, `job_count`,
`resolved`, `other_matches`, `candidates_tried`.

### Where discovery gets it wrong

* **A slug can belong to someone else.** `scale.com` resolves to a six-job Personio board owned by a
  different company called Scale; Scale AI's real board is on Greenhouse under `scaleai`. When the
  slug guess is ambiguous there is no signal that distinguishes them. If the resolved `job_count`
  looks absurd for the employer, pass the board URL instead.
* **Some companies are on two platforms at once.** Mid-migration boards are common. The larger board
  wins, the other is listed in `other_matches` in `discoverOnly` mode, and jobs that appear on both
  are de-duplicated by title and location.
* **A rate-limited probe reads as a miss.** Personio and Workable both answer HTTP 429 when several
  requests land close together. The Actor paces and retries them, but a company whose only board is
  on one of those two can occasionally come back unresolved. Re-run, or pass the board URL.
* **`resolved: false` is the honest answer**, not an error. `candidates_tried` tells you which slugs
  were probed so you can supply the right one.

### Just want to know which ATS a company uses?

[**Company ATS Detector**](https://apify.com/make_no_mistakes/company-ats-detector) is the free
Actor that answers only that question: one row per company with the platform, the board URL, the
public no-auth API endpoint and the open-job count. It runs the same discovery code as this Actor.

**Kept in sync by hand.** Apify builds each Actor from its own directory, so the two cannot share a
Python package: `src/discovery.py`, `src/http.py`, `src/normalize.py` and `src/platforms/` are
copied into `company-ats-detector/src/` with a header naming the origin file. This Actor is the
origin. After changing any of those files here, re-copy them with
`python3 ../company-ats-detector/ops/sync_vendored.py --write`; the same script without `--write`
fails loudly when the two have drifted.

---

## Which job fields you get

One item per job. Every item carries all 24 fields; anything the platform does not publish is `null`
rather than missing, so the dataset exports cleanly to CSV or a database without ragged columns.

```json
{
  "company": "Stripe",
  "company_slug": "stripe",
  "ats": "greenhouse",
  "job_id": "8044460",
  "title": "AI Engineer",
  "department": "1150 Solutions Architecture",
  "team": null,
  "location_raw": "Chicago",
  "city": "Chicago",
  "state": null,
  "country": null,
  "is_remote": false,
  "employment_type": null,
  "salary_min": null,
  "salary_max": null,
  "salary_currency": null,
  "salary_period": null,
  "posted_at": "2026-07-03",
  "updated_at": "2026-08-26",
  "apply_url": "https://stripe.com/jobs/search?gh_jid=8044460",
  "description_html": "<h2><strong>Who We Are</strong></h2>…",
  "description_text": "Who We Are\n\nAbout Stripe\n\nStripe is a financial infrastructure platform…",
  "source_url": "https://boards-api.greenhouse.io/v1/boards/stripe/jobs",
  "raw": { "internal_job_id": 3486653, "requisition_id": "See Opening ID", "offices": [], "…": "…" }
}
```

A record from Ashby, where the compensation block is real:

```json
{
  "company": "Ramp", "ats": "ashby", "title": "Security Engineer, Cloud",
  "department": "Engineering", "team": "Backend",
  "location_raw": "New York, NY (HQ) | Remote (Canada) | Remote (US) | Miami, FL",
  "city": "New York City", "state": "NY", "country": "US", "is_remote": true,
  "employment_type": "FULL_TIME",
  "salary_min": 211400, "salary_max": 290600, "salary_currency": "USD", "salary_period": "YEAR",
  "posted_at": "2026-04-07",
  "apply_url": "https://jobs.ashbyhq.com/ramp/34413f8d-26bf-4bbc-8ade-eb309a0e2245/application"
}
```

`raw` holds the untouched platform record (minus the description blobs, which are already broken
out above), so nothing an ATS publishes is lost even where this Actor's normalized schema has no
home for it.

### Normalization rules worth knowing

* **`employment_type`** is mapped onto `FULL_TIME`, `PART_TIME`, `CONTRACT`, `TEMPORARY`,
  `INTERNSHIP`, `VOLUNTEER`. A value nobody recognises passes through upper-cased rather than being
  dropped.
* **`salary_period`** is `YEAR`, `MONTH`, `WEEK`, `DAY` or `HOUR`.
* **Salary comes only from structured compensation fields.** Descriptions are never regex-mined for
  pay. A guessed number in a `salary_min` column is worse than an honest `null`.
* **`is_remote`** is true when the platform sets a remote flag, or when `location_raw` contains
  remote / anywhere / distributed / work-from-home. Job titles and descriptions are never consulted,
  because they say "remote" for reasons that have nothing to do with the role.
* **`city` / `state` / `country`** come from the platform's structured address where it publishes
  one (Ashby, SmartRecruiters, Recruitee, Workable) and from parsing `location_raw` otherwise. ATS
  location strings are unconstrained free text, so treat the parse as best-effort and
  `location_raw` as the truth.
* **Multiple locations** are joined with ` | ` in `location_raw`; `city`/`state`/`country` describe
  the primary one.
* **Dates are dates**, `YYYY-MM-DD`, in every field on every platform.
* **`company` is the employer's own name where the platform publishes one.** Greenhouse,
  SmartRecruiters, Recruitee, Rippling, Personio and Workable all do. Ashby, Lever and Workday do
  not, so `company` there is derived from what you passed in: `"Match Group"` stays `"Match Group"`,
  but a bare slug like `openai` becomes `"Openai"`. Pass the name you want to see, or read
  `company_slug`, which is always exact.

---

## Input: filter jobs by title, location, remote and posted date

```json
{
  "companies": ["stripe", "openai.com", "Match Group", "https://jobs.ashbyhq.com/ramp"],
  "maxItems": 500,
  "maxItemsPerCompany": 25,
  "postedWithinDays": 7,
  "titleIncludes": ["engineer", "designer"],
  "location": "United States",
  "remoteOnly": false,
  "includeDescription": true
}
```

| Field | Type | Default | Notes |
|---|---|---|---|
| `companies` | array of string | `["stripe","openai","ramp"]` | Names, domains, board slugs or board URLs. Mix them freely. |
| `discoverOnly` | boolean | `false` | Return one row per company (which ATS, which board, how many jobs) and stop. Charged as `company-resolved`, not `job-scraped`. |
| `platforms` | array of enum | all nine | Restrict discovery and fetching to a subset. |
| `maxItems` | integer | `500` | Hard cap across **all** companies combined: a global ceiling filled in company order, so the cap is always spent in full before any board is dropped. `0` = unlimited. |
| `maxItemsPerCompany` | integer | none | Cap on the jobs returned for **each company individually**, applied while that company's board is fetched. Every company is still fetched. `0` or empty = no per-company cap. Independent of `maxItems`. |
| `postedWithinDays` | integer | none | Incremental mode. Keeps a job when the newer of `posted_at` / `updated_at` falls inside the window. |
| `titleIncludes` | array of string | none | OR'd, case-insensitive, matched against the job title. |
| `location` | string | none | Substring match against `location_raw`, `city`, `state` and `country`, case-insensitive. |
| `remoteOnly` | boolean | `false` | Keep only jobs the platform flags remote, or whose location says remote / anywhere / distributed. |
| `includeDescription` | boolean | `true` | Off omits `description_html` and `description_text` entirely, and items get roughly 10x smaller. |

`maxItems` is your spend bound. Set it deliberately: 20 well-known tech companies is about
5,000 open jobs, and a single Workday tenant can be 2,000 on its own.

It is filled **in company order**, so companies earlier in your `companies` array are fetched to
completion first. It is not a per-company quota: with `maxItems: 100` over 40 companies you get
100 jobs from the first company or two, not 2 or 3 from each. Put the companies you care about most
first, or run them in separate runs, if you want an even spread.

### `maxItems` vs `maxItemsPerCompany`

They are two different caps and they do not replace each other. **`maxItems` is a budget ceiling**:
one global number for the whole run, filled in company order, so the first companies can consume all
of it and the companies after the cut are never fetched (the run says so, see *A stopped run says
so, loudly* below). **`maxItemsPerCompany` is a spread control**: it caps each company separately
while its board is being fetched, so every company in the list is still reached and no single large
employer can eat the run.

Use `maxItemsPerCompany` for watchlists, health checks and monitoring, anything where you need a
sample from *every* company (`maxItemsPerCompany: 5` over 50 companies gives you at most 5 from each
of the 50). Use `maxItems` when what you actually need is a hard ceiling on spend. Set both when you
want both: each company stops at whichever cap bites first, and the run still stops at `maxItems`.

### How many companies per run

**Up to about 40 resolved companies per run, and keep the expected job count under this run's charge
ceiling.** Every pay-per-event run carries a `maxTotalChargeUsd`, which Apify sets from your
remaining credit unless you set it yourself, and at $0.004 per job that ceiling converts directly
into a maximum number of items. A $5 ceiling is about 1,240 jobs. When a run reaches it, the
platform stops accepting items, and this Actor stops rather than fetching boards it can no longer
deliver.

**A stopped run says so, loudly.** Companies it never reached are:

* listed in `RUN_SUMMARY.companies_not_fetched`,
* given one entry each in `RUN_SUMMARY.errors` (`"not fetched - the run stopped before reaching
  this company: …"`),
* logged at ERROR level as `RUN TRUNCATED: …`, and
* **left out of `items_per_company` entirely**, never written there as `0`, because `0` means "this
  board is empty" and that is a different fact.

`RUN_SUMMARY` also carries `truncated`, `stopped_reason`, `items_dropped_at_charge_limit`,
`max_chargeable_items` and `max_total_charge_usd`, and the run warns up front when discovery has
already found more open jobs than the run can be charged for. If you see any of that, split the
companies across more runs or raise the run's charge limit. The jobs are there, the run simply ran
out of ceiling.

`scripts/check_batch.py` is the regression check for this: it runs the same 40 boards once as a
single run and once as two 20-company runs and fails if any company comes back short without being
declared missing.

---

## Pricing: $0.004 per job

Pay-per-event. You pay for what the run actually returns.

| Event | Price | When |
|---|---|---|
| Actor start | $0.005 | once per run |
| `job-scraped` | $0.004 | per job written to the dataset |
| `company-resolved` | $0.001 | per company in `discoverOnly` mode |

`job-scraped` and `company-resolved` are never both charged in the same run. Jobs filtered out by
`postedWithinDays`, `location`, `titleIncludes` or `remoteOnly` are never written and never billed.

Worked examples: resolving a 200-company watchlist with `discoverOnly` costs **$0.205**. Pulling
500 jobs costs **$2.005**. A daily `postedWithinDays: 1` sweep over 50 companies typically returns
a few dozen jobs, so it runs at cents a day rather than re-paying for the whole board, which is the
point of incremental mode.

---

## Use it from Python, curl, n8n, Make or MCP

**Python** (`pip install apify-client`):

```python
from apify_client import ApifyClient

client = ApifyClient("<YOUR_APIFY_TOKEN>")

run = client.actor("make_no_mistakes/multi-ats-job-board-api").call(run_input={
    "companies": ["stripe", "openai.com", "Match Group"],
    "titleIncludes": ["engineer"],
    "postedWithinDays": 7,
    "maxItems": 200,
})

for job in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(job["company"], job["ats"], job["title"], job["location_raw"], job["apply_url"])
```

**curl**, one call, jobs back in the response:

```bash
curl -s -X POST \
  "https://api.apify.com/v2/acts/make_no_mistakes~multi-ats-job-board-api/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"companies":["stripe","openai","ramp"],"postedWithinDays":7,"maxItems":200}'
```

**n8n**: *Schedule Trigger* (Days: 1) → *Apify* node, Resource `Actor`, Operation
`Run actor and get dataset`, Actor `make_no_mistakes/multi-ats-job-board-api`, with the same JSON
body → *IF* node on `{{ $json.job_id }}` being non-empty → your Slack / Airtable / CRM node.

**Make**: *Schedule* → *Apify → Run an Actor* (Wait until finished: Yes) with the same input body
→ *Apify → Get Dataset Items* → *Iterator* → your destination module.

**Apify Schedules**: create a schedule on this Actor with `postedWithinDays: 1` for a daily
new-postings sweep over a watchlist.

### Works in n8n, Make, Postman and MCP

Ready-made n8n workflows and Make blueprints, a Postman collection, and MCP client config are maintained at https://github.com/makenomistakesllc/apify-actors (public, MIT). For this Actor, the templates cover a daily watchlist pull into Airtable and a title-match alert by email. Pull the collection or config that matches your tool and point it at the input schema above.

---

## FAQ

**Does Greenhouse have a public API for job postings?**
Yes. The Greenhouse job board API lives at `boards-api.greenhouse.io/v1/boards/{token}/jobs`, no
key required, but you need the employer's board token first. This Actor resolves the token from a
company name or domain, then reads that endpoint and normalizes the result.

**Is there a Lever jobs API?**
Yes, `api.lever.co/v0/postings/{org}`, also keyless and also keyed on a slug you have to know. Same
answer as Greenhouse: pass the company name or `jobs.lever.co` URL and this Actor resolves it.

**How do I scrape jobs from multiple ATS platforms at once?**
Pass a mixed list of companies in one run. The Actor detects each company's ATS independently across
all nine vendors and returns every job on the same 24-field schema, so a Greenhouse job and a Workday
job come back with the same column names.

**Can I find out which ATS a company uses?**
Yes, that is `discoverOnly: true`, charged at $0.001 per company. You get the platform, the board
slug, the board URL and the current open-job count, plus `other_matches` when a company is live on
two platforms.

**How do I scrape a company's careers page?**
Most company careers pages are a thin front end over one of these nine ATS job-board APIs, so this
Actor reads the API rather than the page: no browser, no HTML parsing, no proxies. Companies that run
a custom careers site with no public ATS feed (DoorDash and Retool, for example) cannot be scraped
this way, and the Actor reports them as `resolved: false` rather than guessing.

**Does Ashby publish salary data?**
Ashby is the best of the nine for pay data: full structured min, max, currency and interval. Ramp
fills it on 100% of postings, OpenAI on 90%. Greenhouse has ranges only where the employer turned on
pay transparency, Lever is roughly a coin flip, and the other six publish nothing usable.

**Can I scrape Workday job postings?**
Yes. As a Workday jobs scraper this Actor works, but Workday is the fragile one of the nine. It has no documented public job-board API and no tenant
directory, so the host (`wd1` … `wd12`) and the site name both have to be discovered. That works
today and is not contract. Workday also publishes no department, no team and no salary, so those
columns are always `null` for Workday jobs.

**Do I need an API key or a board token?**
No. All nine endpoints are public and keyless, and the board slug is discovered for you. You only
need your Apify token to run the Actor.

**How much does it cost to scrape job postings?**
$0.004 per job written to the dataset, plus $0.005 to start the run. 500 jobs is $2.005. A
200-company `discoverOnly` sweep is $0.205. Filtered-out jobs are never billed.

**Can I monitor job boards for new postings daily?**
Yes. Set `postedWithinDays: 1` and schedule the run. Only postings that appeared or changed inside
the window are written, so a daily watchlist sweep costs cents rather than re-paying for the whole
board each morning.

---

## For AI agents and MCP clients

Resolves company names or domains to their applicant tracking system and returns every open job on
one normalized schema across nine ATS platforms.

Minimal input that returns something useful:

```json
{ "companies": ["stripe", "openai", "ramp"], "maxItems": 200 }
```

Cheap mode, which ATS does each company use, without paying for jobs:

```json
{ "companies": ["stripe.com", "notion.so", "doordash.com"], "discoverOnly": true }
```

You get one dataset item per job, with `company`, `company_slug`, `ats`, `job_id`, `title`,
`department`, `team`, `location_raw`, `city`, `state`, `country`, `is_remote`, `employment_type`,
`salary_min`, `salary_max`, `salary_currency`, `salary_period`, `posted_at`, `updated_at`,
`apply_url`, `description_html`, `description_text`, `source_url` and `raw` (the untouched platform
record). In `discoverOnly` mode you get one item per company instead: `company_input`,
`company_slug`, `ats`, `board_url`, `job_count`, `resolved`, `other_matches`, `candidates_tried`.

Cost: $0.005 to start a run, plus $0.004 per job returned, or $0.001 per company in `discoverOnly`
mode. The first call above costs $0.005 if it finds nothing and $0.805 if it hits the 200-job cap.
**Set `maxItems` to bound the spend before you call.** A single large employer can have 2,000 open
jobs.

Typical questions it answers: which ATS does this company use; every open engineering role at these
40 companies; what changed on these boards in the last 7 days (`postedWithinDays: 7`); which of
these companies publish salary ranges and what they are; remote-only roles across a competitor set.

---

## Honest limits

**Workday is the fragile one.** It is the only platform of the nine with no documented public
job-board API and no directory of tenants. Every employer lives on its own host (`wd1` … `wd12`)
under a site name it chose (`External`, `NVIDIAExternalCareerSite`, `Salesforce_Careers`, and so
on), and both have to be discovered. This Actor reads the real site name off each host's
`robots.txt`, which is reliable today, and falls back to probing common site names. **None of that
is contract.** If Workday changes its behaviour, Workday companies start coming back unresolved. The
other eight platforms are unaffected, and the run does not fail. Workday also publishes no
department, no team and no salary anywhere in its public feed, so those columns are always `null`
for Workday jobs.

**Salary coverage is thin outside Ashby.** SmartRecruiters, Workable, Personio and Workday publish
no compensation on their public endpoints at all. Recruitee and Rippling have the fields but
employers almost never fill them in. See the coverage table above before you build anything that
assumes a pay range.

**Personio and Workable rate-limit by IP.** Both answer HTTP 429 when requests arrive close
together. Requests to them are paced and retried, but a large watchlist that leans heavily on those
two can see individual companies come back unresolved. Re-run, or pass board URLs.

**Discovery misses employers that are not on these nine.** DoorDash and Retool both run career sites
with no public ATS job-board API. Neither is a bug; there is nothing to fetch.

**Personio's XML feed usually omits the description.** The `<jobDescriptions>` element is empty on
most boards, so `description_html` and `description_text` are frequently `null` there.

**Rippling repeats a job once per work location.** The board endpoint returns the same `uuid`
several times; the Actor merges the locations and de-duplicates on `(ats, job_id)`, so 16 board rows
can legitimately become 7 jobs.

**`updated_at` exists on three platforms only**: Greenhouse, SmartRecruiters and Recruitee.
Everywhere else it is `null` and `postedWithinDays` filters on `posted_at` alone.

**Job counts move.** Boards change hour to hour. A `job_count` from `discoverOnly` is a snapshot,
not a promise about the next run.

**A run cannot exceed its own charge ceiling.** `maxTotalChargeUsd` is set per run by the platform
from your remaining credit (unless you set it yourself) and caps how many `job-scraped` events the
run can charge for. Pack too many companies into one run and it will stop at that ceiling. It will
tell you exactly which companies it did not reach, see *How many companies per run* above, but it
cannot fetch them for you. Split the input, or raise the limit.

---

## How it works

* Calls each vendor's public job-board API directly: nine adapters, one per platform, in
  `src/platforms/`.
* A company that fails, or that the run never reaches, is reported in `RUN_SUMMARY.errors` and in
  the run log. It is never handed back as a silent `0`.
* Every company and every platform is fetched inside its own try/except: one dead source does not
  take down the run, and per-platform counts and an `errors` list are written either way.
* Failed requests retry with exponential backoff and honour `Retry-After`. A 404 board slug is
  treated as "not on this platform", not as an error.
* Jobs are de-duplicated on `(ats, job_id)`. When a company is live on two platforms, postings that
  match on title and location are dropped from the smaller board.
* A `RUN_SUMMARY` record is written to the default key-value store with the resolved filters, item
  counts per platform and per company, unresolved companies, and any per-source errors.
* Runs in 256–1024 MB. A 20-company sweep finishes in well under a minute.

---

## Data source and attribution

All data comes from job-board APIs the ATS vendors publish for exactly this purpose. Greenhouse's is
documented at `developers.greenhouse.io`, Ashby's and Lever's likewise. This Actor reads public
endpoints only: no authentication, no proxies, no HTML scraping, no browser.

**The employer is the system of record.** Job postings belong to the companies that published them,
and their content is theirs. This Actor does not correct, infer or enrich, it normalizes. If a
posting looks wrong, it looks that way on the employer's own careers page too. `source_url` and
`apply_url` on every item point back to where it came from; check the employer's and the ATS
vendor's terms for your use case before redistributing.

No personal data is collected. These are job postings, not candidates.
