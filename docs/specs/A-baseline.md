# Spec A – Make the legacy build run, characterization suite, baseline load test

| | |
|---|---|
| Status | Planned |
| Branch | `phase/a-baseline` |
| JDK | 11 for the app (upstream tag `v1.3` is "fix for jdk 11"), 25 for `characterization-tests/` and `perf/` tooling |
| Depends on | 00 |
| ADRs | [0004](../adr/0004-black-box-invariant-characterization-tests.md), [0010](../adr/0010-legacy-security-findings.md) |
| Contracts | `docs/contracts/legacy-app.md`, `docs/contracts/marketsummary-websocket.md` (captured and frozen in this phase) |
| Exit tag | `a-safety-net` |

## Goal

Get the legacy EAR to build and run on Open Liberty **with as few changes as possible**. Then lock its
observable behavior behind a black-box test suite, record a baseline load test with a plan that will still work
after the migration, and confirm or refute the security findings. Nothing is modernized in this phase.

## Scope

- **Build fixes only.** `mvn install` and `mvn liberty:run` (liberty-maven-plugin 3.3.4) on JDK 11 and Maven 3.9.
  Every change is listed in `docs/MIGRATION.md` with its reason. The build is **unverified** at the time of
  writing: the plan was made by reading the poms, not by running them.
- Base URL `http://localhost:9082/daytrader`, taken from `-Ddaytrader.baseUrl` or `DAYTRADER_BASE_URL`.
- Database: the prebuilt embedded Derby in `daytrader-ee7/resources/data/tradedb`. Each test run starts from a
  fresh copy (the suite never relies on state left by an earlier run).
- **Characterization suite** in `characterization-tests/`: a standalone Maven project (its own `pom.xml`, not a
  module of the legacy build) with JUnit 5, Playwright for Java, `java.net.http.HttpClient` and
  `java.net.http.WebSocket`. Implement every active scenario in the [catalog](characterization-catalog.md).
- **Contracts**, captured from the running app:
  - `legacy-app.md`: every `/app?action=…` (11 actions plus `update_profile`), its parameters, the JSP it renders,
    and the fields the suite reads from it;
  - `marketsummary-websocket.md`: the `/daytrader/marketsummary` endpoint, the `{"action":"update"}` request and
    the JSON pushed back (market summary and recent quote changes), with one recorded example of each;
  - `config.md`: the `/config` actions and parameters (reset, build DB, runtime modes).
- **Baseline load test** in `perf/`:
  - the upstream `daytrader7.jmx` mostly drives JSF pages (`*.faces`), which phase B removes
    ([ADR-0007](../adr/0007-keep-jsp-flow-drop-jsf.md)). Write `perf/daytrader-app.jmx`, which drives **only**
    the `/app` actions (login, home, quotes, buy, portfolio, sell, account, logout) with the same mix of
    requests per user session as the upstream plan;
  - the plan must not need editing in later phases: it already extracts a CSRF token from each form if one is
    present (a no-op on legacy, required from phase F), and it uses only URLs and fields from the
    `legacy-app.md` contract;
  - run **both** plans against `v0-legacy` and store both results. The `/app` plan is the comparison baseline
    for every later phase; the upstream plan's result is kept for reference only;
  - fixed parameters, written into `docs/metrics/README.md`: users, ramp-up, duration, warm-up discarded, JVM
    heap, machine, order processing mode `Sync`;
  - run in non-GUI mode, 3 times, report the median.
- **Security findings** of [ADR-0010](../adr/0010-legacy-security-findings.md): confirm or refute each one on the
  running app and record the result in `MIGRATION.md` in prose, with no exploit steps.
- **Metrics**: `docs/metrics/v0-legacy.json` and the first column of the README metrics table.
- **CI**: a GitHub Actions workflow that builds the EAR, starts Liberty, runs the characterization suite and
  uploads the Playwright traces on failure. From here on, every PR must be green.

## Out of scope

- Upgrading any library, Java version or configuration beyond what the build needs.
- Fixing any security finding (phase F) or any bug found by the suite (documented, not fixed).

## Tasks

1. `build: make legacy build work on Maven 3.9 / JDK 11`: the minimal `pom.xml` and `server.xml` changes.
2. `scripts: build-legacy.ps1, run-legacy.ps1` (fresh Derby copy, start, wait for HTTP 200, print startup time).
3. `test: scaffold characterization-tests project`: pom, `BaseHttpTest` (cookie-aware `HttpClient`, `login()`
   helper, HTML field extraction by label/name), `BaseUiTest` (Playwright browser per class, context per test),
   `MarketSummaryClient` (WebSocket), `TestData` (unique user IDs for `register`).
4. Scenarios, a few per commit, grouped by feature (auth, account, quotes, trading, async orders, market summary,
   admin, misc). Test ids match catalog ids.
5. Capture the three contracts from the running app.
6. Run the suite **three times in a row** against a fresh server to prove it is not flaky.
7. `perf: /app JMeter plan` + `scripts/run-load-test.ps1`; baseline runs with both plans.
8. `docs: security findings` in `MIGRATION.md`.
9. `ci: build, start Liberty, run characterization suite`.
10. `docs: baseline metrics` (`scripts/measure-metrics.ps1 -Label v0-legacy`).

## Acceptance criteria

- [ ] `scripts/build-legacy.ps1` produces `daytrader-ee7/target/daytrader-ee7.ear` on a clean machine with JDK 11 and Maven 3.9.
- [ ] `scripts/run-legacy.ps1` serves `/daytrader/app` with HTTP 200 and prints the startup time.
- [ ] All active catalog scenarios are implemented and pass 3 runs in a row.
- [ ] No test asserts a stock price, an order ID, an account ID or a holding ID by value (see the catalog rules).
- [ ] The suite has no `Thread.sleep`. Async scenarios poll with a timeout.
- [ ] The three contracts exist and contain real recorded examples.
- [ ] `perf/daytrader-app.jmx` runs headless; `docs/metrics/v0-legacy.json` has the median of 3 runs for both plans.
- [ ] Every ADR-0010 finding is marked confirmed or refuted in `MIGRATION.md`.
- [ ] CI runs on every PR and is required by branch protection.

## Risks

| Risk | Mitigation |
|---|---|
| The legacy build or Liberty plugin no longer resolves (old plugin versions, Derby 10.14) | Pin versions, use HTTPS repositories, document each change. Upstream `v1.3` already targets JDK 11 |
| Prices change randomly on every buy and sell, so exact-value assertions are impossible | Invariant-based assertions ([ADR-0004](../adr/0004-black-box-invariant-characterization-tests.md)): deltas computed from the values shown on the confirmation page |
| IDs come from a key table with `allocationSize = 1000`, so they jump | Never assert IDs; find rows by symbol, quantity and status |
| Async order modes complete later, in another transaction | Poll the orders page until the order is `closed`, with a timeout; run async scenarios in their own class with the mode set through `/config` and restored afterwards |
| The market summary is recomputed every 20 s (`@Schedule`), so WebSocket timing varies | Assert message shape and that a push arrives within 30 s, never content timing |
| The `/app` load-test plan does not reproduce the official benchmark | Stated in ADR-0007 and in `docs/metrics/README.md`; both plans are run on `v0-legacy` so the gap is visible |

## Commands

```powershell
.\scripts\build-legacy.ps1
.\scripts\run-legacy.ps1
.\scripts\run-characterization.ps1 -BaseUrl http://localhost:9082/daytrader
.\scripts\run-load-test.ps1 -Plan perf\daytrader-app.jmx -Label v0-legacy
.\scripts\measure-metrics.ps1 -Label v0-legacy
```
