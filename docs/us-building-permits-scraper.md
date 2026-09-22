# Building Permits Scraper: A Building Permit Data API for 16 US Cities

> Mirrored from the Actor's own README, build 0.1.18 (Apify Store page is the live source of truth).

This Actor is a building permit data API. It returns newly issued building permits and construction
permits from the official open-data portals of 16 US cities, normalized onto one schema, as JSON or
CSV.

Cities covered: **Austin, San Antonio, New York City (NYC DOB NOW), Los Angeles, Chicago,
Philadelphia, Boston, Seattle, San Francisco, Cincinnati, Baton Rouge, Washington DC, Denver,
Nashville, Raleigh and Louisville.**

It is built for contractors and home-service companies buying construction leads, building-material
and equipment reps working new jobs, real-estate and proptech analysts tracking construction
activity by ZIP, and developers who want a multi-city building permits dataset without writing 16
city-specific scrapers.

Every permit comes back as one flat record: permit number, permit type, permit class, work
description, status, issued date, applied date, address, ZIP, latitude and longitude, valuation,
owner name, contractor name and contractor phone, with the untouched city record attached under
`raw`. Austin and Raleigh are the only two cities that publish contractor phone numbers, so permits
from those two are a dialable contractor lead list on day one. In the other fourteen you get a
contractor name to match against a license board.

**Pricing: $0.008 per permit returned, plus $0.02 to start a run.** You pay only for permits
actually written to the dataset.

No HTML scraping, no proxies, no logins, no CAPTCHAs and no API keys. It queries the Socrata,
ArcGIS REST, CKAN and Carto endpoints the cities publish for exactly this purpose.

---

## Building permit data from 16 US cities on one schema

All 16 feeds below were verified live before shipping and are refreshed by their cities daily.
"Permits/week" is an actual count for the week ending 2026-08-31, **after** the administrative
permits described below are dropped. It will move week to week, but it tells you the order of
magnitude you should expect.

