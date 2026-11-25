# Geekflare Grounded Search: Pricing Details, Cheaper Tier Options, And Real-World Accuracy Actually — Is This Citation-Rich API Worth It Across Multiple Use Cases For Developers Building Trustworthy Answers?

Welcome to the **Geekflare Search API** guide for developers who need reliable, citation-rich answers straight from Google's SERP. If you've ever built a storefront that needs to surface "People Also Ask" (PAA) snippets, local pack listings, or Google AI Overviews alongside organic results, you know the pain of juggling rate limits, credit accounting, and flaky third-party scrapers. This README walks you through every engineering decision you'll face when integrating the **grounded answer search API**, from raw request mechanics to credit-by-credit cost modeling, and shows you exactly which Geekflare plan (or one-time credit pack) fits your workload without surprise bills.  

We'll stay honest about limits, compare Geekflare to two well-known rivals, and give you runnable code you can copy-paste into a terminal or your Managed Cloud Platform (MCP) server. All numbers, URLs, and feature lists are drawn from verified Geekflare documentation, so you can trust the math and avoid hidden fees. Let's get started.

---  

## What You Actually Get Inside Geekflare's API Suite

Geekflare bundles fourteen distinct APIs under a single API key and a shared credit pool. The **Search API** is the centerpiece for grounded answer retrieval, and it supports four exact request modes, each with a **total** credit cost that is independent of any base charge:

| Mode | Parameter(s) | Total Credits per Call |
|------|--------------|------------------------|
| Standard web search | `POST https://api.geekflare.com/search` (no extra flags) | **2** |
| Full Google SERP JSON | `serp: true` | **2** |
| Search + scrape top 1-10 result pages | `scrape: true` | **4** |
| Grounded AI answer with inline citations | `groundedAnswer: true` | **5** |

> **Note:** Credits are *total* per call. For example, enabling `scrape: true` does **not** add 2 credits for the underlying search; the whole operation costs 4 credits exactly.

**SERP payload you can rely on**

When you set `serp: true`, the response contains a structured JSON object that mirrors Google's first-page SERP. The payload includes:

* **Organic results** – title, snippet, URL, and ranking.
* **Ads** – top-of-page and bottom-of-page paid placements.
* **Google AI Overview** – `data.ai_overview` with plain text, HTML, and a list of cited sources.
* **People Also Ask (PAA)** – each question paired with a concise answer and the source URL.
* **Related searches**, **local pack**, **news**, **images**, and **popular products** sections.

All of these fields are returned at the same 2-credit cost, so you can build a single "search-once, render-everything" UI without worrying about hidden per-feature fees.

**Geo-targeting, device simulation, and filters**

* **Geo** – Pass `city: 'Austin,Texas,United States'` or any of the 65,000+ supported city strings. You can also use ISO country codes via the `location` field.  
* **Device** – Choose `device: 'desktop'` or `device: 'mobile'` to get results that match the chosen user-agent.  
* **Filters** – `source` (web, news, images), `time` (any, h, d, w, m, y, d7), `category` (general, code, pdf, research, linkedin, wiki), and domain inclusion/exclusion (`includeDomains`, `excludeDomains`).  
* **Output formats** – Request `json`, `markdown`, or `html` to match your downstream pipeline.

All geo, device, and filter options are included in the base 2-credit cost for standard or SERP searches. No extra credit charge for targeting a specific city or requesting JSON vs. HTML.

**Rate limits and reliability**

Geekflare enforces **requests per second (req/s)** limits per plan:

* Free – 1 req/s  
* Starter – 5 req/s  
* Growth – 10 req/s  
* Business – 25 req/s  
* Enterprise – up to 1,000 req/s (custom)

These limits are *hard* per-second caps; exceeding them returns a `429 Too Many Requests` error without consuming credits. The platform runs on Google Cloud Platform and Cloudflare, delivering a **99.9% uptime** SLA (see the About page). Failed 5xx responses or captures that return no data are **free** – they do not deduct credits, which protects you from accidental over-spending during transient outages.

