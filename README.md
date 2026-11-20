# @pipeworx/psa-ph

Philippine Statistics Authority (PSA) MCP: find a published table, read its dimensions, and pull
labelled figures (consumer prices, inflation, labour force, population, poverty, national accounts,
trade, agriculture, tourism and more). Philippines only. Live call per request, nothing stored.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2016+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `psa_ph_search_tables(query, limit?)` — find tables by keyword. Returns the native table id, title
  and last-updated date. Index hits for tables PSA has since removed are dropped and counted.
- `psa_ph_table_info(table_id, dimension?, match?)` — dimensions of one table with their valid value
  codes and labels; pass `dimension` (and `match`) to list every value of one dimension.
- `psa_ph_get_data(table_id, select?)` — figures as labelled rows. `select` maps a dimension to a
  value label or code, an array, `"*"`, `"first"`, `"last"` or `"last:N"`. Unselected dimensions are
  defaulted (time dimensions to the latest value, others to the first, usually the total) and every
  default is listed in `selection_resolved` and `note`. Selections above 1,500 cells are refused with
  the per-dimension sizes. A selection that has no published value says why in `empty_reason`.

Table ids are the publisher's native ids, e.g. `2M/PI/CPI/2018/0012M4ACP09.px`; use them exactly as
returned.

## Auth

Keyless.

## Data sources

- Philippine Statistics Authority (PSA), OpenSTAT database of official statistical tables.

## Notes and traps

- The publisher's search index can list tables that have been removed. They are dropped, with a count
  in `note`, so a hit is a table that exists.
- Value codes are positional strings ("0", "1", ...). Select by label.
- A year dimension is usually followed by a period dimension whose last value is the annual average
  ("Ave"), so the last value of Period is not the latest month. Defaults are reported, never silent.
- `data_as_of` is the table's last-updated time as the publisher lists it. The `updated` field inside a
  data response is the time of the query and is not used.
- The units and base year are written in the table title (e.g. "2018=100"); there is no separate unit
  field.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "psa-ph": {
      "url": "https://gateway.pipeworx.io/psa-ph/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/psa-ph/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2016+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/psa_ph_search_tables \
  -H 'Content-Type: application/json' \
  -d '{"query":"consumer price index","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/psa_ph_search_tables`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-psa-ph` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/psa-ph/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "psa-ph": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-psa-ph"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Psa Ph data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
