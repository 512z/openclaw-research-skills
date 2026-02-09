---
name: seeking_alpha
description: Access Seeking Alpha data for stock research, quant ratings, analyst articles, dividend history, and market insights via authenticated API calls.
---

# Seeking Alpha Skill

Access Seeking Alpha (SA) data using authenticated session cookies. This skill provides programmatic access to SA's API and HTML pages for stock research, article reading, quant ratings, and more.

## Prerequisites

- **Credentials**: `sa_credentials.json` (in this directory) contains session cookies exported from a premium SA account.
- **Python packages**: `requests` (standard; no extra install needed).
- **Cookie refresh**: Session cookies expire. If requests return 401/403 or redirect to login, re-export cookies from the browser and update `sa_credentials.json`.

## Quick Start

```python
import json, requests, re

# Load cookies
SKILL_DIR = "/Users/henryzha/Desktop/alpha-1/sa/skills/seeking_alpha"
with open(f"{SKILL_DIR}/sa_credentials.json") as f:
    creds = json.load(f)

cookie_jar = {c["name"]: c["value"] for c in creds["cookies"]}

HEADERS = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36",
    "Accept": "application/json",
}

BASE = "https://seekingalpha.com"

def sa_get(path, params=None):
    """Make authenticated GET request to SA API."""
    return requests.get(f"{BASE}{path}", headers=HEADERS, cookies=cookie_jar, params=params, timeout=15)
```

## Working API Endpoints

All endpoints are `GET` requests to `https://seekingalpha.com/api/v3/...`.

### 1. Symbol Info

```
GET /api/v3/symbols/{TICKER}
```
Returns: company name, exchange, equity type, followers count, etc.

| Param | Example | Description |
|-------|---------|-------------|
| `include` | `peers` | Include related data |

**Response fields**: `companyName`, `exchange`, `currency`, `equityType`, `followersCount`, `tradingViewSlug`, `divYieldType`, etc.

### 2. Tickers (Batch Lookup)

```
GET /api/v3/tickers?filter[slugs]=AAPL,MSFT,GOOGL
```
Returns ticker metadata for multiple symbols at once. Same fields as Symbol Info.

### 3. Analysis Articles

```
GET /api/v3/symbols/{TICKER}/analysis
```
Lists analyst opinion articles for a ticker.

| Param | Example | Description |
|-------|---------|-------------|
| `page[size]` | `20` | Results per page (max ~40) |
| `page[number]` | `2` | Page number |
| `filter[since]` | `0` | Unix timestamp, articles after |
| `filter[until]` | `0` | Unix timestamp, articles before |
| `include` | `author,primaryTickers,secondaryTickers` | Related data |

**Response fields per article**: `id`, `title`, `publishOn`, `commentCount`, `isPaywalled`, `isLockedPro`, `structuredInsights`, `themes`.

### 4. News

```
GET /api/v3/symbols/{TICKER}/news
```
Same params as Analysis. Returns news items.

### 5. Transcripts (Earnings Calls)

```
GET /api/v3/symbols/{TICKER}/transcripts
```
Same params as Analysis. Returns earnings call transcripts.

### 6. Press Releases

```
GET /api/v3/symbols/{TICKER}/press-releases
```
Same params as Analysis.

### 7. Related Analysis

```
GET /api/v3/symbols/{TICKER}/related-analysis
```
Articles related to the ticker from other authors.

### 8. Quant Ratings

```
GET /api/v3/symbols/{TICKER}/ratings
```
Returns SA's quant rating grades - **this is one of the most valuable endpoints**.

**Response `ratings` object fields**:
| Field | Type | Description |
|-------|------|-------------|
| `quantRating` | float | Overall quant rating (1-5 scale, 5=Strong Buy) |
| `authorsRating` | float | SA authors consensus rating |
| `sellSideRating` | float | Wall Street analysts consensus |
| `valueGrade` | int | Valuation grade (1=A+, 13=F) |
| `growthGrade` | int | Growth grade |
| `profitabilityGrade` | int | Profitability grade |
| `momentumGrade` | int | Momentum grade |
| `epsRevisionsGrade` | int | EPS revisions grade |
| `dividendYieldGrade` | int | Dividend yield grade |
| `divSafetyCategoryGrade` | int | Dividend safety grade |
| `divGrowthCategoryGrade` | int | Dividend growth grade |
| `divConsistencyCategoryGrade` | int | Dividend consistency grade |

**Grade mapping**: 1=A+, 2=A, 3=A-, 4=B+, 5=B, 6=B-, 7=C+, 8=C, 9=C-, 10=D+, 11=D, 12=D-, 13=F

**Author rating counts**: `authorsRatingBuyCount`, `authorsRatingHoldCount`, `authorsRatingSellCount`, `authorsRatingStrongBuyCount`, `authorsRatingStrongSellCount` (also available with `30Day` and `90Day` suffixes).

### 9. Dividend History

```
GET /api/v3/symbols/{TICKER}/dividend_history
```
Returns full dividend payment history.

| Param | Example | Description |
|-------|---------|-------------|
| `page[size]` | `20` | Results per page |

**Response fields**: `amount`, `adjusted_amount`, `ex_date`, `pay_date`, `record_date`, `declare_date`, `freq`, `type`, `year`, `split_adj_factor`.

