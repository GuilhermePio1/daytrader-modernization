# Spec C – Re-platform the core to Spring Boot 4.1 / Java 25

| | |
|---|---|
| Status | Planned |
| Branch | `phase/c-boot-core` |
| JDK | 25 |
| Depends on | B |
| ADRs | [0003](../adr/0003-replatform-in-checkpoints-not-version-ladder.md), [0005](../adr/0005-target-stack-java25-boot4.md), [0008](../adr/0008-jms-first-then-kafka-drop-xa.md), [0009](../adr/0009-database-derby-then-postgresql.md), [0010](../adr/0010-legacy-security-findings.md) (S-07, R-02, R-03) |
| Exit tag | `c-boot` |

## Goal

Remove the application server. The same business code runs in a Spring Boot application, with the same JMS
semantics (queue for orders, topic for quote changes) on an embedded broker, so the only thing that changes in
this phase is **who provides the services**, not how the application behaves. JSP stays for now (phase D).

## Scope

- One Maven module `daytrader-app` replaces the EAR, EJB and WAR modules (`war` packaging while JSP exists,
  runnable with `java -jar`). Maven Wrapper.
- `javax.*` → `jakarta.*` with OpenRewrite (`org.openrewrite.java.migrate.jakarta.JavaxMigrationToJakarta`);
  its output is committed separately from manual changes and summarized in `MIGRATION.md`.
- Mapping, one concern per commit:

| Legacy | Spring |
|---|---|
| `TradeSLSBBean` `@Stateless`, `@TransactionAttribute` | `@Service`, `@Transactional` with the same propagation (`REQUIRED`, `REQUIRES_NEW`, `NOT_SUPPORTED`) |
| JNDI lookups, `@EJB`, `TradeAction` indirection | Constructor injection |
| `DTBroker3MDB` on `TradeBrokerQueue` | `@JmsListener` on embedded Artemis |
| `DTStreamer3MDB` on `TradeStreamerTopic` → CDI `@Observes` event | `@JmsListener` (topic) → Spring `ApplicationEvent` |
| `MarketSummarySingleton` `@Schedule(every 20 s)`, `@Lock(READ/WRITE)` | `@Scheduled(fixedRate)`, immutable snapshot in an `AtomicReference` |
| `ManagedThreadFactory` (`Async_ManagedThread`) | Virtual-thread `TaskExecutor` |
| `Async_2-Phase` (XA between Derby and JMS) | `Async_JMS`: local transactions, send after commit (ADR-0008) |
| Static mutable `TradeConfig` + `daytrader.properties` | `@ConfigurationProperties`; `/config` changes a runtime settings bean |
| `persistence.xml`, `@TableGenerator` | Spring Data JPA config, same tables and key table (changed in E) |
| `Log` wrapper | SLF4J |
| `web.xml`, `@WebServlet`, `@WebFilter` | Servlet and filter registration beans, same URLs under `/daytrader` |

- Unit tests (JUnit 5, Mockito) for the trading service: buy and sell balance math, fee, holding creation and
  removal, order state transitions. The first automated tests this code has had.
- `@DataJpaTest` for the entities and named queries.

## Out of scope

- JSP → Thymeleaf, WebSocket stack, executable jar (D). PostgreSQL (E). Security (F). Kafka (H).

## Acceptance criteria

- [ ] `./mvnw verify` green on JDK 25. No `javax.ejb`, `javax.jms`, `javax.faces` and no Liberty files remain.
- [ ] The characterization suite passes, with only the CT-27/CT-30 edits authorized by ADR-0008.
- [ ] `/app` load-test result stored as `docs/metrics/c-boot.json` (same parameters as the baseline).
- [ ] `MIGRATION.md` says what OpenRewrite changed and what was done by hand.

## Risks

| Risk | Mitigation |
|---|---|
| Hibernate 7 no longer ships `DerbyDialect` in core | `hibernate-community-dialects` for this phase only (ADR-0009) |
| Transaction semantics differ between EJB and Spring (rollback on checked exceptions, self-invocation) | Unit tests on rollback cases; `rollbackFor` where the EJB rolled back; no self-invocation of `@Transactional` methods |
| Sending a JMS message before the DB commit was XA-safe before; without XA the consumer can see uncommitted data | Send after commit (`TransactionSynchronization.afterCommit`), consumer re-reads the order; the remaining gap (crash between commit and send) is documented and closed in H |
| Spring Boot 4 support for embedded Artemis | Check the starter at phase start; fallback is an Artemis container started by Docker Compose |
| JSP inside Spring Boot | `war` packaging and `tomcat-embed-jasper` until D |
