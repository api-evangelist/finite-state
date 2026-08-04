---
name: Search Finite State content
description: Run a keyword search across Finite State's blog posts, resources, videos, events and press articles, optionally narrowing to specific content types, then resolve hits to full items.
api: a2a/finite-state-agent-card.json
endpoint: https://finitestate.io/api/a2a
operations:
  - content.search
  - content.get
auth: none
generated: '2026-08-04'
method: generated
---

# Search Finite State content

Use `content.search` on the anonymous A2A endpoint
`https://finitestate.io/api/a2a` when you have a topic rather than a known
content type. The agent card describes it as delegating to the site search.

## Step 1 — search (`content.search`)

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "content.search",
  "params": {
    "query": "cyber resilience act SBOM",
    "limit": 20,
    "offset": 0
  }
}
```

Parameters, as published in the agent card:

| Param | Meaning |
|---|---|
| `query` | search query string |
| `limit` | max results, 1–50, default 20 — note this ceiling is **50**, not the 100 that `content.list` allows |
| `offset` | offset for pagination |
| `types` | optional list of content types to restrict the search to |

Narrow with `types` when the question is format-specific — for example
`["resource", "video"]` for datasheets and recorded sessions, or
`["pressArticle"]` for news coverage.

## Step 2 — resolve a hit (`content.get`)

Search results carry the same `_id` / `slug` / `title` / `url` shape as
listings. To read one in full, call `content.get` with the item's `type` and
`slug` together.

## Step 3 — fall back to browsing

If a search returns nothing useful, switch to `content.list` with an explicit
`type` and `category` filter and sort by `newest`. Counts at capture time were
445 posts, 114 videos, 81 resources, 77 press articles, 45 events and 0
podcasts, so an empty result on a narrow query is common and expected.

## Rules

- Paginate on `offset`; the envelope returns `{items, total, limit, offset, hasMore}`.
- Errors arrive as JSON-RPC error objects under **HTTP 200**; inspect
  `response.error`. `-32601` means you called a method outside the allowlist.
- The endpoint is `POST`-only and unauthenticated. Do not send an API token to
  it — the platform token belongs only to
  `https://app.finitestate.io/api/public/v0`.
- No rate-limit headers are published. Be conservative and serialize requests.
