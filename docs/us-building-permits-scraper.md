# Building Permits Scraper - Construction & Contractor Leads

**Does it cover your city?** Austin, San Antonio, New York (DOB NOW), Los Angeles, Chicago,
Philadelphia, Boston, Seattle, San Francisco, Cincinnati and Baton Rouge — 11 US cities, one call,
one schema.

Scrape newly issued **building permits and construction permits** straight from the cities' official
open-data APIs (Socrata, CKAN, Carto). Every construction job in America starts with a permit, and
every city publishes a different schema on a different platform. This Actor normalizes all 11 onto a
single flat record — permit number, permit type, work description, status, issued date, address,
ZIP, lat/lng, valuation, owner name, contractor name and contractor phone — with the original city
record attached under `raw`.

**Contractor and construction leads:** filter to your trade with `permitTypes` (roof, solar, HVAC,
pool, remodel), pull the last 7 days, and work the list. Austin is the one city that publishes
contractor phone numbers, so its rows are a dialable call list on day one; elsewhere you get a
contractor name to match against a license board.

No HTML scraping, no proxies, no logins, no CAPTCHAs. Just the public APIs the cities built for
this purpose.

---

## For AI agents and MCP clients

Returns issued building permits from 11 US cities on one normalized schema, filtered by issued date and, optionally, by keyword.

Minimal input that returns something useful:

```json
{ "cities": ["austin"], "daysBack": 7, "maxItems": 200 }
```

You get one dataset item per permit, with `city`, `permit_number`, `permit_type`, `permit_class`, `work_description`, `status`, `issued_date`, `applied_date`, `address`, `zip`, `lat`, `lng`, `contractor_name`, `contractor_phone`, `valuation`, `owner_name`, `source_url` and `raw` (the untouched city record).

Cost: $0.02 to start a run, plus $0.005 per permit returned. The call above costs $0.02 if it finds nothing and $1.02 if it hits the 200-permit cap — set `maxItems` to bound the spend before you call.

Typical questions it answers: which contractors pulled permits in ZIP 78704 last week; new solar permits in Boston this month; how many new-construction permits Chicago issued in the last 30 days; which jobs in San Francisco have the highest declared valuation right now.

## Supported cities

All 11 feeds below were verified live before shipping and are refreshed by their cities daily.
"Permits/week" is an actual count for the week ending 2026-08-31, **after** the administrative
permits described below are dropped — it will move week to week, but it tells you the order of
magnitude you should expect.

