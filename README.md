# Keyword Research Tool — API Notes

Private research notes: how we call upstream APIs and what data comes back.

> Scope of this repo (for now): **API calling + findings only**. No app code yet.

| Source | What we get | Docs |
|--------|-------------|------|
| **GoTrends** | Keyword volume, CPC, competition, trend, keyword ideas | This README (below) · [FINDINGS](docs/FINDINGS.md) |
| **Keywords Everywhere** | DA, RD, backlinks count, traffic (ETV), keyword lists, backlink lists | [KEYWORDS-EVERYWHERE](docs/KEYWORDS-EVERYWHERE.md) |

**DR** is **not** from KE — Ahrefs separate API (documented in KE doc).  
**Keyword volume / ideas** are **GoTrends**, not KE.

---

# Part 1 — GoTrends

## Base URL

```
https://insight.gotrends.app
```

| Item | Value |
|------|--------|
| Auth | **None** (no API key / Bearer) |
| Method | `POST` + JSON body |
| Content-Type | `application/json` |
| Upstream | Google Ads Keyword Plan (currently **v25**) |

---

## Endpoints we use (only 2 exist)

### 1) Keyword metrics (volume / CPC / competition / trend)

```
POST /kw/gethistorymetric
```

### 2) Keyword ideas (from seed keyword **or** URL/domain)

```
POST /kw/keywordidea
```

Probed and **not** available: forecast, themes, suggest, related, trends, `/v2x/kw/...`, etc.

**Not on GoTrends:** domain ranking keywords, backlinks, DA/PA.

---

## How we call it

### Headers

```http
Content-Type: application/json
Accept: application/json
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
```

### Geo / language

- `geoTargetConstants`: Google Ads location IDs  
  - `[]` = Worldwide  
  - `["geoTargetConstants/2840"]` = United States (country)  
  - Same field works for **state / province / region / city** (any valid Google Ads geo ID)
- `language`: `languageConstants/1000` (English)
- `keywordPlanNetwork`: `GOOGLE_SEARCH`

### Example — history metrics

```json
{
  "keywords": ["bags"],
  "includeAdultKeywords": true,
  "geoTargetConstants": ["geoTargetConstants/2840"],
  "language": "languageConstants/1000",
  "keywordPlanNetwork": "GOOGLE_SEARCH",
  "aggregateMetrics": { "aggregateMetricTypes": ["DEVICE"] },
  "historicalMetricsOptions": { "includeAverageCpc": true }
}
```

Optional date range (up to ~24 months):

```json
"historicalMetricsOptions": {
  "includeAverageCpc": true,
  "yearMonthRange": {
    "start": { "year": "2024", "month": "JANUARY" },
    "end": { "year": "2025", "month": "DECEMBER" }
  }
}
```

### Example — keyword ideas (seed keyword)

```json
{
  "geoTargetConstants": ["geoTargetConstants/2840"],
  "language": "languageConstants/1000",
  "keywordPlanNetwork": "GOOGLE_SEARCH",
  "includeAdultKeywords": true,
  "pageSize": 100,
  "historicalMetricsOptions": { "includeAverageCpc": true },
  "keywordSeed": { "keywords": ["bags"] }
}
```

Pagination: send `pageToken` from previous response’s `nextPageToken`.

### Example — ideas from URL / domain

```json
{
  "geoTargetConstants": ["geoTargetConstants/2840"],
  "language": "languageConstants/1000",
  "keywordPlanNetwork": "GOOGLE_SEARCH",
  "includeAdultKeywords": true,
  "pageSize": 100,
  "historicalMetricsOptions": { "includeAverageCpc": true },
  "urlSeed": { "url": "https://www.nike.com" }
}
```

### curl

```bash
curl -X POST "https://insight.gotrends.app/kw/gethistorymetric" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d "{\"keywords\":[\"bags\"],\"includeAdultKeywords\":true,\"geoTargetConstants\":[\"geoTargetConstants/2840\"],\"language\":\"languageConstants/1000\",\"keywordPlanNetwork\":\"GOOGLE_SEARCH\",\"historicalMetricsOptions\":{\"includeAverageCpc\":true}}"
```

---

## What data we get

### From `/kw/gethistorymetric`

Each result row:

| Field | Meaning |
|-------|---------|
| `text` | Keyword |
| `keywordMetrics.avgMonthlySearches` | Avg monthly search volume |
| `keywordMetrics.averageCpcMicros` | Avg CPC in micros (`/ 1_000_000` → USD) |
| `keywordMetrics.competition` | `LOW` / `MEDIUM` / `HIGH` |
| `keywordMetrics.competitionIndex` | 0–100 |
| `keywordMetrics.lowTopOfPageBidMicros` | Low top-of-page bid |
| `keywordMetrics.highTopOfPageBidMicros` | High top-of-page bid |
| `keywordMetrics.monthlySearchVolumes[]` | Trend: `{ year, month, monthlySearches }` |

Default trend ≈ **12 months**; with `yearMonthRange` up to ~**24 months**.

### From `/kw/keywordidea`

Top-level:

| Field | Meaning |
|-------|---------|
| `results[]` | Idea rows |
| `totalSize` | Total ideas available |
| `nextPageToken` | Next page cursor (or absent when done) |

Each idea row:

| Field | Meaning |
|-------|---------|
| `text` | Suggested keyword |
| `keywordIdeaMetrics.*` | Same metric fields as history |
| `keywordAnnotations` | Present; often `[]` |

---

## Local SEO (city / state / region)

**Yes — supported.** Same endpoints; only change the geo ID.

Live check (keyword `plumber`):

| Level | Example | Volume differed |
|-------|---------|-----------------|
| Worldwide | `geoTargetConstants: []` | Yes |
| Country | US `2840` | Yes |
| State / region | Google Ads state ID | Yes |
| City | e.g. Los Angeles `1023191` | Yes |

Ideas also work with a city geo.

---

## Live samples (Sep 2026, US / English)

| Call | Result |
|------|--------|
| History `bags` | vol ~165K, CPC, HIGH, 12‑mo trend |
| History worldwide `bags` | vol ~1.22M |
| Ideas keyword `bags` | totalSize ~1780 |
| Ideas URL `nike.com` | totalSize ~1613 |

More detail: [docs/FINDINGS.md](docs/FINDINGS.md) · example JSON: [examples/](examples/)

---

## Reliability note

Mid‑Aug 2026 the proxy briefly failed with Google Ads **v21 deprecated**. As of **11 Sep 2026** it works again on **v25**. If stats go empty, check for `UNSUPPORTED_VERSION` / deprecated errors from the same endpoints.

---

# Part 2 — Keywords Everywhere (domain metrics)

Full detail: **[docs/KEYWORDS-EVERYWHERE.md](docs/KEYWORDS-EVERYWHERE.md)**

Quick map:

| UI | KE endpoint | Field |
|----|-------------|-------|
| DA | `get-domain-link-metrics` | `moz_domain_authority` |
| RD | same | `moz_root_domains_to_subdomain` |
| Backlinks (count) | same | `moz_external_pages_to_subdomain` |
| Traffic | `get-domain-metrics` / `get-url-metrics` | `etv` |
| Keyword list | `get-domain-keywords` / `get-url-keywords` | `data[]` |
| Backlink list | `get-domain-backlinks` / `get-page-backlinks` | `backlinks[]` |

Auth: `api_key` in JSON body. Bases: `data.keywordseverywhere.com` + `links.keywordseverywhere.com`.
