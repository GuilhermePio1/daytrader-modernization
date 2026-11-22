# Architecture Decision Records

Format and rules: [ADR-0001](0001-record-architecture-decisions.md). Template: [0000](0000-template.md).

| ADR | Title | Status | Phase |
|---|---|---|---|
| [0001](0001-record-architecture-decisions.md) | Record architecture decisions | Accepted | 0 |
| [0002](0002-choose-daytrader-as-legacy-base.md) | Use DayTrader 7 as the legacy code base | Accepted | 0 |
| [0003](0003-replatform-in-checkpoints-not-version-ladder.md) | Re-platform in behavior-preserving checkpoints, not a version ladder | Accepted | 0 |
| [0004](0004-black-box-invariant-characterization-tests.md) | Black-box, invariant-based characterization tests | Accepted | A |
| [0005](0005-target-stack-java25-boot4.md) | Target Java 25 and Spring Boot 4.1 | Accepted | 0 |
| [0006](0006-remove-benchmark-primitives-and-direct-mode.md) | Remove the benchmark primitives and the Direct JDBC mode | Proposed | B |
| [0007](0007-keep-jsp-flow-drop-jsf.md) | Keep the JSP flow (`/app`) as the only UI and remove the JSF UI | Proposed | B |
| [0008](0008-jms-first-then-kafka-drop-xa.md) | Keep JMS on embedded Artemis in C, drop XA, Kafka with an outbox after 2.0 | Proposed | C, H |
| [0009](0009-database-derby-then-postgresql.md) | Keep Derby through the re-platform, PostgreSQL in E | Proposed | C, E |
| [0010](0010-legacy-security-findings.md) | Legacy security and correctness findings, confirmed in A, fixed in C, E and F | Proposed | A, C, E, F |