### 10. Article Detail (Preview)

```
GET /api/v3/articles/{ARTICLE_ID}
```
Returns article metadata and a **content preview** (~300 words of HTML). Full content requires browser rendering.

**Response fields**: `title`, `publishOn`, `content` (HTML preview), `summary` (bullet points array), `disclosure`, `commentCount`, `likesCount`, `isPaywalled`, `isLockedPro`, and more.

**Important**: The `summary` field is an array of bullet-point strings summarizing the article - very useful for quick analysis without needing the full text.

### 11. Market Status

```
GET /api/v3/market_open
```
Returns: `marketOpen` (bool), `nextMarketOpen`, `nextMarketClose` (timestamps).

## Extracting Full Article Text (HTML Scraping)

The API only returns article previews. For full text, scrape the article HTML page:

```python
def get_full_article(article_id):
    """Get full article text by scraping the HTML page."""
    h = {**HEADERS, "Accept": "text/html"}
    r = requests.get(f"{BASE}/article/{article_id}", headers=h, cookies=cookie_jar, timeout=15)

    # Extract from SSR_DATA (server-side rendered preview)
    m = re.search(r'window\.SSR_DATA\s*=\s*({.*?});\s*</script>', r.text, re.DOTALL)
    if m:
        ssr = json.loads(m.group(1))
        attrs = ssr.get("article", {}).get("response", {}).get("data", {}).get("attributes", {})
        content_html = attrs.get("content", "")
        summary = attrs.get("summary", [])
        title = attrs.get("title", "")
        # Strip HTML tags
        text = re.sub(r'<[^>]+>', '', content_html).strip()
        return {"title": title, "text": text, "summary": summary}
    return None
```

**Note**: SSR_DATA contains a preview (~300 words). Full article body is hydrated client-side by React. For complete text, you would need a headless browser (Playwright/Selenium). For most research purposes, the `summary` array + preview text is sufficient.

## Example Workflows

### Get Quant Ratings for Multiple Stocks

```python
tickers = ["AAPL", "MSFT", "GOOGL", "AMZN", "NVDA"]
for t in tickers:
    r = sa_get(f"/api/v3/symbols/{t}/ratings")
    if r.status_code == 200:
        data = r.json().get("data", [])
        if data:
            ratings = data[0]["attributes"]["ratings"]
            print(f"{t}: Quant={ratings['quantRating']:.2f}, "
                  f"Value={ratings['valueGrade']}, Growth={ratings['growthGrade']}, "
                  f"Profit={ratings['profitabilityGrade']}, Mom={ratings['momentumGrade']}")
```

### Get Latest Analysis with Summaries

```python
r = sa_get("/api/v3/symbols/AAPL/analysis", {"page[size]": "5", "include": "author"})
for art in r.json().get("data", []):
    attrs = art["attributes"]
    print(f"[{art['id']}] {attrs['title']}")
    print(f"  Published: {attrs['publishOn']}")

    # Get summary
    detail = sa_get(f"/api/v3/articles/{art['id']}")
    summary = detail.json()["data"]["attributes"].get("summary", [])
    for s in summary:
        print(f"  - {s}")
    print()
```

### Screen Stocks by Quant Rating

```python
# SA doesn't have a public screener API, but you can batch-check ratings
import time

sp500_sample = ["AAPL", "MSFT", "GOOGL", "AMZN", "NVDA", "META", "TSLA", "BRK.B", "JPM", "V"]
strong_buys = []

for t in sp500_sample:
    r = sa_get(f"/api/v3/symbols/{t}/ratings")
    if r.status_code == 200:
        data = r.json().get("data", [])
        if data:
            qr = data[0]["attributes"]["ratings"]["quantRating"]
            if qr >= 4.5:
                strong_buys.append((t, qr))
    time.sleep(0.5)  # Be polite

print("Strong Buy stocks:", strong_buys)
```

## Rate Limiting & Best Practices

- **No official rate limit** documented, but be respectful: ~1-2 requests/second.
- Use `time.sleep(0.5)` between requests when iterating over many tickers.
- **Batch where possible**: Use `/api/v3/tickers?filter[slugs]=AAPL,MSFT,...` for basic info.
- **Cache responses**: Ratings and article lists don't change minute-to-minute.
- **Cookie expiry**: The `user_remember_token` cookie is the primary session cookie. If it expires (~1 year), re-login in browser and re-export.

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| 401/403 responses | Cookies expired | Re-export from browser, update `sa_credentials.json` |
| 404 on known endpoint | API version changed | SA periodically updates API; check current page network requests |
| Empty `content` in articles | Normal behavior | API returns preview only; use `summary` field or HTML scraping |
| 429 Too Many Requests | Rate limited | Add `time.sleep(1)` between requests |
| PerimeterX block (403 with captcha) | Bot detection | Ensure `_px3` and `_pxvid` cookies are included |

## Files in This Skill

```
seeking_alpha/
  SKILL.md              # This file
  sa_credentials.json   # Auth cookies (essential-only, cleaned from full export)
  reference/
    api_endpoints.json  # Machine-readable endpoint definitions
    grade_mapping.json  # Grade number -> letter mapping
    example_responses/  # Sample API responses for reference
```