**Credits, rollover, and alerts**

* **No rollover** – Each billing cycle resets your credit balance on day 1.  
* **Alert at 90%** – You receive an email when you've consumed 90% of your monthly allotment.  
* **Hard stop at 100%** – Requests are paused until you upgrade or add a credit pack.  
* **Refund guarantee** – Cancel within 7 days and get a full refund if you used less than 10% of your allocated credits.

These rules make budgeting predictable: you always know exactly how many calls you can make in a month, and you never get surprised by "unused credit" carry-over.

---  

## Working Code: Calling Geekflare in Minutes

Below are two runnable examples that hit the **grounded answer** endpoint. The first uses plain `cURL`; the second demonstrates how to invoke the same request through the hosted **Managed Cloud Platform (MCP)** server, which abstracts authentication and lets you reuse the same JSON payload across languages.

**Example 1 – cURL direct call**

```bash
curl -X POST https://api.geekflare.com/search \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_GEekFLARE_API_KEY" \
  -d '{
    "query": "What are the health benefits of turmeric?",
    "groundedAnswer": true,
    "serp": true,
    "city": "Austin,Texas,United States",
    "device": "desktop",
    "output": "json"
}'
```

*What you get*: A JSON response containing `data.grounded_answer` (the AI-generated answer), `data.citations` (array of source URLs), and the full SERP structure (`data.serp`). The call costs **5 credits**.

**Example 2 – MCP JSON payload**

If you prefer to keep your secrets out of the command line, drop the same payload into the MCP UI or send it via the MCP endpoint:

```bash
curl -X POST https://mcp.geekflare.com/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "search",
    "params": {
      "query": "How does quantum computing differ from classical computing?",
      "groundedAnswer": true,
      "serp": true,
      "city": "London,England,United Kingdom",
      "device": "mobile",
      "output": "markdown"
    }
}'
```

The MCP server forwards the request to `https://api.geekflare.com/search` using the API key you configured when you first logged into MCP. The response arrives in the same shape, but you can directly render the markdown in a documentation portal or a static site generator.  

Both examples illustrate that you can achieve **grounded answers with citations** in under a minute of setup, without writing any custom scraper or handling CAPTCHA challenges yourself. The proxy add-on (available on paid plans) gives you rotating IPs for a modest extra credit cost (+4 credits per request) if you need to bypass geographic restrictions.

---  

## SDKs, MCP Server, AI Frameworks, and No-Code Connectors

**Official language SDKs**

| Language | Package | Install command |
|----------|---------|-----------------|
| Python | `geekflare-api` | `pip install geekflare-api` |
| Node/TypeScript | `@geekflare/api-node` | `npm install @geekflare/api-node` |

Both SDKs expose a `search()` method that mirrors the raw HTTP payload, automatically injects the `x-api-key` header, and returns the parsed JSON. The SDKs also expose a `creditsRemaining()` helper that reads the `X-Credits-Remaining` response header, so you can build runtime alerts in your own monitoring stack.

**Managed Cloud Platform (MCP)**

The hosted MCP server (`https://mcp.geekflare.com/mcp`) ships with **18 built-in tools**, including `search`, `webScrape`, `screenshot`, `metaScrape`, `url2Pdf`, and more. You can invoke any tool via a single JSON payload (see Example 2). The MCP UI also offers a visual request builder, which is handy for non-developers or for quick proof-of-concepts.

**AI framework integrations**

* **LangChain** – Install `langchain-geekflare` (`pip install langchain-geekflare`) to wrap the Search API as a retriever node.  
* **LlamaIndex** – Install `llama-index-tools-geekflare` (`pip install llama-index-tools-geekflare`).  
* **Vercel AI SDK** – The package name is `@geekflare/ai-sdk`; it provides a `search` function that returns grounded answers ready for LLM prompting.  

These integrations let you blend the SERP data with your own LLM pipelines, turning raw citations into context windows for downstream generation.

