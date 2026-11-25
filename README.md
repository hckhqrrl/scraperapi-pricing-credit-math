# Scraper API review: does $49 buy enough, what does a rendered page really cost, and when should you look elsewhere? (full plan table + credit math)

ScraperAPI charges by credits, and the credit multiplier attached to a request decides your bill far more than the plan price does. On the $149 Startup plan a plain HTML page works out to roughly **$0.15 per 1,000 requests**; turn on JavaScript rendering and it becomes $1.49 per 1,000, and the heaviest combination of premium proxies plus rendering lands at **$11.18 per 1,000**. This review lays out the multiplier table, converts it into real cost per 1,000 pages on each plan, explains the billing rule that catches people out, and shows which target sites it handles and which ones it doesn't.

> **Quick verdict.** If you already have working scraping code and want a fetch layer that handles proxies, CAPTCHAs and rendering, ScraperAPI fits, and most teams should start on **Hobby at $49/month for 100,000 credits** to measure their real burn rate, then move to **Startup at $149 for 1,000,000 credits** once plain-HTML volume passes about 100,000 pages. Skip it if your targets sit behind DataDome or you need city-level or ASN-level IP targeting. 👉 [Compare every ScraperAPI plan and its current monthly price](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Every plan on the current price list

| Plan | Credits / month | Concurrent threads | Geotargeting | Price / month | Link |
| --- | --- | --- | --- | --- | --- |
| Free | 1,000 | 5 | US & EU | $0 | [Start on the free plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Hobby | 100,000 | 20 | US & EU | $49 | [Get Hobby at $49/month](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | US & EU | $149 | [Get Startup at $149/month](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Country-level | $299 | [Get Business at $299/month](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | Country-level | Not public | [See Scaling on the pricing page](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Country-level | Not public | [See Professional on the pricing page](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Country-level | Not public | [See Advanced on the pricing page](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Country-level | Not public | [See Enterprise on the pricing page](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

ScraperAPI publishes monthly prices for Hobby, Startup and Business only. The four larger tiers are quoted through sales. Analytics history is capped at 30 days on Hobby and Startup and unlimited from Business up.

Three billing rules sit under that table. Credits **do not roll over** to the next month. Pay-as-you-go overage exists only from Scaling upward, so on Hobby, Startup and Business you either opt into auto-upgrade or scraping pauses until credits renew. Going over your concurrency limit returns a 429 and no charge; the request is refused rather than queued.

## Credit multipliers decide your bill

Every request starts at a base cost set by the target domain, then parameters stack on top.

| Request type | Credits per request |
| --- | --- |
| Standard page | 1 |
| Amazon, Walmart, eBay | 5 |
| Google, Bing (all subdomains) | 25 |
| LinkedIn | 30 |
| Bypassing Cloudflare, Cloudflare Turnstile, DataDome, PerimeterX/HUMAN or Akamai | +10 |
| `render=true` (JavaScript rendering) | 10 |
| `premium=true` (residential or mobile IPs) | 10 |
| `screenshot=true` | 10 |
| `premium=true` + `render=true` | 25 |
| `ultra_premium=true` | 30 |
| `ultra_premium=true` + `render=true` | 75 |

`country_code`, `session_number`, `device_type`, `output_format` and `autoparse` add nothing to the cost. That matters more than it looks: geotargeting is free, so a US-only requirement on Hobby costs the same as an untargeted request.

Convert those multipliers into money and the spread becomes obvious. The figures below are arithmetic from the published multipliers and the three published monthly prices, not estimates from a test.

| Scenario | Credits | Hobby ($49 / 100k) | Startup ($149 / 1M) | Business ($299 / 3M) |
| --- | --- | --- | --- | --- |
| Static HTML page | 1 | $0.49 / 1k | $0.15 / 1k | $0.10 / 1k |
| E-commerce page (Amazon, Walmart, eBay) | 5 | $2.45 / 1k | $0.75 / 1k | $0.50 / 1k |
| JS-rendered page | 10 | $4.90 / 1k | $1.49 / 1k | $1.00 / 1k |
| Premium proxy + rendering | 25 | $12.25 / 1k | $3.73 / 1k | $2.49 / 1k |
| Ultra premium + rendering | 75 | $36.75 / 1k | $11.18 / 1k | $7.48 / 1k |

The practical reading: Hobby is a metering plan for finding out what your targets cost, not a production plan for rendered pages. At $4.90 per 1,000 rendered pages you burn through 100,000 credits in 10,000 requests.

## The plan-jump trap nobody mentions in the pricing table

Suppose your workload needs 1.1 million credits a month. Startup gives you 1 million, and Startup has no pay-as-you-go overage, so the next step is Business at **$299** rather than extending Startup by 10%. A 10% overshoot doubles the bill.

Two habits prevent this. First, price the request before you send it: every response carries an `sa-credit-cost` header, the `/account/urlcost` endpoint quotes a URL with all its parameters applied, and the dashboard has a Domain Cost Estimator. Second, set `max_cost` on every request. If the request would exceed your cap, the API returns a 403 instead of silently charging more. 👉 [Check what a Startup allowance of 1,000,000 credits covers at your request mix](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## The billing rule that shows up on the invoice

ScraperAPI's documented rule is that you are charged only for successful requests, defined as HTTP **200 and 404**. Failed requests are free, which is a genuine advantage over per-GB proxy billing where every blocked attempt costs bandwidth.

The exception is what causes complaints. A page that returns HTTP 200 but contains an error message counts as a successful request. In a November 2026 third-party test, 17 pages on a DataDome-protected category page came back with a 200 status, the text "A system error has occurred" and no products at all; each was billed 10 to 40 credits. From the customer's side that reads as being charged for a blocked page, even though the mechanism is the 200 status.

There is a second timing issue. Difficult targets are served from a 10-minute result cache, so repeated hits can return stale data. Send `cache_control=no-cache` to skip it.

## Where it works, and where it stalls

Two recent benchmarks measured different target sets, so their headline numbers should not be averaged. Both are worth reading separately.

**A 5-target test published 4 November 2026** ran 45 requests: ScraperAPI returned the full page in 38, a competing unlocker in 44. Almost the entire gap came from one DataDome-protected category page, where ScraperAPI's best configuration (premium proxies, rendering, US geotargeting at 40 credits per request) succeeded 3 times out of 10 with a median of 43.28 seconds. ScraperAPI was faster on Indeed (8.82 s vs 12.41 s) and Walmart (6.37 s vs 9.02 s), both on default requests with no rendering. A JavaScript infinite-scroll test loaded 100 items in 5 of 5 runs at 26.08 seconds using rendering instructions.

**An October 2026 benchmark across 11 targets** put ScraperAPI at 60% overall success, fifth of six APIs tested, with a 10.6-second average response and $5.81 per 1,000 requests against an industry average of 72.3% and $3.88. The per-target split is more useful than the average:

| Strong targets | Success | Weak targets | Success |
| --- | --- | --- | --- |
| Zillow | 100% | Booking | 47% |
| Etsy | 96% | Indeed | 41% |
| Amazon | 94% | StockX | 37% |
| Walmart | 89% | Realtor | 37% |
| LinkedIn | 89% | Twitter / Instagram | 0% |

Read together, the pattern is consistent: static pages, e-commerce catalogues and real-estate listings work well, social platforms and travel sites do not. The 60% and the 38-of-45 figure measure different site lists and should not be combined into one success rate.

## Limits that only appear in the details

- **Geotargeting is US and EU only on Hobby and Startup.** Country-level targeting starts at Business ($299).
- **The trial blocks the top proxy pool.** The 7-day, 5,000-credit trial returns HTTP 403 on `ultra_premium=true` requests, so you cannot benchmark the hardest targets before paying.
- **No specific IPs, ASNs or cities.** You choose country, session and IP tier by design. If you need finer routes, buy raw proxies.
- **Slow requests cap out.** Protected domains can take up to 60 seconds; the documentation recommends a 70-second timeout. Volume work belongs on the Async API, which accepts batches up to 50,000 URLs and returns results via status URL or webhook.
- **Older reviews are out of date.** Screenshots (10 credits, PNG link in the `sa-screenshot` header) and scripted interaction through the `x-sapi-instruction_set` header both exist now, and the same header covers clicks, input, scroll, loop and wait steps. Output can be HTML, Markdown or text for any URL, plus JSON or CSV via `autoparse` on supported sites.

The product has also moved past the proxy layer: structured data endpoints for Amazon, Walmart, eBay, Home Depot, Google, Redfin and AI answer engines; DataPipeline for scheduled jobs up to 100,000 URLs per project; a crawler; and an MCP server for LLM workflows.

For a free trial that costs nothing to run: 👉 [Open a 5,000-credit, 7-day trial with no credit card](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Who should not buy ScraperAPI

**DataDome-protected sites are the weak spot.** The 3-of-10 result at 40 credits per request says to validate a small sample before committing a monthly plan. A competing unlocker in the same test returned 10 of 10 at a median of 12.86 seconds with default settings.

**City, ASN or compliance-grade targeting** points to Bright Data, which sells proxy bandwidth by the GB or unlocker requests by count and offers KYC-backed networks and SLAs.

**Running whole scraping jobs rather than a fetch layer** points to Apify: 83,000+ ready-made Actors, results stored in datasets, compute-based pricing, a free $5 monthly allowance and a $19 entry plan. ScraperAPI assumes you already own the code and just need the fetch.

On raw benchmark numbers, Scrapfly posted 95% success at $3.75 per 1,000 in the October 2026 test, and Firecrawl 81% at $2.70. Neither figure is directly comparable to your own targets, but both beat ScraperAPI's 60% on that particular list.

## FAQ

**How much can I test for free?** 5,000 credits over 7 days without a card. After that the free plan gives 1,000 credits a month and 5 concurrent threads.

**Why did one page cost 40 credits?** Base site cost, plus 10 for `render=true`, plus 10 for `premium=true`, plus 10 for bypassing bot protection. All of that landed on a single DataDome category page in the November 2026 test.

**Do unused credits roll over?** No. They reset with each billing cycle, so buy the tier that matches a typical month rather than a peak.

**Can it scrape Google and LinkedIn?** Yes, at 25 credits for Google or Bing and 30 for LinkedIn. Structured SERP endpoints return parsed JSON instead of HTML.

**Does the trial include the best proxies?** No. `ultra_premium=true` returns HTTP 403 until you are on a paid plan.

## How to choose, and what to check before paying

Only static HTML or simple catalogue pages: take **Hobby at $49** for a month, watch `sa-credit-cost` on real URLs, then decide whether Startup's $149 is cheaper per page at your volume.

Heavy JavaScript rendering: multiply your monthly page count by 10 first. 50,000 rendered pages means 500,000 credits, which still fits Startup, while 150,000 rendered pages needs 1.5 million credits and pushes you to Business at $299.

DataDome or similar hard targets: run 20 to 30 requests at premium plus rendering before subscribing, and count how many return real content rather than a 200 error page.

Two checks before checkout: price your highest-volume URL with the Domain Cost Estimator and confirm the monthly total leaves headroom, and confirm you are not within 10% of a tier ceiling, since Startup and below have no overage and a small overshoot moves you up a full plan.

👉 [Pick the plan that matches your credit estimate on ScraperAPI's pricing page](https://www.scraperapi.com/pricing/?fp_ref=coupons)
