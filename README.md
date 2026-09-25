# @pipeworx/etsy

Search active Etsy listings via the Etsy Open API v3 — scoped to the three
endpoints that authenticate with just an API key (no OAuth user token).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

- `etsy_search_listings(keywords?, limit?, offset?, sort_on?, sort_order?, min_price?, max_price?, taxonomy_id?, buyer_country?)`
  — site-wide active-listing search (`findAllListingsActive`).
- `etsy_search_shop_listings(shop_id, keywords?, limit?, offset?, sort_on?, sort_order?)`
  — active listings within one shop (`findAllActiveListingsByShop`).
- `etsy_get_listing(listing_id)` — one listing by ID (`getListing`).

No sold-price / comps data — Etsy's v3 API has none. Active listings only.

## Auth

BYO key only (`_apiKey`), requires an API key. Register a free app at
<https://www.etsy.com/developers/register> to get a keystring + shared
secret. Pass `?_apiKey=<keystring>`; the colon-joined
`<keystring>:<shared_secret>` form your "Your Apps" page displays is accepted
too, and the pack handles the difference for you (see the trap below).

Etsy's v3 reference marks each endpoint's required auth. Only the three
endpoints above are `api_key`-only; everything else (shop/receipt/user
management, listing writes) requires an OAuth user token issued through
Etsy's manual commercial-review flow — out of scope for this pack. If Etsy's
review later grants broader access, extend this pack rather than building a
second one.

Registering an Etsy developer account requires signing in with (or creating)
an Etsy account and a password — a human step this pack's author cannot
perform on Bruce's behalf. See the open Needs Bruce item for platform-key
registration; this pack ships BYOK-gated in the meantime and its own live
call has not yet been verified against a real key.

## Data sources

- <https://developers.etsy.com/documentation/reference/> — Etsy Open API v3.
  Base URL `https://openapi.etsy.com/v3/application`.

Traps for the next person:

- **Etsy contradicts itself about the `x-api-key` header value, and you
  cannot resolve it by experiment.** The api_key reference for these three
  endpoints documents the bare keystring; the quickstart shows the
  colon-joined `keystring:shared_secret` pair (which is really the Basic-auth
  credential for the OAuth token endpoint). An app awaiting Etsy's Personal
  Approval returns a 403 that is *byte-identical* to the 403 a fabricated key
  returns — verified with a negative control on 2026-09-09 (fleet #1279) —
  so no probe before activation can tell the two header forms apart. The pack
  therefore sends the keystring and, only on a 403, retries once with the raw
  value the caller passed. Don't "simplify" that back to one call until a live
  successful response says which form won.
- This pack returns Etsy's response JSON close to verbatim (`count` +
  `results` plus a `source` field) rather than re-mapping field names, since
  the exact Listing resource shape was only checked against the published
  OpenAPI reference, not a live authenticated response (no working key was
  available at build time — see Auth above).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "etsy": {
      "url": "https://gateway.pipeworx.io/etsy/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/etsy/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1683+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/etsy_search_listings \
  -H 'Content-Type: application/json' \
  -d '{"keywords":"vintage brass lamp","limit":10}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/etsy_search_listings`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "etsy": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-etsy"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-etsy
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Etsy data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