**No-code automation**

* **n8n** – Use the official n8n node (`n8n.io`).  
* **Zapier** – Connect the Search API to Google Sheets, Slack, or a CRM via the Zapier integration (`zapier.com`).  
* **Make (formerly Integromat)** – A ready-made module exists (`make.com`).  
* **Google Sheets** – You can call the API directly from Apps Script by setting the `x-api-key` header; no marketplace URL is required.

All of these connectors respect the same credit accounting, so you can safely expose the API to business users without risking uncontrolled spend.

---  

## Modeling Monthly Request Volume Without Pricing Surprises

Below we translate typical use-cases into **credit consumption** and map them to the appropriate Geekflare tier. The math uses the exact per-credit pricing from the verified pricing page; we never multiply custom dollar amounts.

| Use-case | Calls per month | Credits per call | Total credits | Cheapest tier that fits |
|----------|----------------|------------------|---------------|--------------------------|
| 100 keywords daily, standard SERP only | 3,000 queries | 2 | 6,000 | **Starter** ($19/mo, 10 K credits) |
| 2,000 PAA look-ups per month (SERP + grounded answer) | 2,000 | 5 | 10,000 | **Starter** (exact fit) |
| 5,000 product-data extractions (search + scrape) | 5,000 | 4 | 20,000 | **Growth** (100 K credits) |
| 25,000 AI-grounded answers for a knowledge base | 25,000 | 5 | 125,000 | **Business** (1 M credits) |
| 500,000 standard SERP calls for a SaaS dashboard | 500,000 | 2 | 1,000,000 | **Business** (exact fit) |
| Burst of 10,000 grounded answers in a single day (Enterprise) | 10,000 | 5 | 50,000 | **Enterprise** (custom) |

**Credit-by-credit cost comparison (per 1,000 calls)**

