<p align="center"><a href="https://apify.com/DevilScrapes/equity-crowdfunding-leads"><img src="assets/banner.svg" alt="Equity Crowdfunding Leads Scraper — Python example" width="100%"></a></p>

# Equity Crowdfunding Leads Scraper in Python: Wefunder, Republic and StartEngine to pandas

<p align="center">
<a href="https://apify.com/DevilScrapes/equity-crowdfunding-leads"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/equity-crowdfunding-leads-scraper-python/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/equity-crowdfunding-leads-scraper-python/actions/workflows/ci.yml/badge.svg"></a>
</p>

This repository is a compact, tested Python wrapper around the **[Equity Crowdfunding Leads Scraper](https://apify.com/DevilScrapes/equity-crowdfunding-leads)** on Apify. One command asks the Actor for campaigns from Wefunder, Republic and StartEngine and saves them locally as `results.json` and `results.csv`, already in a shared layout. For Wefunder campaigns that means founder, pitch line, region, target, amount raised, number of backers and pre-money valuation as a real number, which is exactly the shape a DataFrame wants. The collection runs on Apify's servers, so a notebook or a cron job on your laptop only waits for the run and reads the file.

> **Actor:** [Equity Crowdfunding Leads Scraper](https://apify.com/DevilScrapes/equity-crowdfunding-leads) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in JavaScript:** [equity-crowdfunding-leads-scraper-javascript](https://github.com/devilscrapes/equity-crowdfunding-leads-scraper-javascript)

## 🔥 What you get

- **A single row layout for three portals**: each campaign carries a `source` value, so Wefunder, Republic and StartEngine records sit in the same table and you filter them with one column instead of three parsers.
- **Wefunder rows come loaded**: the disclosed founder, the one-line pitch, region, goal, money raised so far, investor count, last-year revenue when published, and the pre-money valuation.
- **Valuation as a number you can sort**: shorthand such as `$62.1M` or `$700K` is turned into a plain USD float, and a value that cannot be read becomes `null` rather than a crash.
- **Honest coverage on the thinner sources**: Republic and StartEngine rows give the slug, company name and campaign link; their money fields stay `null` because those sites render them in the browser.
- **Keyword narrowing**: an optional case-insensitive substring is matched against each row's tagline or industry, and rows that match neither are left out before billing.
- **A broken portal costs you one source, not the run**: if a platform fails or stays blocked, it is logged and skipped while the remaining sources still deliver rows.
- **Throttling handled for you**: rate-limited or blocked pages are retried with growing pauses on a new proxy session before a source is given up.

## 🎯 Use cases

### Weekly deal-flow sheet for a scout or syndicate

Schedule `python main.py` on Monday mornings, read the CSV into pandas and sort the Wefunder rows by `raised_amount_usd`. The campaigns that are pulling money fastest float to the top, each one with a link back to its pitch page, and the whole review fits in one notebook cell.

### Valuation benchmarks by sector

Group the funded or raising rows by a keyword taken from `tagline` or `industry`, then compute the median `valuation_usd` per group. That gives founders and analysts a quick reference for what Reg CF companies in a given niche are asking, without copying numbers off campaign pages by hand.

### Raise-velocity tracking with dated snapshots

Each run is a fresh snapshot, so keep every `results.csv` under a dated name and join consecutive files on `source` and `campaign_slug`. The change in `raised_amount_usd` and `num_investors` between two dates is a clean momentum metric for charts or a research note.

### Founder list for outreach research

Explode the `founders` column so every name sits on its own row beside the company and campaign link, then import that sheet into your CRM. Teams selling to early-stage startups get companies that are visibly raising right now.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/equity-crowdfunding-leads-scraper-python.git
cd equity-crowdfunding-leads-scraper-python
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export APIFY_TOKEN=your_token_here
python main.py                      # uses input.json
python main.py my-input.json --out data --max-charge 2
```

The script starts the [Equity Crowdfunding Leads Scraper](https://apify.com/DevilScrapes/equity-crowdfunding-leads), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Stripped down to the minimum, the integration is the snippet below. The `apify-client` package starts the run, blocks until it ends and then iterates the dataset page by page.

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("DevilScrapes/equity-crowdfunding-leads").call(run_input={'sources': ['wefunder'], 'maxPerSource': 5, 'statusFilter': 'active', 'useProxy': True})
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item)
```

## 📥 Input

`main.py` hands `input.json` to the Actor without changing it. The bundled example reads Wefunder only, keeps campaigns that are currently raising and stops after five rows, which is enough to see the full field set for a few cents. Leave `sources` empty to read all three portals, raise `maxPerSource` up to 500 per portal, or set `industryFilter` to a word like `fintech`.

```json
{
  "sources": [
    "wefunder"
  ],
  "maxPerSource": 5,
  "statusFilter": "active",
  "useProxy": true
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `sources` | array | no | `[]` | — |
| `maxPerSource` | integer | no | `50` | — |
| `statusFilter` | string | no | `"active"` | `active`, `funded`, `all` |
| `industryFilter` | string | no | — | — |
| `useProxy` | boolean | no | `true` | — |

## 📤 Output

Each dataset row is one campaign. Money fields are plain floats in USD and `num_investors` is an integer, so pandas types them correctly on load; expect `NaN` in those columns for Republic and StartEngine rows, and for any Wefunder campaign that does not disclose a figure. `founders` is a list, which pandas keeps as a Python list per cell.

| Field | Type | Always present |
|---|---|---|
| `source` | string | yes |
| `campaign_slug` | string | yes |
| `company_name` | string | yes |
| `tagline` | string or null | no |
| `industry` | string or null | no |
| `location` | string or null | no |
| `founders` | array | yes |
| `website_url` | string or null | no |
| `target_amount_usd` | number or null | no |
| `raised_amount_usd` | number or null | no |
| `num_investors` | integer or null | no |
| `valuation_usd` | number or null | no |
| `revenue_usd` | number or null | no |
| `funding_stage` | string or null | no |
| `campaign_url` | string | yes |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Campaign row | $0.005 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/equity-crowdfunding-leads) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in Python

This pandas sketch (`pip install pandas`) ranks the campaigns that disclose both money raised and valuation, adds two ratios, and writes a one-founder-per-row contact sheet next to it.

```python
import json

import pandas as pd

rows = pd.DataFrame(json.load(open("results.json", encoding="utf-8")))

priced = rows.dropna(subset=["raised_amount_usd", "valuation_usd"]).copy()
backers = priced["num_investors"].where(priced["num_investors"] > 0)
priced["usd_per_investor"] = (priced["raised_amount_usd"] / backers).round(0)
priced["raised_share_of_valuation"] = (priced["raised_amount_usd"] / priced["valuation_usd"]).round(4)
ranked = priced.sort_values("raised_amount_usd", ascending=False)
print(ranked[["company_name", "raised_amount_usd", "num_investors", "usd_per_investor", "valuation_usd"]].head(15).to_string(index=False))

contacts = rows.explode("founders").dropna(subset=["founders"])
contacts = contacts.rename(columns={"founders": "founder"})
contacts[["founder", "company_name", "source", "location", "campaign_url"]].to_csv("founders.csv", index=False)
print(f"{len(contacts)} founder rows saved to founders.csv")
```

## ❓ FAQ

### Do I need an account on Wefunder, Republic or StartEngine?

No. The only credential is your Apify API token. None of the three portals asks you for a login, an investor profile or an API agreement to use this script.

### Why are the money columns empty for Republic and StartEngine rows?

Those two sites fill in raise totals, valuations and investor counts in the browser, so the Actor currently returns slug, company name and campaign URL for them. When you need numbers for analysis, set `sources` to `["wefunder"]` and every row has the full field set the campaign discloses.

### Why did I get fewer rows than `maxPerSource` with an industry filter?

The cap is applied when campaigns are collected and the keyword filter runs afterwards, so a cap of 50 with `industryFilter` set to `climate` keeps only the matching part of those 50. Raise the cap if the filtered sample is too small; you are charged only for rows that pass.

### Can I build a history of how campaigns progress?

Yes, on your side. The Actor stores no history between runs, and Apify's default storage on the free plan is removed after seven days. Save each CSV with its date, or append to a Parquet file, and compare snapshots on `source` plus `campaign_slug`.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/equity-crowdfunding-leads) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Equity Crowdfunding Leads Scraper on Apify Store](https://apify.com/DevilScrapes/equity-crowdfunding-leads)
- [JavaScript version of this example](https://github.com/devilscrapes/equity-crowdfunding-leads-scraper-javascript)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [SEC EDGAR Filings Scraper — 10-K, 10-Q & 8-K](https://apify.com/DevilScrapes/sec-edgar-filings-scraper) — follow a crowdfunded issuer to its filings on SEC EDGAR.
- [New Business Filings — US State Registry Leads Feed](https://apify.com/DevilScrapes/new-business-filings-leads-scraper) — freshly registered US companies from state registries, for an even earlier signal.
- [OpenCorporates Alternative — US Business Registry Scraper](https://apify.com/DevilScrapes/opencorporates-alternative-scraper) — confirm a company's registration status and officers in US state registries.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/equity-crowdfunding-leads) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with Wefunder, Republic or StartEngine.
