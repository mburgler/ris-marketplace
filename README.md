# ris-marketplace

A Claude Code marketplace shipping the **RIS** plugin — search and
citation over the Austrian **Rechtsinformationssystem** (RIS): federal, state,
district and municipal law, gazettes (Bundesgesetzblatt, Landesgesetzblätter),
and case law from the **VfGH, VwGH, OGH, BVwG, LVwG**, Datenschutzbehörde, and
a dozen further specialised tribunals.

Read-only. Public data, CC BY 4.0 (Bundeskanzleramt, Republik Österreich). No
write tools, no irreversible operations.

## Install

The plugin ships a pre-wired connector to the hosted MCP server at
`https://ris-mcp-worker.matteo-buergler.workers.dev/mcp` — no local server, no API key.

### Easiest — in the Claude app (chat or Cowork)

For non-technical users. Gives you both the tools **and** the bundled skill.

1. In the left sidebar, go to **Customize**.
2. At **Personal plugins** click **"+"**.
3. Choose **Create Plugin** → **Add marketplace** → **Add from a repository**.
4. Paste: `https://github.com/mburgler/ris-marketplace` and accept.
5. Next to **RIS - Österreichisches Recht Plugin**, click **"+"** to install.
6. In settings, **Check the sidebar:** both the skill **`ris-research`** *and* the connector
   **`ris-mcp`** must be installed. Typically, the connector
   **`ris-mcp`** must be added individually.

Adding the plugin in the Claude app allows for usage in the Claude Mobile App.

### Claude Code (CLI)

```text
/plugin marketplace add https://github.com/mburgler/ris-marketplace
/plugin install ris-mcp@ris-marketplace
/reload-plugins
/reload-skills
```

For command usage, type **`/ris-research`**.

## What you get

Eight tools and one skill:

| Tool | Purpose |
|---|---|
| `ris_about` | Server identity, sources, license, disclaimer |
| `ris_list_scopes` | Valid enum values for a field (Bundesland, Fachgebiet, …) |
| `ris_search` | Universal search across 38 RIS applications, eight corpora |
| `ris_get_document` | Full metadata for one record |
| `ris_get_section` | Full text of one paragraph / article / Rechtssatz |
| `ris_get_related` | Amendments, predecessors, cited norms, citing decisions |
| `ris_verify` | In-force check on a date against consolidated law |
| `ris_get_history` | Change feed for an Anwendung between two dates |

The bundled `ris-research` skill teaches the agent one discipline above all:
the **verbatim-vs-paraphrase** distinction — quotation marks mark text taken
unchanged from RIS, and paraphrase is never dressed as a quote. It keeps a lean
inline-tag vocabulary (`[Modellwissen — zu prüfen]`, `[…]`/`[gekürzt]`,
`[Übersetzung]`) and a one-sentence prompt-injection defence over retrieved
content. Disclaimer once per session, no per-answer boilerplate.

Every record carries a `ris_citation` (RIS internal id + ELI or Geschäftszahl +
`dokument_url` + `retrieved_at`) and an `is_authentic` flag — `true` only for
applications that carry legal force (the gazette and a handful of authentic
registers), `false` for consolidated text (`BrKons`, `LrKons`) and informational
records. The server keeps this as structured data; the skill uses it for routing
rather than surfacing it on every answer.

## Sanity check

```bash
curl https://ris-mcp-worker.matteo-buergler.workers.dev/health
# → { "status": "ok", "ris_reachable": true, ... }
```

## What's inside this repo

```
.claude-plugin/
  marketplace.json        # marketplace manifest
ris-mcp/                  # the plugin itself
  .claude-plugin/plugin.json
  .mcp.json               # connector configuration → hosted endpoint
  README.md               # plugin-level docs
  skills/ris-research/
    SKILL.md              # the agent skill
LICENSE                   # Apache-2.0 (plugin + skill content)
README.md                 # this file
```

The MCP server code (the thing that runs at `ris-mcp-worker.matteo-buergler.workers.dev`) is
maintained separately and is not part of this repository.

## License

- **Plugin / skill content (this repo):** Apache-2.0 — see [`LICENSE`](./LICENSE).
- **RIS data delivered by the server:** [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de) — credit *Bundeskanzleramt, Republik Österreich*.

## Disclaimer

Only specific RIS applications carry legal force. Consolidated text is
informational only — always cite the gazette for legally binding propositions.
**Not legal advice.**

## Contact

- Plugin maintainer: Matteo Bürgler · matteo.buergler@gmail.com
- Upstream support / mass-download coordination: `ris.it@bka.gv.at` (Bundeskanzleramt Abt. VII/6 — E-Government Bund/Verwaltung)
