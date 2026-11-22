# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this is

A step-by-step modernization of IBM's DayTrader 7 (Java EE 7 on Open Liberty) to Java 25 + Spring Boot 4.1.
The plan is in [`docs/specs/`](docs/specs/README.md), the reasons in [`docs/adr/`](docs/adr/README.md), the
behavior that must not change in the [characterization catalog](docs/specs/characterization-catalog.md).

## Rules

- Work on one phase at a time, on its `phase/*` branch. Read its spec first. Do not start H or I before
  `v2.0.0` is tagged.
- One concern per commit. Build and characterization suite green before the next commit.
- Never edit a characterization scenario unless the catalog's "Changed in" column names the current phase and an
  ADR authorizes it.
- Never assert a price, an order/account/holding ID, or timing in tests (see ADR-0004).
- Do not fix security findings (ADR-0010) before phase E/F; document them.
- OpenRewrite output is committed separately from manual changes, and summarized in `docs/MIGRATION.md`.
- Every number in the README comes from `docs/metrics/*.json`. Never write an estimated metric.
- Files changed from upstream get a "Modified by …" line in their license header (Apache-2.0 section 4).
- Write documentation in English.

## Layout (legacy, until phase C)

| Module | Content |
|---|---|
| `daytrader-ee7-ejb` | `TradeServices`, `TradeSLSBBean` (stateless EJB), MDBs, `MarketSummarySingleton`, JPA entities, `TradeConfig` |
| `daytrader-ee7-web` | `/app` servlet + JSP, JSF pages, `/config`, WebSocket `/marketsummary`, benchmark primitives (`web/prims`) |
| `daytrader-ee7` | EAR packaging, Liberty `server.xml`, prebuilt Derby database |
| `jmeter_files` | Upstream JMeter plans |

## Environment

- Windows: clone to a short path (for example `C:\dev\daytrader-modernization`); source paths exceed the
  260-character limit otherwise.
- JDK 11 for phases A–B (legacy), JDK 25 from phase C and for the test tooling.
