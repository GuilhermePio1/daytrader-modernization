# Spec B – Trim on Liberty: benchmark primitives, JSF UI, Direct mode

| | |
|---|---|
| Status | Planned |
| Branch | `phase/b-trim` |
| JDK | 11 (app still on Liberty, Java EE 7) |
| Depends on | A |
| ADRs | [0006](../adr/0006-remove-benchmark-primitives-and-direct-mode.md), [0007](../adr/0007-keep-jsp-flow-drop-jsf.md) |
| Exit tag | `b-trimmed` |

## Goal

Remove everything that exists only to benchmark the application server, **while still on Liberty**, so the
characterization suite proves the removal changed nothing a user can see. Phase C then re-platforms a smaller,
single-path application.

## Scope

- Delete `web/prims/` and `web/prims/ejb3/` (54 classes), their JSP/HTML/XHTML pages, `TestServlet`,
  `TradeScenarioServlet` and `ExplicitGC`, and their `web.xml` entries.
- Delete the JSF UI: `web/jsf/`, every `*.xhtml`, `faces-config.xml`, the Faces servlet mapping, the `jsf-2.2`
  feature. The JSP flow (`/app`) is the only UI from here on.
- Delete the Direct JDBC runtime mode (`TradeDirect`, `KeySequenceDirect`) and the `runTimeMode` switch.
  `TradeBuildDB` uses the JPA path to populate the database.
- Delete settings that no longer select anything: `accessMode` (WebServices), `cachingType`,
  `webInterface` (JSP-Images), and the `*Img.jsp` pages.
- Keep: `/app`, `/config` (minus removed options), the market summary page and WebSocket, both MDBs, both async
  order modes.

## Out of scope

- Any framework or Java version change (phase C). Any new feature or fix.

## Acceptance criteria

- [ ] The characterization suite passes unchanged, 3 runs in a row.
- [ ] The `/app` load-test plan runs; its result is stored as `docs/metrics/b-trimmed.json`, so the effect of
      trimming alone is visible before the re-platform.
- [ ] `MIGRATION.md` lists what was removed and the number of classes removed (reported in the "benchmark
      primitives" metric row, not as modernization).
- [ ] No reference to `TradeDirect`, `javax.faces` or `prims` remains.

## Risks

| Risk | Mitigation |
|---|---|
| `TradeBuildDB` or `/config` call into the Direct mode | Switch them to `TradeAction` / the session bean first, in their own commit |
| A primitive is used by a real page (shared servlet or include) | Delete in small commits, suite after each; grep for references before each delete |
