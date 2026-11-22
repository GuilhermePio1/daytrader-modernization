# Metrics

One JSON snapshot per exit tag (`v0-legacy.json`, `b-trimmed.json`, `c-boot.json`, …), produced by
`scripts/measure-metrics.ps1` and `scripts/run-load-test.ps1` (written in phase A).

## Rules

- Every number comes from a run that can be repeated with the commands below. No estimates.
- Load tests run 3 times; the snapshot stores all three and the README shows the median.
- Same machine, same parameters, same plan for every tag. If any of them must change, all earlier tags are
  measured again and the change is written here.
- Metrics that get worse are reported, not hidden.

## Load-test parameters

Fixed in phase A and copied here. Until then:

| Parameter | Value |
|---|---|
| Plan | `perf/daytrader-app.jmx` (`/app` actions only; see ADR-0007) |
| Users / ramp-up / duration | TBD |
| Warm-up discarded | TBD |
| Order processing mode | `Sync` |
| JVM heap | TBD |
| Database | Embedded Derby (A–D), PostgreSQL in Docker (E onward) |
| Machine | TBD (CPU, RAM, OS, JDK build) |

## Why the upstream benchmark is not the baseline

The upstream `daytrader7.jmx` drives mostly the JSF UI, which phase B removes. It is run once on `v0-legacy`
for reference; every comparison uses the `/app` plan. See
[ADR-0007](../adr/0007-keep-jsp-flow-drop-jsf.md).

## Snapshot format

```json
{
  "label": "v0-legacy",
  "commit": "643d04c",
  "date": "YYYY-MM-DD",
  "machine": "…",
  "load": { "plan": "perf/daytrader-app.jmx", "runs": [ { "throughput": 0, "p50": 0, "p95": 0, "p99": 0, "errors": 0 } ] },
  "startupSeconds": 0,
  "rssMb": { "idle": 0, "loaded": 0 },
  "artifactMb": 0,
  "cves": { "critical": 0, "high": 0 },
  "characterization": { "passed": 0, "total": 0 },
  "coverage": null,
  "javaClasses": { "application": 0, "benchmarkPrimitivesRemoved": 0 }
}
```
