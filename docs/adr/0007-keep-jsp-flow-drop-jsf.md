# ADR-0007: Keep the JSP flow (`/app`) as the only UI and remove the JSF UI

- **Status:** Proposed
- **Date:** 2026-11-03
- **Phase:** B
- **Supersedes / Superseded by:** none

## Context

DayTrader has two complete user interfaces over the same services:

- **JSP**: one servlet, `/app`, dispatching on `action=` (`login`, `home`, `quotes`, `buy`, `sell`,
  `portfolio`, `account`, `update_profile`, `mksummary`, `register`, `logout`) to JSP pages;
- **JSF**: `*.faces` pages backed by about 14 CDI/JSF beans in `web/jsf/`.

The upstream JMeter plan `daytrader7.jmx` drives mostly the JSF pages (22 of 38 samplers hit `*.faces`). The
market summary WebSocket client (`marketSummary.html`) is included by the JSP page, so it belongs to the JSP
flow.

## Decision

We will keep the JSP flow as the only UI and remove JSF in phase B. In phase D the JSP pages become Thymeleaf
templates behind the same URLs.

To keep before/after performance numbers valid, **phase A writes a JMeter plan that drives only `/app`**
(`perf/daytrader-app.jmx`) and runs it against `v0-legacy`. That plan, not the upstream one, is the baseline
for every later measurement. The upstream plan is also run once on `v0-legacy` and stored for reference.

## Alternatives considered

| Option | Why not |
|---|---|
| Keep JSF and migrate it (JoinFaces on Spring Boot) | Doubles the UI work; JSF is rarely chosen for new Spring applications |
| Keep JSF and drop JSP | JSF pages carry more framework state; JSP maps directly to Spring MVC + Thymeleaf |
| Replace both with a SPA | A new product, not a modernization; out of scope (a REST API comes in H) |

## Consequences

- **The official DayTrader benchmark is no longer reproducible after phase B.** Published numbers in this
  repository are for the `/app` plan only, and the README says so.
- The characterization suite only needs to cover one UI.
