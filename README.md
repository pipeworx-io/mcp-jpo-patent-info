# @pipeworx/jpo-patent-info

Japanese patent, utility model and trademark status from the Japan Patent Office public registers, looked up by number.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `jp_patent_status(number, number_type?)` — legal status of a Japanese patent or utility model: stage (filing, examination, grant, refused, ceased), latest status event in English, application / publication / registration numbers and dates, title, applicant or right holder, FI classes.
- `jp_trademark_status(number, number_type?)` — Japanese trademark by application or registration number: word-mark text, kana reading, goods/services classes, owner, dates, mark type, status code.

## Auth

Keyless. No account, key or terms acceptance is used. The pack reads the register the JPO publishes to the public, the same one a visitor uses, and sends an honest User-Agent.

Not the same thing as the JPO's separate credentialed patent information service, which needs a registered ID and is deliberately NOT used here. That service carries things this pack does not: full prosecution history events, cited documents, refusal notices, applicant-code lookup and the IP5 dossier family data (the JPO closed new dossier registrations in 2024-08 anyway). Dossier tools are out of scope.

## Data sources

- Japan Patent Office, patent, utility model and trademark register (public).

Notes the next person would otherwise rediscover:

- Lookups are by number only. There is no name or keyword search here. Design registrations are not covered.
- The register publishes no as-of date, so `data_as_of` is the day of the call (the response says so); trademark rows carry their own `record_updated`.
- Trademark status is returned as the register's code, not decoded.
- A number the register does not know returns `found: false` with a message, never an empty array that reads as success. A rejected or non-JSON answer from the upstream is an error naming the upstream.
- Numbers: year-serial ("2020-000001"; "2020000001" and a 特願 prefix are accepted), or a plain registration number.
- Proxy-live: one upstream call per tool call, nothing stored.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "jpo-patent-info": {
      "url": "https://gateway.pipeworx.io/jpo-patent-info/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/jpo-patent-info/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/jp_patent_status \
  -H 'Content-Type: application/json' \
  -d '{"number":"2020-000001","number_type":"application"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/jp_patent_status`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-jpo-patent-info` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/jpo-patent-info/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "jpo-patent-info": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-jpo-patent-info"]
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
ask_pipeworx({ question: "your question about Jpo Patent Info data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
