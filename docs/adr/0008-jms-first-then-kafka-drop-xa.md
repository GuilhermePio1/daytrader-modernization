# ADR-0008: Keep JMS on embedded Artemis in phase C, drop XA, move to Kafka with an outbox after 2.0

- **Status:** Proposed
- **Date:** 2026-11-03
- **Phase:** C (and H)
- **Supersedes / Superseded by:** none

## Context

Order processing has three modes (`TradeConfig.orderProcessingMode`):

- `Sync`: the order is completed in the same transaction as the buy or sell;
- `Async_2-Phase`: `TradeSLSBBean.queueOrder` sends a JMS message to `TradeBrokerQueue` **inside the
  container-managed (JTA) transaction** that created the order; `DTBroker3MDB` completes the order. Whether
  Liberty runs a real two-phase commit between embedded Derby and its JMS engine is confirmed in phase A
  (transaction trace); the code is written to rely on it;
- `Async_ManagedThread`: a thread from `ManagedThreadFactory` runs `CompleteOrderThread`, which **sleeps 500 ms**
  and then completes the order in a new transaction. The sleep is there to let the creating transaction commit
  first, which is a race: under load the thread can read the order before the commit.

Quote price changes are also published to `TradeStreamerTopic` and pushed to WebSocket clients through
`DTStreamer3MDB` and a CDI event.

Spring Boot has no JTA transaction manager by default. Adding one (Narayana or Atomikos) only to keep XA would
carry the application server's main complexity into the new platform.

## Decision

- **Phase C:** keep JMS semantics on an embedded **Apache Artemis** broker with `@JmsListener`. Drop XA. The
  mode is renamed `Async_JMS`: the message is sent **after the database transaction commits**
  (`TransactionSynchronization.afterCommit`) and the listener completes the order in its own transaction.
  `Async_ManagedThread` uses a virtual-thread executor, also triggered after commit, which removes the 500 ms
  sleep and the race.
- **Phase H (post-2.0):** replace the queue with Kafka and a **transactional outbox** (the order and an outbox
  row in one local transaction, a relay publishes, an idempotent consumer completes), which restores the
  "order and message together" guarantee without a distributed transaction.

## Alternatives considered

| Option | Why not |
|---|---|
| Keep XA with Narayana on Spring Boot | Keeps the hardest part of the old platform; few teams run XA on Spring Boot |
| Go straight to Kafka in phase C | Changes the platform and the messaging guarantees in the same phase (ADR-0003) |
| Remove the async modes | They are the most interesting behavior of the application and part of the catalog (CT-26, CT-27) |

## Consequences

- Between C and H there is a known gap: if the process dies after the commit and before the send, the order
  stays `open`. It is documented in `MIGRATION.md`, and a recovery job that completes stale `open` orders is
  considered in C if the gap shows up in tests.
- Catalog edits authorized: CT-27 and CT-30 (mode name). The observable result, an order that ends `closed`, is
  unchanged.
