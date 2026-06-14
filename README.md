# TrailWeights MCP Server

Public Model Context Protocol server for the TrailWeights ultralight gear
corpus. Lets ChatGPT, Claude, Apple Intelligence, Copilot, Gemini, and any
MCP-compliant client search 5,000+ products, 40,000+ creator transcript
chunks, and 21 in-house pack templates with verified weights and
cottage-first affiliate URLs.

## Transport

Streamable HTTP — JSON-RPC 2.0 over POST. Protocol version: `2025-06-18`.

## Auth

None. All tools are read-only. Rate limit: **60 requests/minute/IP**.
Rate-limit headers (`x-ratelimit-limit`, `x-ratelimit-remaining`,
`x-ratelimit-reset`) ride every response.

## Tools

| Name | Args | Returns |
| --- | --- | --- |
| `search_corpus` | `query`, `source_filter?`, `match_count?` | Top semantic matches across transcripts, products, packs, surveys, and the ultralight Bible. |
| `get_gear_reviews` | `product_id` | Up to 10 verified creator mentions with `youtube_url`, `timestamp_seconds`, and `snippet`. |
| `get_product_specs` | `product_id` or `slug` | Name, brand, category, verified weight (g & oz), MSRP, image, cottage-first affiliate buy URL. |
| `recommend_gear` | `query`, `weight_cap_oz?`, `category?`, `limit?` | Catalog recommendations sorted by relevance with verified weights and cottage-first affiliate URLs. |
| `compare_gear` | `product_ids[]` (2–6) | Side-by-side: name, brand, category, weight, MSRP, buy URL. |
| `find_lighter_alternative` | `product_id`, `limit?` | Up to 10 lighter same-category candidates sorted by weight, with `weight_savings_g`. |
| `get_pack_template` | `template_id` or `slug` | Full item list for one of the 21 in-house pack templates. |

## Quick start

```bash
# tools/list
curl -sX POST https://trailweights.com/api/mcp \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | jq

# recommend_gear
curl -sX POST https://trailweights.com/api/mcp \
  -H 'content-type: application/json' \
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
    "trailweights": { "type": "streamable-http", "url": "https://trailweights.com/api/mcp" }
  }
}
```

Discovery manifest: `https://trailweights.com/.well-known/mcp.json`
