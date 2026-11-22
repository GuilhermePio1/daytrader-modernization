# Spec H – Kafka and a transactional outbox replace JMS and two-phase commit (post-2.0)

| | |
|---|---|
| Status | Post-2.0 (not started before `v2.0.0`) |
| Branch | `phase/h-outbox-kafka` |
| JDK | 25 |
| Depends on | G |
| ADRs | [0008](../adr/0008-jms-first-then-kafka-drop-xa.md) |
| Exit tag | `v2.1.0` |

## Goal

Give asynchronous order processing the guarantee the legacy XA mode gave (the order and its message are
committed together or not at all) without a distributed transaction, and expose a versioned REST API.

## Scope (outline)

- Transactional outbox: the order row and an `outbox` row are written in the same local transaction; a relay
  publishes to Kafka. Idempotent consumer keyed by order ID completes the order.
- Quote price changes published as Kafka events; the WebSocket layer consumes them.
- `Async_JMS` mode → `Async_Outbox`; Artemis removed.
- `/api/v1` REST API (quotes, portfolio, orders) with an OpenAPI contract written first.
- Failure tests: kill the app between commit and publish and prove that the order still completes.

## Why post-2.0

This is the strongest single result of the project ("replaced XA two-phase commit with a transactional outbox"),
and also the largest. It is scheduled after `v2.0.0` so that the committed scope can be finished first.
