Nice — I’ll give you a complete, **responsible**, ready-to-run Python scraper that:

- Uses _Google dork-style queries_ (search operators) to find candidate pages **via the Google Custom Search API** (safer and legal than scraping google.com directly),
    
- Fetches each target page while **respecting robots.txt**,
    
- Extracts a _particular type of thing_ from each page using a **CSS selector** or **regex** (you supply which), and
    
- Saves the results to **JSONL** (and optional CSV) ready for ML training.
    

**Why use the Custom Search API instead of scraping google.com?**  
Scraping google.com/serps directly often violates Google’s Terms of Service and triggers rate-limits/CAPTCHAs. The Custom Search JSON API is a proper programmatic way to run dork-like queries (you still supply the operators in the query string). If you prefer a paid alternative, SerpAPI is another option.

---

## What you need before running

1. **Google Custom Search JSON API key** and a **Custom Search Engine (CSE) ID** (set CSE to search the whole web). See Google docs for how to create these.
    
2. Python 3.9+ and these packages:
    

```bash
pip install requests beautifulsoup4 pandas
```

---

## Full script — parametric and ready-to-run

```python
#!/usr/bin/env python3
"""
google_dork_scraper.py

- Uses Google Custom Search JSON API to find URLs using dork-like queries.
- Respects robots.txt.
- Fetches pages, extracts target content using a CSS selector or regex.
- Saves output to JSONL (one JSON object per line) and optionally CSV.

Usage:
    python google_dork_scraper.py --api_key YOURKEY --cse_id YOURCSE --queries queries.txt \
        --selector ".product-description" --out results.jsonl --max_results 200
"""

import argparse
import json
import time
import random
import logging
import csv
import re
import hashlib
from urllib.parse import urlparse
from urllib import robotparser

import requests
from bs4 import BeautifulSoup

# --------------------------
# Configuration & helpers
# --------------------------
USER_AGENT = "Mozilla/5.0 (compatible; DataScraper/1.0; +https://example.com/bot)"
SESSION = requests.Session()
SESSION.headers.update({"User-Agent": USER_AGENT, "Accept-Language": "en-US,en;q=0.9"})

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")

def get_search_results(api_key, cse_id, query, start=1):
    """Call Google Custom Search JSON API. start is 1-indexed (1,11,21...)."""
    url = "https://www.googleapis.com/customsearch/v1"
    params = {
        "key": api_key,
        "cx": cse_id,
        "q": query,
        "start": start,
    }
    resp = SESSION.get(url, params=params, timeout=20)
    resp.raise_for_status()
    return resp.json()

def allowed_by_robots(url):
    parsed = urlparse(url)
    robots_url = f"{parsed.scheme}://{parsed.netloc}/robots.txt"
    rp = robotparser.RobotFileParser()
    try:
        rp.set_url(robots_url)
        rp.read()
        return rp.can_fetch(USER_AGENT, url)
    except Exception:
        # If robots.txt can't be read, be conservative and allow (alternatively deny)
        logging.warning(f"Could not read robots.txt at {robots_url}; proceeding cautiously.")
        return True

def fetch_page(url, timeout=20):
    try:
        r = SESSION.get(url, timeout=timeout)
        r.raise_for_status()
        return r.text, r.headers.get("Content-Type", "")
    except Exception as e:
        logging.warning(f"Failed to fetch {url}: {e}")
        return None, None

def extract_from_html(html, method, pattern):
    """method: 'css' or 'regex'. pattern: CSS selector or regex string."""
    if html is None:
        return None
    if method == "css":
        soup = BeautifulSoup(html, "html.parser")
        nodes = soup.select(pattern)
        texts = [n.get_text(separator=" ", strip=True) for n in nodes]
        return texts
    elif method == "regex":
        matches = re.findall(pattern, html, flags=re.DOTALL)
        # If groups used in regex, prefer groups
        if matches and isinstance(matches[0], tuple):
            matches = [" ".join(m).strip() for m in matches]
        return matches
    else:
        raise ValueError("Unknown extraction method")

def save_jsonl(path, records, append=True):
    mode = "a" if append else "w"
    with open(path, mode, encoding="utf-8") as f:
        for r in records:
            f.write(json.dumps(r, ensure_ascii=False) + "\n")

def save_csv(path, records, fieldnames):
    with open(path, "w", encoding="utf-8", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        for r in records:
            writer.writerow({k: (json.dumps(r.get(k, ""), ensure_ascii=False) if isinstance(r.get(k, ""), (list, dict)) else r.get(k, "")) for k in fieldnames})

# --------------------------
# Main pipeline
# --------------------------
def run(args):
    api_key = args.api_key
    cse_id = args.cse_id
    queries = []
    # read queries from file or single provided query
    if args.queries_file:
        with open(args.queries_file, encoding="utf-8") as qf:
            queries = [line.strip() for line in qf if line.strip() and not line.strip().startswith("#")]
    else:
        queries = [args.query]

    seen_urls = set()
    out_records = []
    total_found = 0

    for q in queries:
        logging.info(f"Searching: {q}")
        start = 1
        while total_found < args.max_results:
            try:
                results = get_search_results(api_key, cse_id, q, start=start)
            except requests.HTTPError as e:
                logging.error(f"Search API error for query '{q}' start={start}: {e}")
                break
            items = results.get("items", [])
            if not items:
                logging.info("No more items returned by API.")
                break

            for item in items:
                url = item.get("link")
                if not url or url in seen_urls:
                    continue
                seen_urls.add(url)

                if not allowed_by_robots(url):
                    logging.info(f"Blocked by robots.txt: {url}")
                    continue

                # polite rate limiting
                time.sleep(random.uniform(args.min_delay, args.max_delay))

                html, content_type = fetch_page(url)
                if html is None:
                    continue

                extracted = extract_from_html(html, args.extract_method, args.pattern) if args.pattern else []

                record = {
                    "url": url,
                    "search_query": q,
                    "title": item.get("title"),
                    "snippet": item.get("snippet"),
                    "content_type": content_type,
                    "extracted": extracted,
                    "fetched_at": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
                    "id": hashlib.sha256(url.encode("utf-8")).hexdigest()[:16],
                }

                out_records.append(record)
                total_found += 1

                # Save periodically
                if len(out_records) >= args.flush_every:
                    save_jsonl(args.out, out_records, append=True)
                    logging.info(f"Flushed {len(out_records)} records to {args.out}")
                    out_records = []

                if total_found >= args.max_results:
                    break

            # prepare next page (Google API returns 10 items per page typically)
            start += 10
            # Google Custom Search allows start up to 100 (10 pages) for free; handle gracefully
            if start > 100:
                logging.info("Reached maximum paging for this query (start>100).")
                break

        # end of query loop

    # final flush
    if out_records:
        save_jsonl(args.out, out_records, append=True)
        logging.info(f"Saved final {len(out_records)} records to {args.out}")

    # optionally produce a CSV with a subset of fields
    if args.out_csv:
        logging.info(f"Writing CSV to {args.out_csv}")
        # read jsonl to gather lines (simple approach)
        records_for_csv = []
        with open(args.out, encoding="utf-8") as f:
            for line in f:
                records_for_csv.append(json.loads(line))
        fieldnames = ["id", "url", "title", "search_query", "snippet", "extracted", "fetched_at"]
        save_csv(args.out_csv, records_for_csv, fieldnames)
        logging.info("CSV written.")

    logging.info("Done.")

# --------------------------
# CLI
# --------------------------
def parse_args():
    p = argparse.ArgumentParser()
    p.add_argument("--api_key", required=True, help="Google API key")
    p.add_argument("--cse_id", required=True, help="Google Custom Search Engine ID")
    q = p.add_mutually_exclusive_group(required=True)
    q.add_argument("--queries_file", help="File with one dork query per line (can use google operators)")
    q.add_argument("--query", help="Single query/dork string")
    p.add_argument("--pattern", required=False, help="CSS selector or regex to extract target content (if omitted, scraper just collects urls and metadata)")
    p.add_argument("--extract_method", choices=["css", "regex"], default="css", help="Extraction method: css or regex")
    p.add_argument("--out", default="results.jsonl", help="JSONL output file")
    p.add_argument("--out_csv", default=None, help="Optional CSV output file")
    p.add_argument("--max_results", type=int, default=200, help="Maximum number of pages to fetch overall")
    p.add_argument("--min_delay", type=float, default=1.0, help="Min delay between requests (s)")
    p.add_argument("--max_delay", type=float, default=3.0, help="Max delay between requests (s)")
    p.add_argument("--flush_every", type=int, default=10, help="Flush to disk every N records")
    return p.parse_args()

if __name__ == "__main__":
    args = parse_args()
    run(args)
```

