# Harness Atlas — MCP server

A remote [Model Context Protocol](https://modelcontextprotocol.io) server for sourcing
**wire harnesses and cable assemblies**. It answers two questions an agent cannot answer from
general web knowledge: *who manufactures this*, and *what part can replace this one*.

Hosted, public, no API key, nothing to install.

```
https://harnessatlas.com/api/mcp
```

Listed in the official MCP registry as `io.github.pawel-kowalczyk/harness-atlas`.

> The endpoint moved to `harnessatlas.com` on 2026-08-24 (manifest v0.2.0). The previous Supabase
> URL still works and is not being retired, so existing clients keep functioning — but this is the
> address to use.

---

## Why it exists

Wire-harness sourcing data is scattered across trade directories, PDF catalogues and
manufacturer sites that general-purpose models summarise badly and cite unreliably. Harness
Atlas is a structured directory of the sector with per-field provenance, exposed over MCP so an
agent can query it directly instead of guessing.

## Tools

### `find_manufacturer`

Search the directory of wire-harness and cable-assembly manufacturers. All parameters optional.

| Parameter | Type | Notes |
|---|---|---|
| `country` | string | HQ country, e.g. `Germany`, `Poland`, `China` |
| `industry` | enum | `aerospace` · `appliances` · `automotive` · `commercial-vehicles` · `electronics` · `energy` · `hvac` · `industrial` · `marine` · `medical` · `off-road` · `rolling-stock` · `telecommunications` |
| `capability` | enum | `crimp_force_monitoring` · `ultrasonic_welding` · `soldering` · `overmolding` · `braiding` · `twisting` · `sealing_grommets` · `heatshrink` · `labeling_marking` · `testing_continuity` · `hipot` · `functional_test` · `hv_capable` · `ipc_whma_620` |
| `name` | string | Partial company-name match |
| `limit` | integer | 1–50, default 20 |

Returns name, HQ country and city, website, industries served, plant count, capabilities and a
profile URL.

### `find_alternative`

Cross-reference a component part number — connectors, terminals, seals, cable protection —
against the XrefBase equivalence graph. `part_number` is required; `manufacturer` is optional and
disambiguates an MPN that exists under several makers (aliases such as `TE` and `H+S` are
accepted). Punctuation and case are normalised, so `1-967616-1` and `19676161` both match.

**Results are grouped by relationship strength, and the groups are not interchangeable:**

| Group | Meaning |
|---|---|
| `verified_replacements` | Corroborated drop-in or manufacturer-superseded parts |
| `functional_equivalents` | Same job, but needs engineering review |
| `same_series`, `variants` | Same family, different configuration — **not** substitutes |
| `mates_with` | The counterpart it plugs into — specifically **not** a substitute |

Every claim carries a confidence level and, where one exists, a public source URL. The grouping
is the point: a tool that returns one flat list of "alternatives" invites an agent to propose a
mating connector as a replacement part.

## Connecting

**Claude Code**

```bash
claude mcp add --transport http harness-atlas https://harnessatlas.com/api/mcp
```

**Claude Desktop / any client taking a remote MCP URL** — add to your MCP config:

```json
{
  "mcpServers": {
    "harness-atlas": {
      "type": "streamable-http",
      "url": "https://harnessatlas.com/api/mcp"
    }
  }
}
```

**Straight JSON-RPC**, to try it without a client:

```bash
curl -s -X POST https://harnessatlas.com/api/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
        "name":"find_manufacturer",
        "arguments":{"country":"Poland","industry":"automotive","limit":3}}}'
```

## The data, honestly

- **513 published manufacturers, 715 plants, 65 countries.** Coverage is deepest in Europe,
  with meaningful presence in North America and Asia.
- **Profiles are compiled from public sources and manufacturer self-declaration.** Independent
  verification is in progress and is not yet reflected in most listings — treat capability and
  certification claims as sourced, not audited, and follow the profile URL to the operator's own
  material before relying on one commercially.
- **Equivalence claims carry their own confidence level** and a source URL where a public one
  exists. `verified_replacements` is the only group that should be read as a substitution
  recommendation, and even that warrants engineering review for a safety-relevant application.
- Corrections are welcome — open an issue here or use the contact form on the site.

## Scope

Harness Atlas is a directory and reference service. It does not broker, quote or transact, and
listing is not endorsement.

## This repository

The server runs as a hosted endpoint; its implementation is not open source. This repository is
the public home for the server: its registry manifest, documentation and examples.

`server.json` is a mirror published for registry discovery. The canonical copy lives with the
implementation and is what `mcp-publisher` publishes — if the two ever disagree, the version in
the official registry is authoritative.

## Links

- Site — <https://harnessatlas.com>
- About the MCP server — <https://harnessatlas.com/mcp>
- Official MCP registry entry — `io.github.pawel-kowalczyk/harness-atlas`

## Licence

[MIT](LICENSE) — covering this repository's contents: the server manifest, documentation and
examples.

The licence does not extend to the Harness Atlas directory data returned by the hosted endpoint.
That data remains proprietary and is offered for use through the endpoint itself.
