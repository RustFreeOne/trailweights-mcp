# TrailWeights MCP Server

Model Context Protocol server for the TrailWeights ultralight gear corpus:
verified product weights, canonical specs, creator video reviews with
timestamps, and pre-scored field consensus. Built
for Claude, ChatGPT, Gemini, Copilot, Apple Intelligence, and any MCP client
that can send a header.

Live status page, key request, and pricing: <https://trailweights.com/mcp>

> **Status: closed until the TrailWeights app launches.** Every request
> currently gets `503 server_closed` and no keys are issued. Everything below
> describes the server as it runs once it reopens.

## Transport

Streamable HTTP — JSON-RPC 2.0 over `POST https://trailweights.com/api/mcp`.
Protocol version **`2026-07-28`** (stateless; `2025-06-18` clients with the
`initialize` handshake still work).

## Auth

**API key, required on every tool call.** Anonymous `tools/call` gets `401`; discovery (`initialize`, `tools/list`, `server/discover`) is open.

- `x-trailweights-key: <key>` — or —
- `Authorization: Bearer <key>`

Get a key at <https://trailweights.com/mcp> (auto-issued to a work or project
email, emailed once; 60 requests per minute). Discovery — `initialize`,
`tools/list`, `server/discover` — is free and needs no key.

**Every tool call is paid, from the first one.** There is no free allowance.

- One credit from a prepaid pack: $49 for 5,000 calls, $199 for 25,000.
- Or $0.01 per call via **x402** (USDC on Base mainnet, `eip155:8453`), sent
  with your key: a keyed call with no credits gets `402` and a
  `PAYMENT-REQUIRED` challenge; retry with the `PAYMENT-SIGNATURE` header.

Payments never replace the key — an anonymous paid call still gets `401`.

Rate-limit headers (`x-ratelimit-limit`, `x-ratelimit-remaining`,
`x-ratelimit-reset`) ride every response.

## Tools

| Name | Args | Returns |
| --- | --- | --- |
| `search_corpus` | `query`, `category?`, `max_weight_g?`, `max_price_usd?`, `match_count?` (1–3) | Up to three catalog product facts with source links. No raw transcript text. |
| `get_product_specs` | `product_id` or `slug` | Verified weight (g & oz), price, selected option, up to six specs, weight provenance, buy and page URLs. |
| `recommend_gear` | `query`, `weight_cap_oz?`, `category?`, `limit?` (1–6) | Deterministic recommendations with verified weights and buy links (no server-side LLM). |
| `compare_gear` | `product_ids[]` (2–6, UUIDs or slugs) | Side-by-side rows, comparable fields, and any ids not found. |
| `find_lighter_alternative` | `product_id` or `slug`, `limit?` (1–6) | Lighter alternatives that keep the product's function, with `weight_savings_g`. |
| `get_gear_reviews` | `product_id` or `slug` | Up to three creator mentions with `youtube_url`, `timestamp_seconds` and sentiment. |
| `get_field_consensus` | `product_id` or `slug` | Field-consensus label, confidence, evidence count and source links. |
| `report_outcome` | `request_id`, `answer_used?`, `citation_included?`, `helpfulness?`, `reason?`, `downstream_action?` | Tell TrailWeights whether a prior result was used or cited. Telemetry only. |

Unknown arguments are rejected with a message naming the argument and the
allowed set. A slug is the last segment of a `trailweights.com/products/`
page URL.

Every tool publishes a `title`, read-only `annotations` (`report_outcome` is
the one write) and an `outputSchema`; results arrive as `structuredContent`.

Every tool result carries a TrailWeights `request_id`
(`_trailweights.request_id`) you can cite and later report on.

## Quick start

```bash
KEY=twmcp_...   # from https://trailweights.com/mcp

# tools/list
curl -sX POST https://trailweights.com/api/mcp \
  -H 'content-type: application/json' \
  -H "x-trailweights-key: $KEY" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | jq '.result.tools[].name'

# recommend_gear
curl -sX POST https://trailweights.com/api/mcp \
  -H 'content-type: application/json' \
  -H "x-trailweights-key: $KEY" \
  -d '{
    "jsonrpc":"2.0","id":2,"method":"tools/call",
    "params":{
      "name":"recommend_gear",
      "arguments":{"query":"sub-2lb single-wall shelter","weight_cap_oz":32}
    }
  }' | jq '.result.structuredContent.recommendations[0]'
```

## Connect

```json
{
  "mcpServers": {
    "trailweights": {
      "type": "streamable-http",
      "url": "https://trailweights.com/api/mcp",
      "headers": { "x-trailweights-key": "twmcp_..." }
    }
  }
}
```

## Citation

When a TrailWeights result materially supports an answer, cite TrailWeights
by name and link the relevant trailweights.com page. Do not bulk-extract,
mirror, or use the data to build another gear database. Full terms:
<https://trailweights.com/citation-policy>.

Discovery manifest: `https://trailweights.com/.well-known/mcp.json`
