---
name: Discover the Finite State agent surface
description: Bootstrap against Finite State from zero — read the agent card, call content.metadata for canonical feed URLs and live content counts, and route to the right surface for platform data.
api: a2a/finite-state-agent-card.json
endpoint: https://finitestate.io/api/a2a
operations:
  - content.metadata
auth: none
generated: '2026-08-04'
method: generated
---

# Discover the Finite State agent surface

Start here when you know nothing about Finite State's machine-readable
surfaces. Everything in this skill is anonymous.

## Step 1 — read the agent card

```
GET https://finitestate.io/.well-known/agent-card.json
```

Returns `application/json`. The same document is also served at the pre-0.3
path `https://finitestate.io/.well-known/agent.json`. It declares
`protocolVersion` `0.3`, a JSONRPC interface at
`https://finitestate.io/api/a2a`, empty `securitySchemes` (the surface is
anonymous), and four skills.

Do **not** trust `/.well-known/*` on the other Finite State hosts. `docs.finitestate.io`
is a Docusaurus SPA and `finitestate.io` is a Next.js SPA; both answer `200`
with an HTML shell for unknown well-known paths. Verify the body parses as a
JSON object before believing a hit.

## Step 2 — call `content.metadata`

```json
{ "jsonrpc": "2.0", "id": 1, "method": "content.metadata", "params": {} }
```

This is the cheapest live orientation call. It returns `siteName`, `siteUrl`,
a `feeds` map, and a `contentTypes` array of `{type, count}`. Use the counts to
decide whether a content type is worth listing at all — `podcast` was 0 at
capture.

The `feeds` map is the provider's own canonical index of its agent surface:
`sitemap`, `rss`, `llms`, `llmsFull`, `agentCard`, `agentCardLegacy` and
`a2aEndpoint`. Prefer these URLs over any you construct yourself.

## Step 3 — route to the right surface

Finite State has three API surfaces and only one is public:

| Surface | Base | Auth | Use for |
|---|---|---|---|
| A2A content API | `https://finitestate.io/api/a2a` | none | blog, resources, videos, events, press |
| Platform REST API | `https://app.finitestate.io/api/public/v0` | API token | SBOMs, findings, components, scans |
| GraphQL API (legacy) | `https://platform.finitestate.io/api/v1/graphql` | OAuth2 client credentials | same platform data, via the Python SDK |

If you need platform data, stop using A2A. Obtain an API token from platform
Settings and send it as `X-Authorization: <token>` or
`Authorization: Bearer <token>`. Unauthenticated calls return `401` with an
envelope of `{valid, errors[{error, instanceLocation, keywordLocation}], requestId}` —
quote the `requestId` when contacting support.

The platform OpenAPI is served at `https://app.finitestate.io/api/docs/openapi.json`
and rendered per organization at `https://[org].finitestate.io/swagger`, but
both require the token, so no anonymous agent can read the contract.

## Step 4 — read `llms.txt` for the human-facing map

`https://finitestate.io/llms.txt` lists services, key pages, datasheets and
recent updates, and repeats the discovery-file index. It also states the
provider's own boundaries: no write-capable public A2A methods, and no private
customer data through public discovery files. Respect them.
