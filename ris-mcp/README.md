RIS brings search and citation across Austrian federal, state, district, and municipal law, gazettes, and case law from the Verfassungsgerichtshof (VfGH), Verwaltungsgerichtshof (VwGH), Oberster Gerichtshof (OGH), Bundesverwaltungsgericht (BVwG), Landesverwaltungsgerichte (LVwG), Datenschutzbehörde, and a dozen further specialised tribunals into Claude. Built on the public OGD API of the Bundeskanzleramt Rechtsinformationssystem (RIS). Every result carries a `ris_citation` (RIS internal id + ELI or Geschäftszahl + `dokument_url`) and an `is_authentic` flag distinguishing legally binding gazette entries (`BgblAuth`, `LgblAuth`, `Vbl`, `Bvb`, `GrA`, `PruefGewO`, `Avsv`, `Spg`, `Avn`, `KmGer`) from informational consolidated text (`BrKons`, `LrKons`). The flag is structured data the bundled skill uses for routing — it is not surfaced as per-answer boilerplate. The skill's one core discipline is **verbatim vs. paraphrase**: quoted statutory or decisional text is reproduced unchanged from RIS, and paraphrase is never dressed as a quote. Read-only — no write tools, no irreversible operations.

- Universal search across 38 RIS applications grouped into eight corpora (`federal_law`, `state_law`, `case_law`, `gazettes`, `drafts_and_bills`, `municipal`, `official_announcements`, `english_translations`).
- Article- and paragraph-level retrieval (`ris_get_section`) with historical snapshots via `fassung_vom`.
- Document-level metadata (`ris_get_document`) with structured content URLs — canonical view plus the amtssignierte `.pdfsig` for legally binding apps.
- Citation / amendment graph (`ris_get_related`) — predecessors, successors, amendments, cited norms, citing decisions.
- In-force verification on any date (`ris_verify`) against consolidated text.
- Change feed per application between two dates (`ris_get_history`) — the freshness primitive for "what's new in BgblAuth this week."
- Discovery primitives (`ris_about`, `ris_list_scopes`) so the agent introspects valid enum values rather than memorising them.

## Example use cases

1. What does Article 6 GDPR look like in Austrian implementation under the Datenschutzgesetz? Quote §1 verbatim.
2. Has the Verwaltungsgerichtshof ruled on §73 AVG (administrative-procedure delay) since 2024? Show the three most recent decisions.
3. Is the Wasserrechtsgesetz 1959 currently in force? Live check, please.
4. What amends the Mietrechtsgesetz, and what does the consolidated §16 say as of today?
5. Which BVwG decisions since January cite §3 AsylG 2005?

## When to use

Any question answerable from Austrian primary law, secondary legislation, or case law in the RIS corpus. Examples include:

- **Federal law:** text, structure, in-force status, version on a specific date for any consolidated Bundesnorm (`Wasserrechtsgesetz`, `ABGB`, `StGB`, `UGB`, `ASVG`, `GewO`, …).
- **State law:** the same, for any of the nine `Landesgesetze` (consolidated or as gazetted).
- **Gazettes:** authentic and historical Bundesgesetzblatt (BGBl. I/II/III), Staatsgesetzblatt 1945, Reichsgesetzblatt 1848–1940, and all Landesgesetzblätter.
- **Case law:** decisions of the Verfassungsgerichtshof, Verwaltungsgerichtshof, Oberster Gerichtshof, Bundesverwaltungsgericht, Landesverwaltungsgerichte, Datenschutzbehörde (and historical Datenschutzkommission), Bundesdisziplinarbehörde, Personalvertretungsaufsichtsbehörde, Gleichbehandlungskommissionen, Asylgerichtshof, Unabhängiger Bundesasylsenat, Umweltsenat, Bundeskommunikationssenat, Vergabekontrollbehörden, and the unabhängige Verwaltungssenate.
- **Begutachtungsentwürfe and Regierungsvorlagen** (consultation drafts and government bills from 2004 onward).
- **English translations** of selected federal law (the `Erv` corpus).
- **District and municipal law:** Kundmachungen der Bezirksverwaltungsbehörden (NÖ, OÖ, Tirol, Vbg, Bgld, Stmk) and rechtsverbindliche Gemeindekundmachungen (Tirol, OÖ, Vbg).
- **Ministerratsprotokolle** (from 2020), ministry decrees (`Erlässe`), and amtliche Veterinärnachrichten.
- **Change tracking:** "What's been added/modified/deleted in BgblAuth since [date]?" via `ris_get_history`.

## When not to use

- **Non-Austrian jurisdictions** — EU, US, German, Swiss, UK and other national law are out of corpus; use a jurisdiction-appropriate research tool for those.
- **Drafting tasks** — RIS returns authority; pair with a drafting skill or tool. RIS itself doesn't draft.
- **Outcome prediction or factual analysis of a client matter** — out of scope; defer to lawyer judgement.
- **Boolean search syntax with multi-word wildcards** (`A* AND B`, `(x OR y) AND z`) — RIS supports one `*` per word with ≥2 characters on each side, and uses German operators (`und`, `oder`, `nicht`) instead of English.
- **Pulling entire applications as a bulk corpus** — the FAQ explicitly asks operators to notify `ris.it@bka.gv.at` first and schedule off-hours. The MCP server refuses requests that would page past 1,000 records unless `bulk_acknowledged=true`.
- **Legally binding citation without authenticity check** — `BrKons` and `LrKons` are informational only. For binding propositions, cite the parallel `BgblAuth` / `LgblAuth` record (the MCP server surfaces this as `content_urls.authentic`).

## How it connects

The shipped [`.mcp.json`](./.mcp.json) points at the hosted MCP server at `https://ris-mcp-worker.matteo-buergler.workers.dev/mcp`. Installing this plugin from the marketplace wires the connector automatically — no local server required.

Liveness probe (unauthenticated):

```bash
curl https://ris-mcp-worker.matteo-buergler.workers.dev/health
# → { "status": "ok", "ris_reachable": true, ... }
```

If `ris_reachable` is `false`, the upstream RIS OGD API at `data.bka.gv.at` is unreachable from the MCP server — usually transient.

## Links

- **Upstream RIS OGD API:** https://data.bka.gv.at/ris/api/v2.6
- **Citizen application:** https://www.ris.bka.gv.at/
- **Dataset entry on data.gv.at:** https://www.data.gv.at/datasets/0fb9ae1a-92cb-4ab8-a589-470c16d4fe21
- **License:** RIS data is published under [Creative Commons BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de) — credit the Bundeskanzleramt, Republik Österreich. The skill, plugin, and MCP server code are Apache-2.0 © Matteo Bürgler.
- **Upstream support / mass-download coordination:** `ris.it@bka.gv.at` (Bundeskanzleramt Abt. VII/6 — E-Government Bund/Verwaltung).
- **Plugin maintainer:** Matteo Bürgler · matteo.buergler@gmail.com
- **Disclaimer (surfaced verbatim by `ris_about`):** *"Only specific RIS applications carry legal force. Consolidated text is informational only — always cite the gazette for legally binding propositions. The RIS data is provided by the Bundeskanzleramt under CC BY 4.0. Not legal advice."*
