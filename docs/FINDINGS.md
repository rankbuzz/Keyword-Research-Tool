# GoTrends deep findings

Last live probe: **11 Sep 2026** — **WORKING**.

## Status

Both core endpoints return HTTP 200 with full metrics.

Upstream Google Ads version is **v25** (seen in validation enum paths). Previously broken on deprecated **v21** (mid‑Aug 2026).

Public SPA JS hash unchanged (`app-main.fb3fbd2f9206cc9e0b00.js`) — `/kw/*` proxy backend was upgraded, not the Metabase frontend.

## Endpoints

| Path | Status | Use |
|------|--------|-----|
| `POST /kw/gethistorymetric` | OK | Volume, CPC, competition, bids, monthly trend |
| `POST /kw/keywordidea` | OK | Ideas via `keywordSeed` or `urlSeed` + same metrics + pagination |

### Not available (404 / empty)

`generateadgroupthemes`, `generatekeywordforecastmetrics`, `suggest`, `related`, `trends`, `metrics`, `volume`, `ideas`, fake `/v2x/kw/...` paths, etc.

### Not on GoTrends

Domain ranking keywords, backlinks, DA/PA.

## Live samples (US `2840` / English `1000`)

| Call | Result |
|------|--------|
| `gethistorymetric` `bags` | vol **165,000**, CPC micros ~7.1M, competition HIGH, **12** months |
| `gethistorymetric` + `yearMonthRange` | up to **24** months |
| `gethistorymetric` geo `[]` | worldwide vol **1,220,000** |
| `keywordidea` keywordSeed `bags` | **1780** total + `nextPageToken` |
| `keywordidea` urlSeed `nike.com` | **1613** total |

## Metric keys

`competition`, `competitionIndex`, `avgMonthlySearches`, `averageCpcMicros`, `lowTopOfPageBidMicros`, `highTopOfPageBidMicros`, `monthlySearchVolumes[]`

Ideas may include `keywordAnnotations` (often empty).

## Local / geo levels

`geoTargetConstants` accepts any Google Ads criterion ID:

- Country (e.g. `2840` US, `2586` Pakistan)
- State / province / region
- City (e.g. Los Angeles `1023191`)

Same keyword returns **different volumes** per geo — confirmed live with `plumber`.

## Request conventions

- Auth: none  
- Language: `languageConstants/{id}`  
- Network: `GOOGLE_SEARCH`  
- Ideas: `pageSize` (we use up to 200) + `pageToken`  
- CPC: divide micros by `1_000_000` for USD  

## Outage lesson (Aug 2026)

```
Version v21 is deprecated. Requests to this version will be blocked.
errorCode.requestError = UNSUPPORTED_VERSION
```

Client cannot pick Ads API version — only GoTrends proxy can. Headers / body version fields did nothing.
