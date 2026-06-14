# TrailWeights MCP Server

Public Model Context Protocol server for the TrailWeights ultralight gear
corpus. Lets ChatGPT, Claude, Apple Intelligence, Copilot, Gemini, and any
MCP-compliant client search 5,000+ products, 40,000+ creator transcript
chunks, and 21 in-house pack templates with verified weights and
cottage-first affiliate URLs.

- **Wire endpoint (production):** `POST https://trailweights.com/api/mcp`
- **Discovery manifest:** `https://trailweights.com/.well-known/mcp.json`

## Why

The TrailWeights monetization thesis treats MCP as the highest-leverage
distribution surface: every conversational agent that connects to the
server gets read access to the corpus and routes its readers through
cottage-first affiliate links. The catalog and creator-transcript moat
becomes addressable from outside trailweights.com.

## Transport

Streamable HTTP — JSON-RPC 2.0 over POST. The server speaks
`application/json` by default and can return a single-event SSE stream
when the client sends `Accept: text/event-stream`. GET returns 405
because every tool is stateless and short-lived; we do not keep
server→client streams open between calls.

Protocol version advertised: `2025-06-18`.

## Auth

None for v1. All seven tools are read-only against public catalog data.
Rate limit: **60 requests/minute/IP**, enforced at the edge of `/api/mcp`.
Rate-limit headers (`x-ratelimit-limit`, `x-ratelimit-remaining`,
`x-ratelimit-reset`) ride every response.

## Tools

| Name | Args | Returns |
| --- | --- | --- |
| `search_corpus` | `query`, `source_filter?`, `match_count?` | Top semantic matches across transcripts, products, packs, surveys, and the ultralight Bible. |
| `get_gear_reviews` | `product_id` | Up to 10 verified creator mentions. Each row carries `name`, `youtube_video_id`, `youtube_url`, `timestamp_seconds`, and `snippet`. |
| `get_product_specs` | `product_id` or `slug` | Canonical specs — name, brand, category, verified weight (g & oz), MSRP, image, cottage-first affiliate buy URL, and the trailweights.com page URL. |
| `recommend_gear` | `query`, `weight_cap_oz?`, `category?`, `limit?` | Structured catalog recommendations sorted by relevance, joined to verified weights and cottage-first affiliate URLs. |
| `compare_gear` | `product_ids[]` (2–6) | Side-by-side rows with name, brand, category, weight (g & oz), MSRP, buy URL. Reports `missing_ids` when an ID isn't in the browsable catalog. |
| `find_lighter_alternative` | `product_id`, `limit?` | Up to 10 lighter same-category candidates, sorted ascending by verified weight, with `weight_savings_g`. |
| `get_pack_template` | `template_id` or `slug` | Full item list for one of the 21 in-house pack templates. |

Tool argument schemas are JSON Schema draft-07 and surface verbatim
through `tools/list`.

## Quick start — `curl`

```bash
# 1) tools/list
curl -sX POST https://trailweights.com/api/mcp \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | jq

# 2) recommend_gear
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

### Claude Desktop

Add this to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "trailweights": {
      "url": "https://trailweights.com/api/mcp",
      "transport": "streamable-http"
    }
  }
}
```

Restart Claude Desktop. The seven tools appear under the Tools menu.

### ChatGPT (Custom GPT / Connectors)

In the GPT builder, add a connector with the URL
`https://trailweights.com/api/mcp` and select **Streamable HTTP** as the
transport. No auth header needed.

### Any MCP-compliant client

Most clients accept the `.well-known/mcp.json` manifest as a discovery
document. Point the client at `https://trailweights.com/.well-known/mcp.json`
and it will resolve the streamable-http endpoint and tool list
automatically.

## Roadmap

- v1.1: cache the cottage-first URL resolution in-process to bring p50 latency under 200 ms.
- v1.2: subdomain `mcp.trailweights.com` (Cloudflare DNS + Vercel domain alias — separate ticket).
- v2: write tools (`save_pack`, `track_buy_intent`) gated by an opt-in token tied to the user's TrailWeights account.
