# TrailWeights MCP Server

Model Context Protocol server for the TrailWeights ultralight gear corpus:
verified product weights, canonical specs, creator video reviews with
timestamps, pre-scored field consensus, and in-house pack templates. Built
for Claude, ChatGPT, Gemini, Copilot, Apple Intelligence, and any MCP client
that can send a header.

Live status page, key request, and pricing: <https://trailweights.com/mcp>

## Transport

Streamable HTTP — JSON-RPC 2.0 over `POST https://trailweights.com/api/mcp`.
Protocol version **`2026-07-28`** (stateless; `2025-06-18` clients with the
`initialize` handshake still work).

## Auth

**API key, required on every request.** Anonymous calls get `401`.

- `x-trailweights-key: <key>` — or —
- `Authorization: Bearer <key>`

Get a free key at <https://trailweights.com/mcp>: auto-issued, emailed once,
**500 tool calls per day** (resets 00:00 UTC), 60 requests per minute. Past
the daily cap the server answers `429` with `Retry-After`. Prepaid credit
packs and per-call payment (x402 on Base, Stripe MPP) are listed on the same
page; `402` responses carry the payment challenge when a key is out of
allowance.

Rate-limit headers (`x-ratelimit-limit`, `x-ratelimit-remaining`,
`x-ratelimit-reset`) ride every response.

## Tools

| Name | Args | Returns |
| --- | --- | --- |
| `search_corpus` | `query`, `source_filter?`, `match_count?` | Top semantic matches across creator transcripts, products, packs, surveys, and the curated gear knowledge base. |
| `get_product_specs` | `product_id` or `slug` | Name, brand, category, verified weight (g & oz), price, image, buy URL. |
| `recommend_gear` | `query`, `weight_cap_oz?`, `category?`, `limit?` | Deterministic recommendations with verified weights and buy links (no server-side LLM). |
| `compare_gear` | `product_ids[]` (2–6) | Side-by-side: name, brand, category, weight, price, buy URL. |
| `find_lighter_alternative` | `product_id`, `limit?` | Lighter same-category candidates sorted by verified weight, with `weight_savings_g`. |
| `get_gear_reviews` | `product_id` | Up to 10 verified creator mentions with `youtube_url`, `timestamp_seconds`, and `snippet`. |
| `get_field_consensus` | `product_id` or `slug` | What independent creators and the field agree on, scored on a deterministic 100-point rubric with a source-diversity gate. |
| `get_pack_template` | `template_id` or `slug` | Full item list for one of the in-house pack templates. |
| `report_outcome` | `request_id`, `answer_used?`, `citation_included?`, `helpfulness?` | Tell TrailWeights whether a prior result was used or cited. Telemetry only. |

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
