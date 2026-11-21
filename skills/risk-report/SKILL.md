---
name: risk-report
description: Emit the assessment as an HTML Artifact — the security graph, attack-surface facets (reach, violated tools, exfil sinks, attack paths), findings by weakness and control, and the ARS. Use when a security engineer needs a shareable report at the end of a local assessment.
---

# Risk Report

Fold the enriched graph + findings into a single, shareable HTML Artifact rendered in the client's
harness. This is the deliverable; it reads the local artifacts and the AsterVault enrichment, and
renders — no server round-trip to display.

## When to Activate

- At the end of the pipeline, once findings + the graph exist.
- The engineer asks for a report, a BOM view, or an attack-surface summary.

## Core Concepts

- **Attack-surface facets** are derived from the graph (not stored): `reach` (regulated store ·
  classes · via-identity · columns), `tools_violated`, `sinks`, `paths`. Get them from
  `graph.enrich`, or compute from the local graph if offline.
- **One number:** the ARS (0–10, CVE-calibrated) from `risk.score` is the headline; every "overall
  risk" surface shows the same number.
- **Grouping that reads:** CVEs collapse to one row per vulnerable package (N packages · M CVEs),
  OS/base-image separated from app deps; findings group by weakness and by control family.

## Method

1. Call `graph.enrich` + `risk.score` (or load `./.asterguard/graph.json` + `findings.json`).
2. **Map the JSON to the render-model** (see `references/visual-grammar.md`): nodes→{id,label,sub,kind},
   edges→{from,to,label,onPath} (onPath from the flow's `path_labels`), the path + badges (`proven`
   from a `canary.verify` hit), the control chips (from each flow's `missing_control` + state), and the
   ARS band from `risk.score`.
3. **Render data-driven, not hand-placed, and publish as an Artifact.** Two templates, both auto-layout:
   - `templates/report.html` (**the composite** — use this by default): replace its four consts with the
     **raw tool JSON** — `ENRICH` = `graph.enrich`, `CONTROLS` = `controls.map`, `RISK` = `risk.score`
     (features on free, +ars on enterprise), `SCA` = the package roll-up or null. It draws the risk
     features, the information-flow (proven path), the security graph, and the crosswalk from that JSON.
   - `templates/attack-path.html` (the focused single-path view) for just the attack path.
   - `templates/engagement-console.html` (**the offensive/pentest view**): replace its two consts with
     the engagement roll-up + the red-team **run records** (`ENGAGEMENT` = `./.asterguard/engagement.json`,
     `RUNS` = `./.asterguard/runs/*.json`; schema in `behavioral-redteam/references/run-record.md`). It
     draws the risk cards, the lethal-trifecta kill-chain, and one transcript panel per run (attacker/agent
     turns, canary highlighted, per-turn verdicts for a multi-turn crescendo) with out-of-band evidence,
     verdict, ASR@budget, weakness and control chips. This is the view `pentest-report` renders.
   Both graph templates lay out *any* enrich output (layers by path distance, color by kind, proven path
   red). Add more views (BOM, exec scorecard, control coverage) the same way: feed a template the JSON.
4. Numbers only from the data; the picture is computed from the schema, never drawn by hand.

## Practical Guidance

- **Collapse, don't drop:** a full container SBOM is hundreds of OS CVEs — show vulnerable packages
  with drill-down, not a flat CVE wall (the graph holds the detail).
- Reach reachable by an **anonymous/external** identity is the lede — surface it at the top.
- Keep the report self-contained; it must render from the local artifacts even with no network.

## Output formats

Besides the HTML attack-surface Artifact, risk-report emits **SARIF** (`./.asterguard/report.sarif`)
when asked (`formats: [sarif, html]`), so the same findings feed GitHub code-scanning / CI gates. The
HTML stays the human view, SARIF the machine view — one finding set, two renderings.

## Integration

- all graph skills + security-judge — provide graph + findings
- AsterVault `graph.enrich` / `risk.score` — facets + ARS
- Output: an HTML Artifact (the console/BOM view), shareable from the harness

## References

- [Visual grammar (shared across views)](./references/visual-grammar.md)
- [Composite report renderer — inject graph.enrich/controls.map/risk.score JSON](./templates/report.html)
- [Attack-path renderer template (data-driven)](./templates/attack-path.html)
- [Engagement console (offensive) — inject engagement.json + runs/*.json](./templates/engagement-console.html)
- [Offensive run-record schema (the console's input)](../behavioral-redteam/references/run-record.md)
- [Graph schema (SSOT)](../../docs/graph-schema.md)
- [AsterVault MCP tools](../weakness-graph/references/astervault-tools.md)
