# @pipeworx/va-scc-entities

Virginia corporations and LLCs from the State Corporation Commission, Clerk's Information System, as the Virginia Open Data Portal publishes them, with the data's staleness stated in every response.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2024+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `va_scc_search_entities(name, kind?, status?, limit?)` -- find a Virginia corporation or LLC by name. A row is a hit only when its normalised name equals the name asked for (Inc / Incorporated, LLC / L.L.C. / Limited Liability Company, a trailing full stop and & vs and all match); names that only start with it come back separately as `also_starting_with`.
- `va_scc_entity(entity_id, kind?)` -- the record for one entity by its SCC id: status and reason, formation date, state of formation, principal office, and the type-specific flags.
- `va_scc_new_entities(since, kind?, name_query?, limit?)` -- entities formed on or after a date, newest first. A history feed to the date the data stops, not a live feed.

## Auth

Keyless.

## The data is stale and cut short

The tables stop early, and this is measured on every call, not assumed:

- The LLC table holds exactly 1,048,575 rows (a spreadsheet row cap) and its newest formation date is 2022-06-16 (measured 2026-11-16).
- The corporation table's newest formation date is 2024-11-09 (measured 2026-11-16), although the portal lists it as modified in 2026-03.

Every response carries `data_as_of` (the older of the two newest formation dates), `data_as_of_by_table`, `stale: true` and a `notice` saying that an entity formed after those dates is absent. A name search that finds nothing returns `found: false` together with `not_found_notice`; it never means "no such company in Virginia". When the portal cannot be reached the tools throw an error instead of returning an empty result. If the live measurement of the newest date fails, the 2026-11-16 figure is used and the response says so in `measurement_note`.

## Data sources

- Virginia State Corporation Commission, Clerk's Information System, corporation and limited liability company tables, published on the Virginia Open Data Portal. The portal lists the licence as "not specified", so the tools call the portal live per request and copy nothing.

## What is not served

No names of individuals. The officer dataset is not read, and the registered agent (often a person) is not read into any result. The principal office is the entity's own address.

## Notes for the next person

- Entity ids in the tables carry a leading tab and trailing spaces, and corporation ids are 8 digits with leading zeros; the lookup trims both and pads a short numeric id.
- The two tables store dates differently (LLC `m/d/yyyy`, corporation ISO text with a time); both are returned as `yyyy-mm-dd`.
- The portal's SQL endpoint refuses some functions and its firewall refuses others. A query shape that is fine in a database can return a 403 challenge page, which the pack reports as an upstream failure.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "va-scc-entities": {
      "url": "https://gateway.pipeworx.io/va-scc-entities/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/va-scc-entities/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2024+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/va_scc_search_entities \
  -H 'Content-Type: application/json' \
  -d '{"name":"Acme Motor Company Inc","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/va_scc_search_entities`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-va-scc-entities` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/va-scc-entities/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "va-scc-entities": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-va-scc-entities"]
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
ask_pipeworx({ question: "your question about Va Scc Entities data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
