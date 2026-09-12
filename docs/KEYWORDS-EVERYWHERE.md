# Keywords Everywhere — domain / URL metrics

How we call **Keywords Everywhere (KE)** for domain metrics (DA, RD, backlinks count, traffic, keyword lists, backlink lists) and what comes back.

> Related: [GoTrends keyword volume/ideas](../README.md) · [GoTrends findings](FINDINGS.md)

Auth: **`api_key` in JSON body** (not a Bearer header).  
Env: `KE_API_KEY` (required for live calls).

---

## Base URLs

| Host | Default |
|------|---------|
| Data | `https://data.keywordseverywhere.com/service` |
| Links | `https://links.keywordseverywhere.com/service` |

---

## What maps to UI labels

| UI label | Source | KE field |
|----------|--------|----------|
| **DA** | `get-domain-link-metrics` | `data.moz_domain_authority` |
| **RD** (referring domains) | `get-domain-link-metrics` | `data.moz_root_domains_to_subdomain` |
| **Backlinks** (count) | `get-domain-link-metrics` | `data.moz_external_pages_to_subdomain` |
| **Spam** | `get-domain-link-metrics` | `data.moz_spam_score` |
| **PR** | `get-domain-link-metrics` | `data.page_rank` |
| **Traffic** (domain) | `get-domain-metrics` | `data.etv` |
| **Traffic** (URL) | `get-url-metrics` | `data.etv` |
| **Keywords count** | domain/url metrics | `data.total_keywords` |
| **Keyword list** | `get-domain-keywords` / `get-url-keywords` | `data[]` rows |
| **Backlink list** | `get-domain-backlinks` / `get-page-backlinks` | `backlinks[]` |
| **DR** | **Not KE** — Ahrefs free/public DR API | `domain_rating.domain_rating` |

---

## Endpoints we use (7)

### A) Domain link metrics — DA / RD / BL count / Spam / PR

```
POST https://data.keywordseverywhere.com/service/get-domain-link-metrics
Accept: application/x.seometrics.v4+json
Content-Type: text/plain;charset=UTF-8
```

```json
{
  "api_key": "YOUR_KE_KEY",
  "domains": ["example.com", "moz.com"]
}
```

Response shape (array):

```json
[
  {
    "domain": "example.com",
    "data": {
      "page_rank": 6,
      "moz_domain_authority": 45,
      "moz_spam_score": 3,
      "moz_root_domains_to_subdomain": 1200,
      "moz_external_pages_to_subdomain": 8500
    }
  }
]
```

### B) Domain traffic + keyword count

```
POST https://data.keywordseverywhere.com/service/get-domain-metrics
Accept: application/x.seometrics.v4+json
Content-Type: text/plain;charset=UTF-8
```

```json
{
  "api_key": "YOUR_KE_KEY",
  "domains": ["example.com"],
  "country": "us"
}
```

`data.etv` = traffic estimate · `data.total_keywords` = ranking keyword count.  
`country` = ISO-2 (`us`, `pk`, …) — **not** Google Ads geo IDs.

### C) URL traffic + keyword count

```
POST https://data.keywordseverywhere.com/service/get-url-metrics
```

```json
{
  "api_key": "YOUR_KE_KEY",
  "urls": ["https://example.com/page"],
  "country": "us"
}
```

Same `etv` / `total_keywords` under `data`.

### D) Domain keyword list (explore)

```
POST https://data.keywordseverywhere.com/service/get-domain-keywords
Accept: application/x.seometrics.v4+json
Content-Type: application/json
```

```json
{
  "api_key": "YOUR_KE_KEY",
  "domain": "example.com",
  "country": "us",
  "num": 5000
}
```

Response highlights: `domain`, `etv`, `total_keywords`, `data[]` with `{ keyword, position, etv }`.

### E) URL / page keyword list

```
POST .../get-url-keywords
```

```json
{
  "api_key": "YOUR_KE_KEY",
  "url": "https://example.com/page",
  "country": "us",
  "num": 5000
}
```

### F) Domain backlinks list

```
POST https://links.keywordseverywhere.com/service/get-domain-backlinks
Accept: application/x.seometrics.v3+json
Content-Type: application/json
```

```json
{
  "api_key": "YOUR_KE_KEY",
  "domain": "example.com",
  "num": 5000
}
```

Each `backlinks[]` row: `domain_source`, `url_source`, `domain_target`, `url_target`, `anchor_text`, `harmonic_centrality`, `last_found_date`.

### G) Page backlinks list

```
POST .../get-page-backlinks
```

```json
{
  "api_key": "YOUR_KE_KEY",
  "page": "https://example.com/page",
  "num": 5000
}
```

---

## How we call it (summary)

1. Strip `www.` for domain keys where needed.  
2. POST JSON with `api_key` + params.  
3. Use correct **Accept** version (`v4` data, `v3` links).  
4. Link/traffic batch endpoints use `Content-Type: text/plain;charset=UTF-8` in our client.  
5. Explore keyword/backlink list endpoints use `Content-Type: application/json`.  
6. No offset pagination for backlinks — one shot up to `num`.

### curl (link metrics)

```bash
curl -X POST "https://data.keywordseverywhere.com/service/get-domain-link-metrics" \
  -H "Accept: application/x.seometrics.v4+json" \
  -H "Content-Type: text/plain;charset=UTF-8" \
  -d "{\"api_key\":\"YOUR_KE_KEY\",\"domains\":[\"moz.com\"]}"
```

---

## DR (separate — Ahrefs, not KE)

```
GET https://api.ahrefs.com/v3/public/domain-rating-free?target=example.com
Authorization: Bearer <optional AHREFS_API_KEY>
```

Response: `domain_rating.domain_rating` → DR.  
In KwRank SERP flow DR is currently skipped (`null`) until re-enabled.

---

## Not from KE in our stack

| Metric | Note |
|--------|------|
| Moz **PA** | Not mapped / not used |
| Ahrefs DR | Separate API |
| Keyword volume / ideas | **GoTrends**, not KE |

---

## Example JSON files

- [../examples/ke-domain-link-metrics.json](../examples/ke-domain-link-metrics.json)
- [../examples/ke-domain-metrics.json](../examples/ke-domain-metrics.json)
- [../examples/ke-domain-keywords.json](../examples/ke-domain-keywords.json)
- [../examples/ke-domain-backlinks.json](../examples/ke-domain-backlinks.json)
