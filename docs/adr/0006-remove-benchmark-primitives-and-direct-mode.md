# ADR-0006: Remove the benchmark primitives and the Direct JDBC mode

- **Status:** Proposed
- **Date:** 2026-11-03
- **Phase:** B
- **Supersedes / Superseded by:** none

## Context

54 of the 108 Java classes are "primitives" in `web/prims/` and `web/prims/ejb3/`: servlets that each
exercise one Java EE feature (servlet to JSP, session to entity, MDB queue and topic ping, explicit GC, CDI
bean manager lookup, and so on) so that application servers can be compared. They are not part of the trading
application. The application also has two complete implementations of `TradeServices`: the EJB/JPA one
(`TradeSLSBBean`) and a hand-written JDBC one (`TradeDirect`), selected at runtime by `runTimeMode`. Other
settings select features that no longer exist in this build (`accessMode` "WebServices", `cachingType`
"DistributedMap", the `JSP-Images` web interface).

## Decision

We will delete the primitives, `TradeDirect`, `KeySequenceDirect`, the `runTimeMode` switch and the dead
settings in phase B, while still on Liberty, so that the characterization suite proves no user-visible behavior
depended on them.

## Alternatives considered

| Option | Why not |
|---|---|
| Migrate the primitives too | Each one tests a container feature that no longer exists after phase C; migrating them means writing new benchmarks, not modernizing |
| Keep `TradeDirect` and migrate it to `JdbcClient` | Two implementations of every operation double the migration and test effort; the JPA path is the one the default configuration uses |
| Remove them during phase C | Mixes deletions with the re-platform, so a failure could come from either |

## Consequences

- The class count drops by about half before any modernization. The metrics table reports this separately
  ("benchmark primitives removed") so the migration is not credited with it.
- Some upstream documentation pages (`web_prmtv.html`, `docs/`) are removed or rewritten.
- If a comparison against the full upstream benchmark is ever needed, `v0-legacy` and the upstream repository
  remain available.
