# DayTrader – Java EE to Spring Boot Modernization

> Taking IBM's **DayTrader 7** benchmark (Java EE 7, EJB, MDB, JMS, JSF, EAR on WebSphere Liberty) to
> **Java 25 + Spring Boot 4.1**, step by step, without breaking its behavior, and measuring every step with
> the same load test.

[DayTrader](https://github.com/WASdev/sample.daytrader7) is the end-to-end benchmark IBM uses to compare
Java EE application servers. It models an online stock brokerage: users log in, look up quotes, buy and sell
shares, and watch a live market summary. Inside it uses most of Java EE: stateless and singleton session beans,
message-driven beans on a JMS queue and topic, two-phase commit between the database and JMS, JPA, JSP, JSF,
CDI and WebSockets.

IBM maintains ports of DayTrader to newer Jakarta EE versions on Liberty
([daytrader8](https://github.com/OpenLiberty/sample.daytrader8),
[daytrader10](https://github.com/OpenLiberty/sample.daytrader10)). **No version runs on Spring Boot, as an
executable jar, without an application server.** That is what this repository builds, as a series of small,
reviewable phases.

This repository keeps the full upstream history (tag [`v0-legacy`](../../tree/v0-legacy)). Each phase is one
pull request, one spec and one or more ADRs. A black-box **characterization test suite**, written before any
change, runs unchanged against every phase. It is the proof that the behavior was preserved.

## Why this project

This is the kind of system that still runs in banks and insurers: an EAR with EJBs and MDBs on an IBM
application server, configured through JNDI and XML, with no automated tests. Modernizing it means:

- **making the legacy build and server run again** before touching anything;
- **pinning down current behavior** with tests that do not depend on the internals about to be replaced,
  including a load test whose numbers can be compared before and after;
- **re-platforming without a big bang**: removing the application server while keeping the JMS semantics,
  then replacing what no longer fits (JSF, XA two-phase commit) with explicit trade-offs;
- **paying security debt**: plain-text passwords, passwords and card numbers in logs and `toString()`,
  an unprotected admin page that can wipe the database;
- leaving the system **operable**: container image, CI, health checks, metrics.

## Status

The **v2.0.0 line** marks the end of the committed scope. Phases below it are planned but are not needed to
call the project done. (Tags `v1.0`–`v1.4` belong to IBM's upstream history, so this project's release starts
at `v2.0.0`.)

| Phase | Description | Spec | Status | Exit tag |
|---|---|---|---|---|
| 0 | Repository setup | [00-setup](docs/specs/00-setup.md) | 🟢 Done | `v0-legacy` |
| A | Legacy build on Liberty, characterization suite, baseline load test, CI | [A-baseline](docs/specs/A-baseline.md) | ⚪ Planned | `a-safety-net` |
| B | Trim on Liberty: remove benchmark primitives, JSF UI, Direct JDBC mode | [B-trim](docs/specs/B-trim.md) | ⚪ Planned | `b-trimmed` |
| C | Re-platform the core to Spring Boot 4.1 / Java 25, JMS kept (embedded Artemis) | [C-boot-core](docs/specs/C-boot-core.md) | ⚪ Planned | `c-boot` |
| D | Web layer: JSP → Thymeleaf, Spring WebSocket, executable jar | [D-web](docs/specs/D-web.md) | ⚪ Planned | `d-web` |
| E | PostgreSQL, Flyway, Testcontainers, money types | [E-data](docs/specs/E-data.md) | ⚪ Planned | `e-postgres` |
| F | Security: Spring Security, BCrypt, protected admin, no PII in logs | [F-security](docs/specs/F-security.md) | ⚪ Planned | `f-secure` |
| G | Release: container image, CI hardening, final metrics, docs | [G-release](docs/specs/G-release.md) | ⚪ Planned | **`v2.0.0`** |
| | **— v2.0.0 line —** | | | |
| H | Kafka + transactional outbox replaces JMS and two-phase commit; REST API | [H-outbox-kafka](docs/specs/H-outbox-kafka.md) | ⚪ Post-2.0 | `v2.1.0` |
| I | Observability and performance experiments (virtual threads, GC, CDS) | [I-observability](docs/specs/I-observability.md) | ⚪ Post-2.0 | `v2.2.0` |

Legend: ⚪ Planned · 🟡 In progress · 🟢 Done

## Before / after

| Concern | Legacy (`v0-legacy`) | Target (`v2.0.0`) |
|---|---|---|
| Runtime | Java 8 source level, EAR on WebSphere Liberty (Open Liberty) | Java 25, executable jar, container image |
| Business logic | `@Stateless` session bean, container-managed transactions | Spring `@Service`, `@Transactional` |
| Async orders | MDB on a JMS queue, optional XA two-phase commit | `@JmsListener` on embedded Artemis, local transactions (Kafka + outbox in H) |
| Market summary | `@Singleton` + `@Schedule` + `@Lock`, MDB on a JMS topic → CDI event → JSR-356 WebSocket | `@Scheduled`, Spring WebSocket, same message format |
| Configuration | JNDI, `web.xml`, `ejb-jar.xml`, `server.xml`, static mutable `TradeConfig` | `application.yml`, typed `@ConfigurationProperties` |
| Web | Servlet + JSP, plus a second JSF UI | Thymeleaf (one UI) |
| Persistence | JPA 2.1, embedded Derby, `@TableGenerator` key table | Hibernate 7, PostgreSQL, Flyway, sequences |
| Security | Plain-text passwords, app-level login, admin page not protected | Spring Security, BCrypt with upgrade-on-login, admin role |
| Tests | None (one manual `TestServlet`) | JUnit 5, Mockito, Testcontainers, characterization suite |
| Delivery | Copy the EAR into a Liberty server | Docker image, GitHub Actions, Trivy, Dependabot |

## Metrics

Measured with the same `/app` JMeter plan on the same machine at every exit tag. Snapshots go to
[`docs/metrics/`](docs/metrics/). No estimated numbers: a cell stays `TBD` until a measurement exists.

| Metric | `v0-legacy` | `c-boot` | `v2.0.0` |
|---|---|---|---|
| Throughput at fixed load, `/app` plan (req/s) | TBD | TBD | TBD |
| Latency p50 / p95 / p99 (ms) | TBD | TBD | TBD |
| Startup time to first 200 (s) | TBD | TBD | TBD |
| Resident memory, idle / under load (MB) | TBD | TBD | TBD |
| Deployable size: EAR + server vs. jar / image (MB) | TBD | TBD | TBD |
| Known CVEs, critical / high | TBD | TBD | TBD |
| Characterization scenarios passing | TBD | TBD | TBD |
| Line coverage, unit + integration | TBD | TBD | TBD |
| Java classes, application code only | TBD | TBD | TBD |
| Java classes removed as benchmark primitives (not modernization) | – | TBD | TBD |

The last row exists on purpose: about half of the legacy classes are benchmark primitives (`web/prims/`) that
are removed in phase B. They are counted separately so that the class count is not credited to the migration.

## Running it

### Legacy version (phases 0 – B)

Requirements: JDK 11, Maven 3.9. **Not verified yet**: phase A makes this work and records what it took.

```powershell
mvn install
cd daytrader-ee7
mvn liberty:run          # → http://localhost:9082/daytrader
```

> On Windows, clone the repository to a short path (for example `C:\dev\daytrader-modernization`). Source paths
> are up to 124 characters long and the build fails beyond the 260-character Windows limit.

### Modern version (from phase C)

_Instructions are added as the phases land._

## How the work is documented

- **[Specs](docs/specs/)**: what each phase does, its acceptance criteria and its risks.
- **[ADRs](docs/adr/)**: why each significant decision was made.
- **[Characterization catalog](docs/specs/characterization-catalog.md)**: the behavior that must never break,
  and the behavior that is changed on purpose (with the ADR that allows it).
- **[Contracts](docs/contracts/)**: the frozen legacy surface (`/app` actions, the `/marketsummary` WebSocket
  messages), captured in phase A.
- **[Migration journal](docs/MIGRATION.md)**: what actually happened, including what OpenRewrite automated and
  what was fixed by hand.

## Credits and license

DayTrader 7 is © IBM Corporation and builds on the Apache Geronimo DayTrader project (© Apache Software
Foundation). The upstream README is kept in [`docs/legacy/ORIGINAL_README.md`](docs/legacy/ORIGINAL_README.md).

This derivative work is distributed under the same **Apache License 2.0** ([`LICENSE`](LICENSE)). The upstream
`NOTICE` files are kept, and files changed in this repository say so in their header, as section 4 of the
license requires.
