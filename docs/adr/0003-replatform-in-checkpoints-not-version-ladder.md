# ADR-0003: Re-platform in behavior-preserving checkpoints, not a version ladder

- **Status:** Accepted
- **Date:** 2026-11-03
- **Phase:** 0
- **Supersedes / Superseded by:** none

## Context

When an application is already on Spring, the safe path is a version ladder: Spring 4 → Boot 2 → Boot 3 →
Boot 4, each step a runnable checkpoint. DayTrader is not on Spring. Its services come from the application
server: EJB container, JTA, JMS provider, timer service, managed executors, JNDI, JSR-356. There is no
intermediate version of "DayTrader on Spring" to climb through. Upgrading it inside Jakarta EE (8, then 10) is
what IBM's own ports already did, and it would not remove the server.

## Decision

We will migrate through checkpoints defined by **what changes**, each one runnable and verified by the same
characterization suite and the same load-test plan:

1. **B – Trim, still on Liberty:** remove what only exists to benchmark the server, so the suite proves the
   removal is invisible.
2. **C – Swap the provider of services:** Spring replaces the container, but the semantics stay (JMS queue and
   topic on embedded Artemis, the same transactions, the same tables). JSP stays.
3. **D – Swap the web layer:** Thymeleaf and Spring WebSocket behind frozen URLs and messages.
4. **E – Swap the database:** PostgreSQL and Flyway.
5. **F – Change behavior on purpose:** security fixes, each one authorized here and in ADR-0010.
6. **H (post-2.0) – Change the guarantees on purpose:** outbox and Kafka instead of JMS.

Inside each phase: one concern per commit, build and suite green before the next. OpenRewrite is used where
recipes exist (`javax` → `jakarta`), and its output is committed and logged separately from manual work.

## Alternatives considered

| Option | Why not |
|---|---|
| Version ladder on Jakarta EE (8 → 10 → 11 on Liberty), then Spring | Repeats IBM's ports and doubles the work; the server is still there at the end |
| Big-bang rewrite on Spring Boot | Loses the behavioral baseline; shows rewriting, not modernizing |
| Strangler with a proxy (old and new apps side by side, routing per URL) | Both apps would share the Derby database and JMS destinations, which needs more infrastructure than the code base justifies. Reconsidered if phase C proves too large for one PR |
| Re-platform and change the web layer in one phase | Two unknowns at once; when the suite fails, the cause is unclear |

## Consequences

- Phase C is the largest single step. If its PR becomes too large to review, it is split by service
  (trading, market summary, async orders), each merged with the suite green.
- Every exit tag is runnable, so `git bisect` works across the migration, and the metrics show the curve, not
  only the end state.