---

## How to use it (examples)

1. Put your dork queries in `queries.txt`:
    

```
site:example.com intitle:"product" "specifications"
inurl:recipes "ingredients" "Serves"
filetype:pdf "financial statement" "2024"
```

2. Run:
    

```bash
python google_dork_scraper.py \
  --api_key YOUR_GOOGLE_API_KEY \
  --cse_id YOUR_CSE_ID \
  --queries_file queries.txt \
  --pattern ".product-description" \
  --extract_method css \
  --out product_desc.jsonl \
  --out_csv product_desc.csv \
  --max_results 500
```

Or using regex extraction:

```bash
--pattern "<div class=\"ingredients\">(.*?)</div>" --extract_method regex
```

---

## Notes, cautions & best practices (important)

- **Respect robots.txt**: this script checks robots.txt; do not bypass it.
    
- **Rate limits & politeness**: keep delays high (e.g., 1–3s or more), and throttle if you see CAPTCHAs or HTTP 429 responses.
    
- **Terms of service**: ensure target sites permit crawling/republishing. For large scale datasets, contact site owners or use official APIs.
    
- **Labeling for ML**: scraped data is often noisy. Consider manual verification or semi-automatic labelling, and store provenance (url, timestamp).
    
- **Legal & privacy**: avoid scraping personal data or copyrighted content where you don't have rights to use it. If data contains PII, follow privacy laws (e.g., GDPR).
    
- **Google Custom Search limits**: free quota is small; for heavy usage buy a paid plan or use a commercial SERP API (SerpAPI, BrightData, etc.).
    

---

## Improvements & extensions you might want

- Use **Playwright**/Selenium for pages that heavily rely on JavaScript.
    
- Add **deduplication** on extracted text (fingerprint hashes).
    
- Use **concurrent fetching** with careful concurrency limits and per-domain rate limits.
    
- Add **content-language** filtering and encoding normalization.
    
- Add **labeling interface** (small web app) to quickly validate samples for training.
    

---

If you want, I can:

- adapt the script to target a specific _type_ (tell me the type — e.g., "product spec paragraphs", "ingredient lists", "legal clauses") and I’ll pick likely CSS selectors and show extraction examples; **or**
    
- provide a version that uses **Playwright** for JS sites, or a small **labeling** CSV/JSONL schema for ML training.
    

Which would you like next?
