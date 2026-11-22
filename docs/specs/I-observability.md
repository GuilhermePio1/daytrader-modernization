# Spec I – Observability and performance experiments (post-2.0)

| | |
|---|---|
| Status | Post-2.0 (not started before `v2.0.0`) |
| Branch | `phase/i-observability` |
| JDK | 25 |
| Depends on | G |
| Exit tag | `v2.2.0` |

## Goal

Make the running system observable, and use DayTrader's nature as a benchmark to run controlled experiments
with published, reproducible results.

## Scope (outline)

- Micrometer metrics with Prometheus and a Grafana dashboard (orders per second, order latency by mode,
  queue lag), OpenTelemetry tracing across the async order path, structured JSON logs.
- Experiments, each with its own results page under `docs/metrics/experiments/`:
  - platform threads vs. virtual threads on the request path;
  - G1 vs. ZGC (generational) under the `/app` plan;
  - startup with AppCDS / Spring AOT vs. plain jar;
  - optionally, a GraalVM native image.
- Each experiment states its hypothesis before it runs, and reports the result even when the hypothesis fails.
