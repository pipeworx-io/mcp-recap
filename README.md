# RECAP — documents filed in US federal court cases

Direct links to the complaints, motions, orders and exhibits filed in federal
cases. No API key, no PACER fee.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Auth

None.

## How this is possible without a key

RECAP documents are **not** in CourtListener's bulk export. They are mirrored to
the Internet Archive under an identifier that can be **derived**:

```
gov.uscourts.<court_id>.<pacer_case_id>
```

Federal docket coverage carries `court_id` for 100% of dockets and
`pacer_case_id` for 84.8%, so a case's filings have a computable address. No
agreement with anyone, no per-page charge.

## What you get, and what you don't

**You get:** a link to each PDF, its docket-entry number and attachment number,
its size, and the docket sheet where one exists.

**You don't get text.** Measured across 400 dockets: 141 archive items carry
PDFs and **none** carry extracted text. Reading a filing means opening the
document. Making these searchable would be an OCR project; this is not one.

## Coverage is partial, and uneven

RECAP holds what PACER users have paid for and contributed, so coverage follows
where lawyers use it. Measured over 400 dockets across 16 courts:

| Court | | Court | |
|---|---|---|---|
| `cafc` Federal Circuit | **76%** | `cacd` C.D. Cal. | 20% |
| `ca2` Second Circuit | **68%** | `cand` N.D. Cal. | 16% |
| `paed` E.D. Pa. | 60% | `txsd` S.D. Tex. | 12% |
| `ca9` Ninth Circuit | 56% | `ilnd` N.D. Ill. | 12% |
| `nysd` / `dcd` | 48% | `cacb` C.D. Cal. Bankr. | **8%** |

**Overall: 35.8%.**

The consequence matters more than the number: **"no documents" means none have
been contributed to RECAP — not that nothing was filed.** Roughly two cases in
three will come back empty, and every response says so. Treating an empty
result as a fact about the court record would be wrong most of the time.

## Tool

`recap_filings` — give it a `docket_id` from any docket search, or a `court`
plus `docket_number`.

```
recap_filings({docket_id: 73433096})
  → V.O.S. Selections, Inc. v. Trump (cafc), 18 documents
```

Filenames encode the structure: `….12.0.pdf` is docket entry 12, `….12.1.pdf`
its first attachment. That ordering is the only structure the archive provides —
there are no document titles — so it is returned rather than hidden.

## Related

- `federal_civil_search`, `federal_criminal_search`, `federal_appeal_search`,
  `bankruptcy_search` — find the docket first
- `court-listener` — opinions and full case text, a different corpus entirely

## Data sources

- https://archive.org/ — RECAP archive items
- Federal docket coverage (CourtListener bulk data, Public Domain Mark)

RECAP exists because Free Law Project built it and PACER users contributed to
it: <https://free.law/donate/>

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "recap": {
      "url": "https://gateway.pipeworx.io/recap/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/recap/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/recap_filings \
  -H 'Content-Type: application/json' \
  -d '{"docket_id":73433096}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/recap_filings`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-recap` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/recap/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "recap": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-recap"]
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
ask_pipeworx({ question: "your question about Recap data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
