# @pipeworx/al-sos-business

Alabama business entities from the Alabama Secretary of State: is a company registered in Alabama, under what entity number, and is it in good standing, dissolved, withdrawn or merged. Corporations, LLCs, LPs, LLPs, nonprofits, and foreign entities qualified to do business in Alabama, in every status.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2014+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `alsos_search_entities(name, status?, limit?)` — search by company name. A row is a match only when its name equals the query once case, punctuation, "&" vs "and" and the legal-form spelling (Inc/Incorporated, LLC/L.L.C., Co/Company) are set aside; each match says `exact` or `legal_form_variant`. Other names that merely contain the same words come back as `related_names` on a miss, never as matches. Each row carries entity number, name, `name_kind` (current, previous_name or registered_name), entity type, city and state, status as the register words it, and `status_normalized` (active or inactive).
- `alsos_get_entity(entity_number)` — one entity by its entity number ("000-027-643", or "27643"): current name, entity type, status and its dissolution or withdrawal date, domestic or foreign, place and date of formation, date qualified in Alabama, principal address, nature of business, authorized capital, and the registered agent when it is a business.

There is no formed-since or new-entities tool. The register offers no listing by date, so none is offered.

## Auth

Keyless.

## What is not served

Entity-level data only. The register's detail page also lists incorporators, members and partners and a transaction history that names individuals; those are counted in `individuals_not_served` and never named. A registered agent is returned only when it is plainly a business (a commercial agent such as a corporation service company); an agent that is a person, or a name that cannot be classed, is withheld together with the registered office address that usually goes with it, and the response says so in `registered_agent_withheld`.

## Data sources

- Alabama Secretary of State, Business Entity Records. Every call reads the register live; nothing is stored. The register publishes no revision date, so `data_as_of` is the date of the read and `data_as_of_basis` says so.

Quirks worth knowing:

- The register finds names that contain every word typed, in any order, and reads punctuation literally ("Apple, Inc." finds nothing where "Apple Inc" finds the company). The tool sends the name as typed, each spelling of its legal form, and its core, all without commas and full stops.
- A common word fills the first page with other names. The tool reads up to three more pages looking for a match and sets `truncated` when it stopped before the end.
- A search row can be a company's previous name or registered name; its status column then says so instead of giving a status. `alsos_get_entity` gives the current name and status.
- A miss is `no_name_match`. A register that did not answer, or answered with a page the tool does not recognise, is `upstream_unavailable` or `no_name_match_incomplete`, never an empty success.
- The company-register search (`company_register_search`, country US) asks this pack for the Alabama register alongside the other US state registers.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "al-sos-business": {
      "url": "https://gateway.pipeworx.io/al-sos-business/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/al-sos-business/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2014+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/alsos_search_entities \
  -H 'Content-Type: application/json' \
  -d '{"name":"Acme Auto Leasing LLC"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/alsos_search_entities`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-al-sos-business` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/al-sos-business/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "al-sos-business": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-al-sos-business"]
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
ask_pipeworx({ question: "your question about Al Sos Business data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
