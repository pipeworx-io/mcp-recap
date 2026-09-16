# RECAP — documents filed in US federal court cases

Direct links to the complaints, motions, orders and exhibits filed in federal
cases. No API key, no PACER fee.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

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

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

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

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-recap
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

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
