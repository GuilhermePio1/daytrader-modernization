# Specs

One spec per phase. A spec says **what** the phase delivers and **how we know it is done**. The **why**
lives in the [ADRs](../adr/README.md), the **exact interfaces** live in the [contracts](../contracts/README.md),
and **what actually happened** goes in [`../MIGRATION.md`](../MIGRATION.md).

| Spec | Phase | Branch | Size | Scope |
|---|---|---|---|---|
| [00-setup](00-setup.md) | Repository setup | `phase/0-setup` | S | v2.0.0 |
| [A-baseline](A-baseline.md) | Legacy build, characterization suite, baseline load test, CI | `phase/a-baseline` | L | v2.0.0 |
| [B-trim](B-trim.md) | Remove primitives, JSF UI, Direct mode (still on Liberty) | `phase/b-trim` | M | v2.0.0 |
| [C-boot-core](C-boot-core.md) | Spring Boot 4.1 / Java 25, JMS kept on embedded Artemis | `phase/c-boot-core` | L | v2.0.0 |
| [D-web](D-web.md) | Thymeleaf, Spring WebSocket, executable jar | `phase/d-web` | M | v2.0.0 |
| [E-data](E-data.md) | PostgreSQL, Flyway, Testcontainers, money types | `phase/e-data` | M | v2.0.0 |
| [F-security](F-security.md) | Spring Security, BCrypt, admin role, no PII in logs | `phase/f-security` | M | v2.0.0 |
| [G-release](G-release.md) | Container, CI hardening, final metrics, docs | `phase/g-release` | S | v2.0.0 |
| [H-outbox-kafka](H-outbox-kafka.md) | Kafka + transactional outbox, REST API | `phase/h-outbox-kafka` | L | post-2.0 |
| [I-observability](I-observability.md) | Metrics, tracing, performance experiments | `phase/i-observability` | M | post-2.0 |
| [characterization-catalog](characterization-catalog.md) | The behavioral contract | – | – | – |

Sizes are relative (S < M < L), not estimates in days.

**Rule for the v2.0.0 line:** nothing from H or I is started before `v2.0.0` is tagged. If a post-2.0 idea
comes up during an earlier phase, it is written down in the H or I spec and left there.

## Spec template

Every spec uses the same sections:

- **Header**: status, branch, JDK, dependencies, related ADRs and contracts, exit tag.
- **Goal**: one paragraph.
- **Scope / Out of scope**.
- **Tasks**: ordered, each small enough for one commit.
- **Acceptance criteria**: checkboxes, copied into the PR description.
- **Risks**: each with a mitigation.
- **Commands**: how to build, run and verify.

Specs for later phases are shorter on purpose. Each one is detailed (tasks and commands) when the phase before
it is done, because what that phase finds changes the plan.

Status values: `Planned`, `In progress`, `Done`.