| Tier | Standard / SERP (2 credits) | Search + scrape (4 credits) | Grounded answer (5 credits)  | Link |
|------|------------------------------|-----------------------------|------------------------------ | --- |
| Free | $0 / 250 queries | – | –  | [Start Free](https://geekflare.com/pricing/) |
| Starter | $3.80 / 1 K | $7.60 / 1 K | $9.50 / 1 K  | [Get Starter](https://geekflare.com/pricing/) |
| Growth | $1.38 / 1 K | $2.76 / 1 K | $3.45 / 1 K  | [Get Growth](https://geekflare.com/pricing/) |
| Business | $0.70 / 1 K | $1.40 / 1 K | $1.75 / 1 K  | [Get Business](https://geekflare.com/pricing/) |

Because credits reset each month, you can safely purchase a **Spark** pack ($10 for 5,000 credits) on top of a Free plan to handle occasional spikes without upgrading to a paid tier. The same logic applies to Blaze, Inferno, and Supernova packs.

---  

## Inside Geekflare's Pricing Tiers and Shared Credit Pool

| Plan | Monthly price | Credits / month | Rate limit (req/s) | Team members | Log retention | Proxy add-on |
|------|---------------|-----------------|--------------------|--------------|---------------|--------------|
| **Free** | $0 | 500 | 1 | 1 | 7 days | **No** |
| **Starter** | $19 | 10,000 | 5 | 3 | 30 days | +4 credits per request (paid only) |
| **Growth** | $69 /mo *or* $58 /mo billed yearly | 100,000 | 10 | 5 | 30 days | +4 credits |
| **Business** | $349 | 1,000,000 | 25 | 25 | 90 days | +4 credits |
| **Enterprise** | Custom | Custom | up to 1,000 | Custom | Custom | +4 credits (custom) |

*All plans share a **single credit pool** across the fourteen Geekflare APIs, so you can mix SERP, web-scrape, screenshot, and URL-to-PDF calls without juggling separate balances.*

**Key billing mechanics**

* **No credit rollover** – Unused credits vanish on the first of the next month.  
* **90% usage alert** – Email notification when you cross the 90% threshold.  
* **Hard stop at 100%** – Requests are blocked until you upgrade or add a credit pack.  
* **Failed 5xx requests are free** – No credit charge for server-side errors.  
* **7-day money-back guarantee** – If you cancel within a week and have consumed less than 10% of your allocated credits, you get a full refund.  

**One-time credit packs (stackable)**

| Pack | Cost | Credits | Approx. per-1K credit cost |
|------|------|---------|----------------------------|
| Spark | $10 | 5,000 | $2.00 |
| Blaze | $50 | 50,000 | $1.00 |
| Inferno | $100 | 100,000 | $1.00 |
| Supernova | $200 | 200,000 | $1.00 |

Credit packs are valid for one year from purchase and can be applied to **any** plan, including Free. This gives you a cheap way to handle occasional high-volume bursts without committing to a higher-priced subscription.

---  

## The Main Alternatives, Honestly Compared

| Provider | Free tier | Credit cost (per 1 K) | Key strength | Major trade-off |
|----------|-----------|-----------------------|--------------|----------------|
| **Geekflare** | 500 credits / month, no credit card required | Starter $3.80 / 1 K (standard), $9.50 / 1 K (grounded) | Full Google SERP JSON with PAA, AI Overview, and inline citations; shared credit pool; 99.9% uptime | SERP limited to Google only; page-1 only |
| **Perplexity Search API** | No permanent free tier | $5 / 1 K search + Sonar token fees | High-quality Sonar grounded answers | No raw page scrape; no free tier; pricing adds token fees |
| **Tavily** | 1,000 free credits | $8 / 1 K (search) | Agent-centric bundling of snippets & synthesized answers | No full Google SERP fields; higher per-credit cost than Geekflare Growth |

**Takeaway:** Geekflare wins on **feature completeness** (PAA, local pack, AI Overview) and **predictable credit pricing**, especially once you move to Growth or Business tiers. Perplexity offers superior answer quality at a higher price and without raw SERP data, while Tavily gives a modest free quota but lacks the granular SERP fields many storefronts need.

---  

## Which Plan Should You Start With — And When to Upgrade?

| Monthly request pattern | Credits needed | Recommended plan | Reason |
|--------------------------|----------------|------------------|--------|
| Occasional ad-hoc queries (< 250 standard calls) | ≤ 500 | **Free** | No card required, fits within 500-credit limit. |
| Daily keyword monitoring (≈ 3,000 standard calls) | 6,000 | **Starter** | 10 K credits cover the load; 5 req/s is ample for cron jobs. |
| PAA extraction for 2,000 queries (grounded answer) | 10,000 | **Starter** | Exact match; you stay under the 10 K ceiling. |
| Product-catalog enrichment (5,000 scrape calls) | 20,000 | **Growth** | 100 K credits give headroom for future growth; 10 req/s supports parallel pipelines. |
| Enterprise-grade dashboard (500,000 standard calls) | 1,000,000 | **Business** | 1 M credits align perfectly; 25 req/s handles burst traffic. |
| Sudden spike of 10,000 grounded answers in a day | 50,000 | **Enterprise** (custom) or **Growth** + **Blaze** pack | Enterprise gives the needed 1,000 req/s; alternatively, a Blaze pack adds 50 K credits instantly. |

If you're on a paid plan and anticipate a one-off surge, buying a **Blaze** pack ($50 for 50 K credits) is often cheaper than upgrading to a higher tier for a single month. Remember that proxy access (+4 credits per request) is only available on paid tiers, so any scrape-heavy workload should stay on at least **Starter**.

---  

## What Actually Matters When Evaluating Grounded Answer Search API Tools

1. **Citation fidelity** – Does the API return source URLs alongside the answer? Geekflare's grounded answer mode includes explicit citations, which is essential for compliance audits.  
2. **Credit transparency** – Fixed per-call credit costs let you forecast spend. Some rivals hide per-token fees behind "usage tiers," making budgeting harder.  
3. **Feature breadth** – Access to PAA, local pack, Google AI Overview, and full SERP JSON in one call reduces the number of round-trips and simplifies data pipelines.  
4. **Rate-limit elasticity** – If your product needs to burst (e.g., a flash-sale price check), higher req/s limits prevent throttling. Enterprise can scale to 1,000 req/s.  
5. **Reliability & uptime** – 99.9% uptime across GCP and Cloudflare means you can trust the service for production-grade storefront monitoring.  

When you line up these criteria, Geekflare consistently checks the boxes for enterprise-ready, audit-friendly search, while still offering a free tier for early experimentation.

---  

## Should You Pay Yearly or Stack One-Time Credit Packs?

* **Yearly billing** – Growth drops from **$69 /mo** to **$58 /mo** when you commit to an annual contract. That's a two-month-free discount, but you still pay the same per-credit rates (0.00069 $ per credit). If your usage is stable, annual billing saves you the equivalent of two months of subscription fees.  
* **One-time credit packs** – If your workload is irregular, buying a **Spark** ($10 / 5,000 credits) or **Blaze** ($50 / 50,000 credits) pack lets you stay on the Free or Starter plan and only pay for the extra capacity you actually need. Packs never expire within a year, so you can keep them on hand for seasonal spikes.  
* **Refund guarantee** – Regardless of billing cadence, you can cancel within seven days and receive a full refund if you've used less than 10% of your allocated credits. This safety net makes it low-risk to try a higher tier for a pilot.  

In practice, most teams start on **Starter** (or Free for prototyping), monitor credit consumption for a month, then decide whether the **Growth** plan's lower per-credit price or a **Blaze** pack best fits their growth trajectory.

---  

## The Honest Bottom Line: When to Pick Geekflare vs an Alternative

Geekflare delivers a **complete, citation-rich Google SERP** in a single API call, with transparent credit pricing and a generous free tier that requires no credit card. Its 4-digit credit pool, 99.9% uptime, and 4.9-average user rating (G2 4.8, Google 4.9, Product Hunt 5.0) make it a solid default for any storefront that needs **People Also Ask**, local packs, or AI Overviews alongside raw organic results.  

The main limitations are the **Google-only SERP** and the **single-page scope** – you won't get deep pagination or non-Google verticals like Flights or Trends. If your product relies on multi-page crawling or needs Bing results, a competitor such as Perplexity (for pure answer quality) or Tavily (for a modest free quota) might be worth a look.  

Overall, for **grounded answer search** where compliance, audit trails, and predictable spend are non-negotiable, Geekflare is the most balanced, honest, and cost-effective option.

---  

## Frequently Asked Questions

### How do I retrieve the raw Google SERP JSON without paying extra credits?
Set `serp: true` in your request payload. The entire SERP (organic, ads, PAA, AI Overview, local pack, etc.) is returned for a flat **2-credit** cost.

### Can I use the Search API without a credit card?
Yes. The **Free** tier provides 500 credits per month and does not require a credit card. For any plan that needs proxy access or higher limits, a card is required.

### What happens if I exceed my monthly credit limit?
Requests are paused once you hit 100% of your allocated credits. You'll receive an email at 90% usage. To continue, either upgrade your plan or purchase a one-time credit pack (Spark, Blaze, etc.).

### Are failed 5xx responses charged against my credit balance?
No. Any request that results in a server error (5xx) or a capture that returns no data is **free** and does not deduct credits.

### Is there a way to test the API before committing to a paid plan?
You can start on the **Free** tier, which gives you 500 credits per month. This is enough for a few dozen grounded answer calls and lets you validate response structure, citation format, and rate-limit behavior.

---

Explore plans on [Geekflare Pricing](https://geekflare.com/pricing/), read the [Search API Docs](https://docs.geekflare.com/guides/search.md), or test the [Geekflare Search API](https://geekflare.com/search/) on the card-free Free tier (500 credits/month).