| City | Source | Permits/week | Lag |
|---|---|---|---|
| `austin` — Austin, TX | [data.austintexas.gov](https://data.austintexas.gov/d/3syk-w9eu) · Issued Construction Permits | ~1,150 | same day |
| `san-antonio` — San Antonio, TX | [data.sanantonio.gov](https://data.sanantonio.gov/dataset/building-permits) · Permits Issued | ~1,070 | 2–4 days |
| `new-york` — New York, NY | [data.cityofnewyork.us](https://data.cityofnewyork.us/d/rbx6-tga4) · DOB NOW: Build Approved Permits | ~3,600 | same day |
| `los-angeles` — Los Angeles, CA | [data.lacity.org](https://data.lacity.org) · LADBS Building + Electrical + Mechanical | ~2,750 | 1–2 days |
| `chicago` — Chicago, IL | [data.cityofchicago.org](https://data.cityofchicago.org/d/ydr8-5enu) · Building Permits | ~730 | same day |
| `philadelphia` — Philadelphia, PA | [OpenDataPhilly](https://opendataphilly.org/datasets/licenses-and-inspections-building-permits/) · L&I Building Permits | ~470 | same day |
| `boston` — Boston, MA | [data.boston.gov](https://data.boston.gov/dataset/approved-building-permits) · Approved Building Permits | ~650 | same day |
| `seattle` — Seattle, WA | [data.seattle.gov](https://data.seattle.gov/d/76t5-zqzr) · Building Permits | ~80 | 1–3 days |
| `san-francisco` — San Francisco, CA | [data.sfgov.org](https://data.sfgov.org/d/i98e-djp9) · Building Permits | ~380 | same day |
| `cincinnati` — Cincinnati, OH | [data.cincinnati-oh.gov](https://data.cincinnati-oh.gov/d/uhjb-xac9) · Building Permits | ~200 | 1–3 days |
| `baton-rouge` — Baton Rouge, LA | [data.brla.gov](https://data.brla.gov/d/7fq7-8j7r) · Permits Issued | ~160 | 1–3 days |

Seattle and Cincinnati are building permits only (no separate trade-permit feed), which is why their
counts look low next to their metro size. Los Angeles is three LADBS datasets merged into one city.

### Administrative permits are dropped by default

Three of these registers are *general* permit registers rather than construction-permit feeds: they
publish non-construction, administrative records in the same table as building work. Those rows have
no job site scope, no valuation and no contractor to sell to, so the Actor excludes them **in the
query sent to the city**, not after the fact — they never consume your `maxItems` budget and you are
never charged for them. Set `includeAdministrative: true` to get the full register instead.

| City | Excluded types | Share of the feed |
|---|---|---|
| San Antonio | Garage Sale, Tree Affidavit Permit, Across the Street Banner, Temporary Weekend Sign, Feather Sign, Avenue Sign, Event Sign, Inflatable Sign | **~10%** (1,376 of 12,585 permits over 60 days) |
| Philadelphia | Operations Permit (temporary tents and canopies, fireworks displays) | ~0.6% |
| Baton Rouge | Tire Business, Short Term Rental, Complaint, Code Violations, Stop Work Order, Donation Box, Sign Permit (Political Campaign), Occupancy Permit (Special Event) | ~2% |

Permits that involve actual physical work are deliberately **kept**, even where the name looks
administrative: permanent sign and billboard erection, San Antonio tree-removal permits,
Philadelphia zoning permits and certificates of occupancy, Baton Rouge occupancy and re-roof permits.
Garage sales are the single biggest offender — San Antonio issues about 3,800 a year.

One other thing to know about San Antonio: its `work_description` is the city's PROJECT NAME field,
which for small permits is often just the address repeated.

**Why other big metros aren't here:** Dallas, Houston, Phoenix, Denver, Miami, Fort Worth, Nashville
and Mesa were all tested and rejected. Their open-data permit feeds are either discontinued, frozen
years in the past, or served only as ArcGIS layers that no longer respond. Shipping a city that
silently returns zero rows is worse than not shipping it, so they're excluded until their feeds come
back.

---

## Input

```json
{
  "cities": ["austin", "san-antonio"],
  "daysBack": 7,
  "maxItems": 1000,
  "permitTypes": "roof, solar",
  "includeAdministrative": false
}
```

| Field | Type | Default | Notes |
|---|---|---|---|
| `cities` | array of enum | `["austin"]` | Any of the 11 keys in the table above. |
| `daysBack` | integer | `7` | Days of issued permits counting back from today (UTC). Max 365. |
| `startDate` | string (ISO) | — | Earliest issued date, `YYYY-MM-DD`. Overrides `daysBack`. |
| `endDate` | string (ISO) | today | Latest issued date, `YYYY-MM-DD`. |
| `maxItems` | integer | `1000` | Cap across **all** selected cities combined, split evenly between them so no single city eats the whole budget. Unused allowance rolls forward. `0` = unlimited. |
| `permitTypes` | string | — | Comma-separated keywords. A permit is kept when any keyword appears in its permit type, permit class or work description (case-insensitive). |
| `includeAdministrative` | boolean | `false` | Include the non-construction records some registers mix in — San Antonio garage sales and tree affidavits, Philadelphia tent/fireworks permits, Baton Rouge business and short-term-rental licences. Off by default; see the table above. |

The date filter always runs against the **issued** date, not the application date.

---

## Output

One item per permit. Every item carries all 18 fields; anything the city doesn't publish is `null`
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
a city publishes is lost even if this Actor's normalized schema doesn't have a home for it.

### What each city actually fills in

Field coverage is a property of the city, not of the scraper. Measured on every permit issued in
the 10 days ending 2026-09-02, with `includeAdministrative` off:

| City | Contractor | Contractor phone | Valuation | Owner | Applied date | Lat/lng | ZIP |
|---|---|---|---|---|---|---|---|
| Austin | 94% | **92%** | 37% | 17% | 100% | 56% | 100% |
| San Antonio | 98% | — | 3% | — | 100% | 95% | 96% |
| New York | 100% | — | 100% | 100% | 100% | 100% | 100% |
| Los Angeles | — | — | 100% | — | 60% | 99% | 100% |
| Chicago | 98% | — | 86% | 98% | 100% | 98% | 97% |
| Philadelphia | 87% | — | — | 95% | — | 100% | 100% |
| Boston | 95% | — | 100% | — | — | 98% | 100% |
| Seattle | — | — | 100% | — | 93% | 100% | 91% |
| San Francisco | — | — | 100% | — | 100% | 100% | 100% |
| Cincinnati | 99% | — | 100% | — | 100% | 99% | 99% |
| Baton Rouge | 100% | — | 100% | 80% | 100% | 100% | 99% |

A dash means the city does not publish that field at all, so it is `null` on every row:

* **Philadelphia and Boston publish no application date.** Neither the Philadelphia L&I `permits`
  table nor Boston's approved-permits resource has an application/filed-date column — only issued,
  expiration and completion dates. There is nothing to map.
* **Austin's lat/lng gap is Austin's.** 44% of Austin's issued permits ship with no coordinates at
  all — the rows are missing `latitude`, `longitude` *and* the `location` object, so there is no
  other key to fall back to. Every other city geocodes 95–100% of its rows.
* **Valuation is a percentage of rows carrying a number, not a number above zero.** Austin, San
  Antonio and Baton Rouge all publish `0` on a meaningful share of their trade permits. Austin's 37%
  is up from 14%: where the city leaves `total_job_valuation` empty the Actor now sums the per-trade
  component valuations (building, electrical, mechanical, plumbing, medical gas, and their remodel
  variants) instead of reporting nothing.
* Los Angeles and Seattle are the only cities whose applied-date coverage is partial rather than
  all-or-nothing.

**Austin is the only city that publishes contractor phone numbers.** If dialable leads are the point,
Austin is the city to start with. For the rest you get a contractor name to match against a license
board or a business directory.

---

**`permit_number` is not a unique key.** Several cities issue one number per job or project and then publish one row per trade or address under it: New York (one job, separate Plumbing / Sprinklers / Standpipe rows), San Antonio (one commercial project, separate electrical / plumbing / mechanical rows), Cincinnati (one project, separate Building / HVAC / Excavation rows) and San Francisco (one permit, one row per street address it covers). Every such row is a distinct source record and is returned. Deduplicate on `permit_number` + `permit_type` + `address` if you need one row per trade, or on `permit_number` alone if you need one row per job.

## What people use this for

**Contractor lead generation.** A permit for a new roof is a buying signal for gutters, solar and
siding. Filter to your trade with `permitTypes`, pull the last 7 days, and work the list. Austin's
contractor phone numbers make it a dialable call list on day one.

**Building-material and equipment sales.** New-construction and addition permits with a declared
valuation tell a supply rep which jobs are about to need product and roughly how big they are. Sort
`valuation` descending, filter by `zip`, and route by territory.

**Real-estate and investment research.** Permit velocity by ZIP is one of the earliest available
signals of where capital is actually landing — visible months before it shows up in sales comps or
assessor data. `lat`/`lng` are populated well enough in most cities to map directly.

**Market and competitive analysis.** Group by `contractor_name` to see who is winning work by trade,
neighborhood and job size, and how that shifts quarter over quarter.

**Compliance and monitoring.** Watch permits at addresses or ZIPs you care about and get told when
someone pulls one.

---

## How it works

* Queries each city's official API directly — Socrata SoQL (`$where` / `$order` / `$limit` /
  `$offset`, 1,000 rows a page), CKAN `datastore_search_sql`, or the Carto SQL API.
* Cities are scraped independently inside their own try/except: one city's outage doesn't take down
  the run, and per-city counts are logged either way.
* Failed requests retry three times with exponential backoff.
* Results are de-duplicated by permit number within each city.
* A `RUN_SUMMARY` record is written to the default key-value store with the resolved date range and
  the item count per city.

Runs comfortably in 256–1024 MB. A single-city week is a few seconds of compute.

### Refresh cadence

Every source refreshes daily. Most publish same-day; San Antonio typically lags 2–4 days, and a few
of the smaller feeds lag 1–3 days, so a `daysBack` of 2 can legitimately return zero rows for those
cities. `daysBack: 7` on a daily schedule is the reliable pattern.

---

## Data source and attribution

All data comes from public municipal open-data portals published by the cities themselves. This
Actor only reads documented public API endpoints — no HTML scraping, no authentication, no proxies.

Permit records are public records. Licensing follows the source portal (San Antonio, for example,
publishes under CC-BY). Attribute the originating city when you redistribute, and check the source
portal's terms for your use case. `source_url` on every item points back to the record or dataset it
came from.

The cities are the system of record. If a permit looks wrong, it looks that way in the city's feed
too — this Actor does not correct, infer or enrich, it normalizes.

---

## Pricing

Priced per event: a small flat fee when a run starts, plus a per-permit charge for each record
written to the dataset. You pay for permits you actually receive.
