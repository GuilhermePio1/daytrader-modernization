# ADR-0005: Target Java 25 and Spring Boot 4.1

- **Status:** Accepted
- **Date:** 2026-11-03
- **Phase:** 0
- **Supersedes / Superseded by:** none

## Context

The target has to be a current, supported stack that a team would choose today for a new service of this kind.
DayTrader's async order paths use threads (`ManagedThreadFactory`) and blocking I/O, which makes virtual threads
relevant.

## Decision

We will target **Java 25** (LTS, September 2025) and **Spring Boot 4.1** (Spring Framework 7, Jakarta EE 11
APIs, Hibernate 7, Jackson 3). If a newer Boot 4.x line is current when phase C starts, we use it and update this
ADR's title through a superseding ADR.

## Alternatives considered

| Option | Why not |
|---|---|
| Java 21 | Still LTS, but Java 24's JEP 491 (virtual threads no longer pinned by `synchronized`) matters for this code base, which uses `synchronized` in its utilities |
| Jakarta EE 11 on Open Liberty | Keeps the application server; already done in spirit by IBM's daytrader10 port |
| Quarkus | A valid target; Spring Boot is chosen because it is what most Java job descriptions in the target market ask for |

## Consequences

- Phase C needs JDK 25 locally and in CI; phase A and B still need JDK 11 for the legacy build.
- Experiments in phase I (virtual threads, GC, CDS) run on the same JDK as production.