| City | Source | Permits/week | Lag |
|---|---|---|---|
| `austin` (Austin, TX) | [data.austintexas.gov](https://data.austintexas.gov/d/3syk-w9eu) · Issued Construction Permits | ~1,150 | same day |
| `san-antonio` (San Antonio, TX) | [data.sanantonio.gov](https://data.sanantonio.gov/dataset/building-permits) · Permits Issued | ~1,070 | 2–4 days |
| `new-york` (New York, NY) | [data.cityofnewyork.us](https://data.cityofnewyork.us/d/rbx6-tga4) · DOB NOW: Build Approved Permits | ~3,600 | same day |
| `los-angeles` (Los Angeles, CA) | [data.lacity.org](https://data.lacity.org) · LADBS Building + Electrical + Mechanical | ~2,750 | 1–2 days |
| `chicago` (Chicago, IL) | [data.cityofchicago.org](https://data.cityofchicago.org/d/ydr8-5enu) · Building Permits | ~730 | same day |
| `philadelphia` (Philadelphia, PA) | [OpenDataPhilly](https://opendataphilly.org/datasets/licenses-and-inspections-building-permits/) · L&I Building Permits | ~470 | same day |
| `boston` (Boston, MA) | [data.boston.gov](https://data.boston.gov/dataset/approved-building-permits) · Approved Building Permits | ~650 | same day, but pauses (see below) |
| `seattle` (Seattle, WA) | [data.seattle.gov](https://data.seattle.gov/d/76t5-zqzr) · Building Permits | ~80 | 1–3 days |
| `san-francisco` (San Francisco, CA) | [data.sf.gov](https://data.sf.gov/d/i98e-djp9) · Building Permits | ~380 | same day |
| `cincinnati` (Cincinnati, OH) | [data.cincinnati-oh.gov](https://data.cincinnati-oh.gov/d/uhjb-xac9) · Building Permits | ~200 | 1–3 days |
| `baton-rouge` (Baton Rouge, LA) | [data.brla.gov](https://data.brla.gov/d/7fq7-8j7r) · Permits Issued | ~160 | 1–3 days, but pauses (see below) |
| `washington-dc` (Washington, DC) | [opendata.dc.gov](https://opendata.dc.gov/search?q=building%20permits) · DOB Building Permits (FEEDS/DCRA) | ~920 | 3–5 days |
| `denver` (Denver, CO) | [Denver Open Data](https://opendata-geospatialdenver.hub.arcgis.com/search?q=permits) · Commercial + Residential Construction + Demolition | ~120 | 2–3 days |
| `nashville` (Nashville, TN) | [data.nashville.gov](https://data.nashville.gov/datasets/building-permits-issued) · Building Permits Issued | ~205 | 1–2 days |
| `raleigh` (Raleigh, NC) | [data-ral.opendata.arcgis.com](https://data-ral.opendata.arcgis.com/search?q=building%20permits) · Building Permits Issued Past 180 Days | ~100 | 3–5 days |
| `louisville` (Louisville, KY) | [data.louisvilleky.gov](https://data.louisvilleky.gov/search?q=construction%20permits) · Active Construction Permits | ~285 | same day–1 day |

Seattle and Cincinnati are building permits only (no separate trade-permit feed), which is why their
counts look low next to their metro size. Los Angeles is three LADBS datasets merged into one city,
and so is Denver (commercial construction, residential construction, demolition).

**Washington DC publishes one ArcGIS layer per calendar year**, not one table. The Actor discovers
the year layers by name at run time and queries every year your date range touches, so a window that
straddles New Year works without any special handling on your side.

**Raleigh's source layer only holds the trailing 180 days.** Anything issued more than about 6
months ago has dropped out of it, so a `startDate` further back than that legitimately returns
nothing for Raleigh. There is no deeper public layer to fall back to. Every other city goes back
years.

### Administrative permits are dropped by default

Six of these registers are *general* permit registers rather than construction-permit feeds: they
publish non-construction, administrative records in the same table as building work. Those rows have
no job site scope, no valuation and no contractor to sell to, so the Actor excludes them **in the
query sent to the city**, not after the fact. They never consume your `maxItems` budget and you are
never charged for them. Set `includeAdministrative: true` to get the full register instead.

| City | Excluded types | Share of the feed |
|---|---|---|
| San Antonio | Garage Sale, Tree Affidavit Permit, Across the Street Banner, Temporary Weekend Sign, Feather Sign, Avenue Sign, Event Sign, Inflatable Sign | **~10%** (1,376 of 12,585 permits over 60 days) |
| Philadelphia | Operations Permit (temporary tents and canopies, fireworks displays) | ~0.6% |
| Baton Rouge | Tire Business, Short Term Rental, Complaint, Code Violations, Stop Work Order, Donation Box, Sign Permit (Political Campaign), Occupancy Permit (Special Event) | ~2% |
| Washington DC | Home Occupation (registering a business run out of a home) | ~5% (1,683 of 33,824 over 12 months) |
| Nashville | Change Contractor (residential and commercial), which re-papers a permit that already exists | ~1.6% |
| Louisville | Tent, Tax Moratorium | ~0.9% |

Permits that involve actual physical work are deliberately **kept**, even where the name looks
administrative: permanent sign and billboard erection, San Antonio tree-removal permits,
Philadelphia zoning permits and certificates of occupancy, Baton Rouge occupancy and re-roof permits,
Nashville sign and tree-removal permits, DC "Post Card" (over-the-counter minor work) and "Shop
Drawing" permits (every sampled row is elevator installation, alteration or repair), and Louisville
wrecking, fence, retaining-wall, parking-lot and change-of-use permits. Garage sales are the single
biggest offender: San Antonio issues about 3,800 a year.

One other thing to know about San Antonio: its `work_description` is the city's PROJECT NAME field,
which for small permits is often just the address repeated.

**Why other big metros are not here:** Dallas, Houston, Phoenix, Miami, Kansas City MO, Sacramento
and Chattanooga were all tested and rejected. Their open-data permit feeds are either discontinued,
frozen years in the past (Kansas City's building-permit listings stop in 2020), or published only as
one-off year dumps rather than a live feed. Fort Worth's feed is live again but publishes a null work
description, a null street address and a null job value on most rows, and has no issue-date column at
all, only a filing date and a status date, so it is still excluded. Shipping a city that silently
returns zero rows, or rows with nothing in them, is worse than not shipping it.

---

## Contractor leads from permits: phone numbers by city

**Austin and Raleigh are the only two of the 16 cities that publish contractor phone numbers.**
Measured on a live run of all 16 cities (7 days ending 2026-09-21, 100+ rows sampled per city):

| City | Rows with a `contractor_phone` |
|---|---|
| `austin` (Austin, TX) | **98%** |
| `raleigh` (Raleigh, NC) | **78%** |
| all 14 others | **0%**, the source has no phone column at all |

The other fourteen city modules map `contractor_phone` to `null` unconditionally, because there is
no phone field in the feed to map. This is a property of the cities, not a gap in the scraper. For
those cities you get a `contractor_name` to match against a license board or a business directory.

Set **`contractorPhoneOnly: true`** to drop every permit with no phone number, so each row you get
back is a dialable lead. **The filter runs before the record is written to the dataset**, which means
a phone-less permit never consumes your `maxItems` budget and is never charged for. The practical
consequence: with `contractorPhoneOnly: true`, selecting any city other than Austin or Raleigh
returns zero rows today, and costs only the run-start fee.

A permit for a new roof is a buying signal for gutters, solar and siding, and a new-construction
permit is a new construction lead for everyone who sells into a job site. To build a roofing, solar,
HVAC or remodeling lead list, filter to your trade with `permitTypes`, pull the last 7 days, and
work the list:

```json
{
  "cities": ["austin", "raleigh"],
  "daysBack": 7,
  "permitTypes": "roof, solar",
  "contractorPhoneOnly": true,
  "onlyNewSinceLastRun": true,
  "maxItems": 0
}
```

---

## Which permit fields you get

One item per permit. Every item carries all 18 fields; anything the city does not publish is `null`
rather than missing, so the dataset exports cleanly to CSV or a database without ragged columns.

```json
{
  "city": "Austin, TX",
  "permit_number": "1995-014830 BP",
  "permit_type": "Building Permit",
  "permit_class": "Residential",
  "work_description": "Add Bedroom & Level Portion Of Exist Foundation",
  "status": "Active",
  "issued_date": "2026-08-31",
  "applied_date": "1995-05-08",
  "address": "4907 SHOAL CREEK BLVD",
  "zip": "78756",
  "lat": 30.3230315,
  "lng": -97.74450439,
  "contractor_name": "Butlin Homes, Inc.",
  "contractor_phone": "5127732944",
  "valuation": 11000.0,
  "owner_name": null,
  "source_url": "https://abc.austintexas.gov/web/permit/public-search-other?t_detail=1&t_selected_folderrsn=629045",
  "raw": { "permittype": "BP", "work_class": "Addition", "tcad_id": "0227000414", "...": "..." }
}
```

`raw` holds the untouched city record (minus geometry blobs and internal index columns), so nothing
a city publishes is lost even if this Actor's normalized schema does not have a home for it.

### Field coverage by city

Field coverage is a property of the city, not of the scraper. Measured on every permit issued in
the 10 days ending 2026-09-02, with `includeAdministrative` off. `n/a` means the city does not
publish that field at all, so it is `null` on every row.

| City | Contractor | Contractor phone | Valuation | Owner | Applied date | Lat/lng | ZIP |
|---|---|---|---|---|---|---|---|
| Austin | 94% | **92%** | 37% | 17% | 100% | 56% | 100% |
| San Antonio | 98% | n/a | 3% | n/a | 100% | 95% | 96% |
| New York | 100% | n/a | 100% | 100% | 100% | 100% | 100% |
| Los Angeles | n/a | n/a | 100% | n/a | 60% | 99% | 100% |
| Chicago | 98% | n/a | 86% | 98% | 100% | 98% | 97% |
| Philadelphia | 87% | n/a | n/a | 95% | n/a | 100% | 100% |
| Boston | 95% | n/a | 100% | n/a | n/a | 98% | 100% |
| Seattle | n/a | n/a | 100% | n/a | 93% | 100% | 91% |
| San Francisco | n/a | n/a | 100% | n/a | 100% | 100% | 100% |
| Cincinnati | 99% | n/a | 100% | n/a | 100% | 99% | 99% |
| Baton Rouge | 100% | n/a | 100% | 80% | 100% | 100% | 99% |
| Washington DC | 55% | n/a | n/a | 97% | n/a | 100% | 100% |
| Denver | 97% | n/a | 99% | n/a | 100% | 100% | n/a |
| Nashville | 100% | n/a | 100% | n/a | 100% | 100% | 100% |
| Raleigh | 89% | **80%** | 100% | 63% | 100% | 100% | 100% |
| Louisville | 11% | n/a | 100% | n/a | n/a | 99% | 100% |

* **Philadelphia and Boston publish no application date.** Neither the Philadelphia L&I `permits`
  table nor Boston's approved-permits resource has an application or filed-date column, only issued,
  expiration and completion dates. There is nothing to map.
* **Austin's lat/lng gap is Austin's.** 44% of Austin's issued permits ship with no coordinates at
  all: the rows are missing `latitude`, `longitude` *and* the `location` object, so there is no
  other key to fall back to. Every other city geocodes 95–100% of its rows.
* **Valuation is a percentage of rows carrying a number, not a number above zero.** Austin, San
  Antonio and Baton Rouge all publish `0` on a meaningful share of their trade permits. Austin's 37%
  is up from 14%: where the city leaves `total_job_valuation` empty the Actor now sums the per-trade
  component valuations (building, electrical, mechanical, plumbing, medical gas, and their remodel
  variants) instead of reporting nothing.
* Los Angeles and Seattle are the only cities whose applied-date coverage is partial rather than
  all-or-nothing.
* **Washington DC, Louisville and Boston publish no application date** and **DC and Philadelphia
  publish no valuation.** DC's `FEES_PAID` is the permit fee, not a declared job value, so it is
  deliberately not mapped into `valuation`. Reporting a $51 mechanical fee as a job value would be
  worse than a `null`.
* **Denver publishes no ZIP column** and its address string carries no ZIP either, so `zip` is
  `null` on every Denver row. Its `lat`/`lng` come from the layer's point geometry (requested in
  WGS84), not from attribute columns, which is why coverage is 100%.
* **Two cities have a thin `work_description`.** DC fills `DESC_OF_WORK` on 47% of rows (it is
  routinely blank on Post Card and Supplemental trade permits); Louisville publishes no free-text
  scope at all, so `work_description` carries its `WORK_TYPE` ("New Single Family", "Temporary
  Pole") and is populated on 72% of rows. Denver has no free-text scope either: `work_description`
  and `permit_class` both carry its `CLASS` work type ("Alteration/Tenant Finish", "6-Wreck").
* **Louisville's contractor coverage is genuinely low (11%).** The city ships the column blank on
  most rows, mostly on residential electrical permits.
* **`contractor_name` is the permit applicant in DC and Nashville**, which is the licensed
  contractor on contractor-pulled jobs and the homeowner on owner-pulled ones. DC publishes the
  property owner separately, in `owner_name`.

**`permit_number` is not a unique key.** Several cities issue one number per job or project and then
publish one row per trade or address under it: New York (one job, separate Plumbing / Sprinklers /
Standpipe rows), San Antonio (one commercial project, separate electrical / plumbing / mechanical
rows), Cincinnati (one project, separate Building / HVAC / Excavation rows), San Francisco (one
permit, one row per street address it covers) and Washington DC (one job, separate Construction and
Supplemental trade rows). Every such row is a distinct source record and is returned. Deduplicate on
`permit_number` + `permit_type` + `address` if you need one row per trade, or on `permit_number`
alone if you need one row per job.

---

## Input: filter permits by city, date range and trade

```json
{
  "cities": ["austin", "san-antonio"],
  "daysBack": 7,
  "maxItems": 1000,
  "permitTypes": "roof, solar",
  "includeAdministrative": false,
  "contractorPhoneOnly": false,
  "onlyNewSinceLastRun": false,
  "deltaStateName": "us-building-permits-delta"
}
```

| Field | Type | Default | Notes |
|---|---|---|---|
| `cities` | array of enum | `["austin"]` | Any of the 16 keys in the table above. |
| `daysBack` | integer | `7` | Days of issued permits counting back from today (UTC). Max 365. |
| `startDate` | string (ISO) | none | Earliest issued date, `YYYY-MM-DD`. Overrides `daysBack`. |
| `endDate` | string (ISO) | today | Latest issued date, `YYYY-MM-DD`. |
| `maxItems` | integer | `1000` | Cap across **all** selected cities combined, split evenly between them so no single city eats the whole budget. Unused allowance rolls forward. `0` = unlimited. |
| `permitTypes` | string | none | Comma-separated keywords. A permit is kept when any keyword appears in its permit type, permit class or work description (case-insensitive). |
| `includeAdministrative` | boolean | `false` | Include the non-construction records some registers mix in: San Antonio garage sales and tree affidavits, Philadelphia tent/fireworks permits, Baton Rouge business and short-term-rental licences, DC home-occupation registrations, Louisville tents and tax moratoriums, Nashville change-contractor records. Off by default; see the table above. |
| `contractorPhoneOnly` | boolean | `false` | Keep only permits with a non-empty `contractor_phone`. Applied **before** anything is written to the dataset, so filtered-out permits never consume `maxItems` and are never charged for. Only `austin` and `raleigh` publish phone numbers; see above. |
| `onlyNewSinceLastRun` | boolean | `false` | Delta mode: return only permits that have not been returned on a previous run. See [Delta mode](#delta-mode-only-pay-for-permits-you-have-not-seen-yet). |
| `deltaStateName` | string | `us-building-permits-delta` | Name of the named key-value store in **your** account that holds the delta state. Only used when `onlyNewSinceLastRun` is on. Use different names to run independent delta streams side by side. |

The date filter always runs against the **issued** date, not the application date.

---

## Delta mode: only pay for permits you have not seen yet

Set `onlyNewSinceLastRun: true` and every run returns **only the permits it has not already given
you**. Permits you already received are dropped *before* they are written to the dataset, so you
are not charged for them a second time. A scheduled run on a quiet day costs the $0.02 start fee
and nothing else.

```json
{
  "cities": ["austin", "raleigh"],
  "daysBack": 7,
  "maxItems": 0,
  "onlyNewSinceLastRun": true
}
```

**How the state works.** The Actor keeps one small record per city in a **named key-value store in
your own account** (default name `us-building-permits-delta`, override with `deltaStateName`). Named
stores persist across runs, unlike the default run store. Each record holds:

* `last_issued_date`, the newest issued date delivered to you for that city, and
* `fingerprints`, a rolling set of the last 5,000 record fingerprints (a hash of city + permit
  number + permit type + address + issued date), so the same permit is recognised even when the
  city republishes it.

On each run, for each city:

1. The issued-date window you asked for is narrowed to no earlier than `last_issued_date` minus
   **3 days**. The 3-day tail exists because cities routinely back-fill a permit a day or two after
   its issued date; without it, late-arriving rows would be missed forever.
2. Everything inside that window is fetched as normal, and any record whose fingerprint is already
   in the state is dropped before the push. Same-day re-runs therefore return nothing.
3. State is updated **after** the push succeeds, and only for records that actually reached the
   dataset, so a run that dies halfway leaves the rest still eligible next time.

The **first** run with no state behaves exactly like a normal run (it returns the full window) and
then records what it returned. Nothing else about the Actor changes: `permitTypes`,
`contractorPhoneOnly`, `includeAdministrative` and `maxItems` all still apply.

**`maxItems` interacts with delta mode.** The state only knows what was actually delivered. If a run
is capped at 50 items and the window held 300 new permits, the next run returns the next 50; the
250 it never sent are still new to you. For a true "everything new since last time" feed, set
`maxItems: 0` (unlimited) and let `daysBack` bound the work instead.

**Separate streams.** Two workflows that both want their own delta feed must use two different
`deltaStateName` values, otherwise the first one to run consumes the permits for both.

`RUN_SUMMARY` in the run's default key-value store reports what delta mode did:

```json
{
  "delta_mode": true,
  "delta_state_name": "us-building-permits-delta",
  "delta_skipped_count": 96,
  "delta_new_count": 0,
  "delta_state_updated_at": "2026-09-21T17:26:57.054988+00:00",
  "delta_state_error": null
}
```

`delta_skipped_count` is how many already-delivered permits were dropped (and not charged for);
`delta_new_count` is how many were actually returned. If the state store is unreadable the run does
**not** fail. It falls back to a normal full run, sets `delta_mode: false` and puts the reason in
`delta_state_error`.

---

## Use it from Python, curl, n8n, Make or MCP

**Python** (`pip install apify-client`):

```python
from apify_client import ApifyClient

client = ApifyClient("<YOUR_APIFY_TOKEN>")

run = client.actor("make_no_mistakes/us-building-permits-scraper").call(run_input={
    "cities": ["austin", "raleigh"],
    "daysBack": 7,
    "contractorPhoneOnly": True,
    "maxItems": 500,
})

for permit in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(permit["issued_date"], permit["address"],
          permit["contractor_name"], permit["contractor_phone"])
```

**curl**, one call, permits back in the response:

```bash
curl -s -X POST \
  "https://api.apify.com/v2/acts/make_no_mistakes~us-building-permits-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"cities":["austin","raleigh"],"daysBack":7,"maxItems":0,"onlyNewSinceLastRun":true}'
```

```
0 7 * * *  /usr/local/bin/new-permits.sh >> /var/log/new-permits.log 2>&1
```

**n8n**: *Schedule Trigger* (Days: 1) → *Apify* node, Resource `Actor`, Operation
`Run actor and get dataset`, Actor `make_no_mistakes/us-building-permits-scraper`, Custom body:

```json
{ "cities": ["austin", "raleigh"], "daysBack": 7, "maxItems": 0, "onlyNewSinceLastRun": true, "deltaStateName": "n8n-permits-feed" }
```

→ *IF* node on `{{ $json.permit_number }}` being non-empty → your Slack / Sheets / CRM node. On a
day with no new permits the Apify node returns an empty array and the rest of the workflow simply
does not run.

**Make**: *Schedule* → *Apify → Run an Actor* (Wait until finished: Yes) with the same input body
→ *Apify → Get Dataset Items* → *Iterator* → your destination module.

**Apify Schedules**: create a schedule on this Actor with the input above (for example daily at
07:00). Nothing else to configure; the delta state store is created on the first run.

Keep `daysBack` at 7 even on a daily schedule. It costs nothing extra (the 3-day tail plus the
fingerprint set means the overlap is filtered out, not re-charged) and it means a day the schedule
misses, or a city that publishes on a 3–5 day lag, still gets picked up.

### Works in n8n, Make, Postman and MCP

Ready-made n8n workflows and Make blueprints, a Postman collection, and MCP client config are maintained at https://github.com/makenomistakesllc/apify-actors (public, MIT). For this Actor, the templates cover a weekly permits pull into Google Sheets and a new-contractors alert into Slack. Pull the collection or config that matches your tool and point it at the input schema above.

---

## Pricing: $0.008 per permit

Priced per event: a small flat fee when a run starts, plus a per-permit charge for each record
written to the dataset. You pay for permits you actually receive.

| Event | Price | When |
|---|---|---|
| Actor start | $0.02 | once per run, per GB of memory |
| `permit-scraped` | $0.008 | per permit written to the dataset |

Worked examples: a 200-permit pull costs $1.62. A week of Austin (about 1,150 permits) costs
$9.22. A daily delta run on a quiet day costs $0.02. Permits removed by `permitTypes`,
`contractorPhoneOnly`, `includeAdministrative` or delta mode are never written and never billed, so
`maxItems` is a hard spend bound: set it before you call.

---

## FAQ

**Is there an API for building permits?**
Not one national one. Each US city publishes its own permit register on its own platform (Socrata,
ArcGIS, CKAN, Carto) with its own column names, and there is no federal or state-level feed that
aggregates them. This Actor is that missing API for 16 cities: one call, one schema, JSON or CSV.

**How do I get building permit data for multiple cities at once?**
Pass a list of city keys in `cities`. The Actor queries each city's official API independently,
normalizes the results onto the same 18 fields, and returns them in one dataset. `maxItems` is split
evenly between the selected cities so no single large city eats the whole budget.

**Can I get contractor phone numbers from building permits?**
Only in Austin (98% of rows) and Raleigh (78%). Those are the only two of the 16 cities that publish
a contractor phone column at all. Set `contractorPhoneOnly: true` to keep only permits that carry a
dialable number; the filter runs before billing, so you are not charged for the rest.

**Which cities does this building permits scraper cover?**
Austin, San Antonio, New York (DOB NOW), Los Angeles, Chicago, Philadelphia, Boston, Seattle, San
Francisco, Cincinnati, Baton Rouge, Washington DC, Denver, Nashville, Raleigh and Louisville. Dallas,
Houston, Phoenix, Miami, Kansas City MO, Sacramento, Chattanooga and Fort Worth were tested and
rejected because their public feeds are discontinued, frozen or mostly empty.

**How much does building permit data cost here?**
$0.008 per permit returned plus $0.02 to start the run. Nothing is charged for permits filtered out
before the dataset write, and `maxItems` caps the spend on a run.

**Can I use building permits for contractor lead generation?**
Yes, that is the most common use. Filter to your trade with `permitTypes` (roof, solar, HVAC, pool,
remodel), pull the last 7 days, and work the list. Add `contractorPhoneOnly: true` for a dialable
Austin or Raleigh call list, and `onlyNewSinceLastRun: true` so a daily schedule bills you only for
permits you have not already worked.

**Is building permit data public?**
Yes. Building permits are public records, and every source here is an official municipal open-data
portal. Licensing follows the source portal (San Antonio, for example, publishes under CC-BY).
`source_url` on every item points back to the record or dataset it came from.

**How often is the permit data updated?**
Every source refreshes daily. Most publish same day; San Antonio typically lags 2–4 days, Washington
DC and Raleigh 3–5 days, and a few smaller feeds 1–3 days. Municipal portals also stop publishing for
stretches, so use `daysBack: 7` on a daily schedule rather than `daysBack: 1`.

**Can I get only the new permits since my last run?**
Yes. Set `onlyNewSinceLastRun: true`. The Actor keeps per-city delta state in a named key-value store
in your own account and drops already-delivered permits before the dataset write, so a run that finds
nothing new costs only the $0.02 start fee.

---

## For AI agents and MCP clients

Returns issued building permits from 16 US cities on one normalized schema, filtered by issued date and, optionally, by keyword.

Minimal input that returns something useful:

```json
{ "cities": ["austin"], "daysBack": 7, "maxItems": 200 }
```

You get one dataset item per permit, with `city`, `permit_number`, `permit_type`, `permit_class`, `work_description`, `status`, `issued_date`, `applied_date`, `address`, `zip`, `lat`, `lng`, `contractor_name`, `contractor_phone`, `valuation`, `owner_name`, `source_url` and `raw` (the untouched city record).

Cost: $0.02 to start a run, plus $0.008 per permit returned. The call above costs $0.02 if it finds nothing and $1.62 if it hits the 200-permit cap. Set `maxItems` to bound the spend before you call.

Two inputs exist specifically to stop you paying for records you cannot use:

* `"onlyNewSinceLastRun": true` returns, on a schedule, **only permits this account has not
  received before**. Already-delivered permits are dropped before they reach the dataset, so a
  run that finds nothing new costs $0.02. See [Delta mode](#delta-mode-only-pay-for-permits-you-have-not-seen-yet).
* `"contractorPhoneOnly": true` keeps only permits that carry a dialable contractor phone
  number. Applied before anything is written to the dataset, so phone-less permits are never
  charged for. Only `austin` and `raleigh` publish phone numbers (verified above).

Typical questions it answers: which contractors pulled permits in ZIP 78704 last week; new solar permits in Boston this month; how many new-construction permits Chicago issued in the last 30 days; which jobs in San Francisco have the highest declared valuation right now.

---

## What people use this for

**Contractor lead generation.** A permit for a new roof is a buying signal for gutters, solar and
siding. Filter to your trade with `permitTypes`, pull the last 7 days, and work the list. Austin's
and Raleigh's contractor phone numbers make it a dialable call list on day one. Add
`contractorPhoneOnly: true` to drop the rows you cannot call before you are charged for them, and
`onlyNewSinceLastRun: true` so a daily schedule only ever bills you for permits you have not already
worked.

**Building-material and equipment sales.** New-construction and addition permits with a declared
valuation tell a supply rep which jobs are about to need product and roughly how big they are. Sort
`valuation` descending, filter by `zip`, and route by territory.

**Real-estate and investment research.** Permit velocity by ZIP is one of the earliest available
signals of where capital is actually landing, visible months before it shows up in sales comps or
assessor data. `lat`/`lng` are populated well enough in most cities to map directly.

**Market and competitive analysis.** Group by `contractor_name` to see who is winning work by trade,
neighborhood and job size, and how that shifts quarter over quarter.

**Compliance and monitoring.** Watch permits at addresses or ZIPs you care about and get told when
someone pulls one.

---

## How it works

* Queries each city's official API directly: Socrata SoQL (`$where` / `$order` / `$limit` /
  `$offset`, 1,000 rows a page), CKAN `datastore_search_sql`, the Carto SQL API, or the ArcGIS REST
  `/query` endpoint (`where` / `orderByFields` / `resultOffset`, 1,000 features a page).
* Cities are scraped independently inside their own try/except: one city's outage does not take down
  the run, and per-city counts are logged either way.
* Failed requests retry three times with exponential backoff.
* Results are de-duplicated by permit number within each city.
* Every filter (`permitTypes`, `contractorPhoneOnly`, `includeAdministrative` and delta mode)
  is applied **before** `push_data`, so a filtered-out permit is never written to the dataset and
  never billed.
* A `RUN_SUMMARY` record is written to the default key-value store with the resolved date range, the
  item count per city, and the delta-mode counters (`delta_mode`, `delta_state_name`,
  `delta_skipped_count`, `delta_new_count`, `delta_state_updated_at`, `delta_state_error`) plus
  `contractor_phone_only` / `contractor_phone_skipped_count`.
* With `onlyNewSinceLastRun`, delta state is read and written in a named key-value store in the
  caller's own account. Nothing about it is shared between accounts.

Runs comfortably in 256–1024 MB. A single-city week is a few seconds of compute.

### Refresh cadence, and what a zero-row week means

Every source refreshes daily. Most publish same day; San Antonio typically lags 2–4 days, Washington
DC and Raleigh 3–5 days, and a few of the smaller feeds lag 1–3 days, so a `daysBack` of 2 can
legitimately return zero rows for those cities. `daysBack: 7` on a daily schedule is the reliable
pattern.

**Municipal portals also stop publishing for stretches at a time, and a zero-row week is usually
that, not a broken feed.** Two observed cases, both confirmed against the source rather than
inferred: on 2026-09-14 Boston's newest `issued_date` was 2026-09-03 (11 days, after publishing
100–185 permits a day right up to 2026-09-02) and Baton Rouge's newest `issueddate` was 2026-09-04
(10 days). Both endpoints answered `HTTP 200` with the documented columns the whole time. The
datasets were still being rewritten daily, just with no newer permits in them. Boston's stated
"same day" and Baton Rouge's "1–3 days" describe the normal cadence, not a floor.

If a city returns zero rows, check the source before assuming the Actor is at fault. Every city
module carries a `PROBE_URL` constant that returns `max(<date field>)` for exactly this:

```
https://data.boston.gov/api/3/action/datastore_search_sql?sql=SELECT max("issued_date") AS max_date FROM "6ddcd912-32a0-43df-9908-63574f8c7e77"
https://data.brla.gov/resource/7fq7-8j7r.json?$select=max(issueddate) as max_date
```

For the ArcGIS cities (DC, Denver, Nashville, Raleigh, Louisville) the same probe is a statistics
query on the layer, and the answer comes back as epoch milliseconds:

```
https://services1.arcgis.com/79kfd2K6fskCAkyg/arcgis/rest/services/active_construction_permits/FeatureServer/0/query?where=1=1&returnGeometry=false&f=json&outStatistics=[{"statisticType":"max","onStatisticField":"ISSUE_DATE","outStatisticFieldName":"max_date"}]
```

Widen `daysBack` to 14–30 to ride out a pause.

---

## Data source and attribution

All data comes from public municipal open-data portals published by the cities themselves. This
Actor only reads documented public API endpoints: no HTML scraping, no authentication, no proxies.

Permit records are public records. Licensing follows the source portal (San Antonio, for example,
publishes under CC-BY). Attribute the originating city when you redistribute, and check the source
portal's terms for your use case. `source_url` on every item points back to the record or dataset it
came from.

The cities are the system of record. If a permit looks wrong, it looks that way in the city's feed
too. This Actor does not correct, infer or enrich, it normalizes.
