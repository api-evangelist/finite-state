---
name: Browse and retrieve Finite State content
description: List Finite State blog posts, resources, videos, events, podcasts or press articles with pagination and filters, then fetch a single item in full by its slug.
api: a2a/finite-state-agent-card.json
endpoint: https://finitestate.io/api/a2a
operations:
  - content.list
  - content.get
auth: none
generated: '2026-08-04'
method: generated
---

# Browse and retrieve Finite State content

Finite State exposes a public, anonymous, read-only A2A API at
`https://finitestate.io/api/a2a`. It speaks JSON-RPC 2.0 over `POST` only —
a `GET` returns `405`. No credentials, tokens or headers beyond
`Content-Type: application/json` are required.

Only four methods are allowlisted: `content.list`, `content.get`,
`content.search`, `content.metadata`. Anything else returns JSON-RPC error
`-32601` with the allowlist in `error.data`.

## Step 1 — list items of one content type (`content.list`)

Pick exactly one `type` from `post`, `resource`, `video`, `event`, `podcast`,
`pressArticle`.

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "content.list",
  "params": {
    "type": "post",
    "limit": 10,
    "offset": 0,
    "sort": "newest"
  }
}
```

Parameters, as published in the agent card:

| Param | Meaning |
|---|---|
| `type` | `post` \| `resource` \| `video` \| `event` \| `podcast` \| `pressArticle` |
| `limit` | max items to return, 1–100, default 10 |
| `offset` | items to skip, default 0 |
| `category` | optional category filter |
| `sort` | `newest` \| `oldest` \| `a-z` \| `z-a` |

The result envelope is `{items, total, limit, offset, hasMore}`. Page by
incrementing `offset` by `limit` while `hasMore` is `true` — do not assume a
cursor, there is none.

Every item carries `_id`, `slug`, `title`, `url`, `image`, `imageAlt` and
`tags`. Type-specific fields differ: `post` adds `author`, `authorAvatar`,
`categories`, `date`, `excerpt`; `resource` adds `gated` and `resourceType`;
`video` adds `duration`, `views`, `quality`, `featured`; `event` adds `start`,
`end`, `venue`, `city`, `country`, `registrationUrl` and timezone fields;
`pressArticle` adds `externalUrl` and `status`.

Note that `podcast` is a declared type with zero items — treat an empty
`items` array as a valid result, not an error.

## Step 2 — fetch one item in full (`content.get`)

Take the `slug` from the listing and pair it with the same `type`. The slug
alone is not a key; the `(type, slug)` pair is.

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "content.get",
  "params": { "type": "post", "slug": "why-appsec-sca-fails-for-firmware" }
}
```

## Error handling

Errors come back as JSON-RPC error objects with **HTTP 200** — check
`response.error`, never the status code alone.

| Code | Meaning | What to do |
|---|---|---|
| `-32601` | Method not found | Use one of the four allowlisted methods; `error.data` lists them |

Responses are sent with `cache-control: no-store`, so do not cache them
locally. `access-control-allow-origin` is `*`, so the endpoint is callable
from a browser.

## Do not

- Do not attempt writes. The provider states in its own `llms.txt` that it does
  not "provide write-capable public A2A methods; published A2A methods are
  read-only."
- Do not use this endpoint for platform data (SBOMs, findings, components).
  That is the separate, token-authenticated Finite State Platform API at
  `https://app.finitestate.io/api/public/v0`.
