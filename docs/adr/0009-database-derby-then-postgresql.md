# ADR-0009: Keep Derby through the re-platform, move to PostgreSQL in phase E

- **Status:** Proposed
- **Date:** 2026-11-03
- **Phase:** C, E
- **Supersedes / Superseded by:** none

## Context

The legacy application runs on embedded Apache Derby 10.14 with a prebuilt database
(`daytrader-ee7/resources/data/tradedb`) and DDL scripts for Derby, DB2 and Oracle. IDs come from a key table
(`KEYGENEJB`) through `@TableGenerator`. Hibernate 6 and later no longer ship `DerbyDialect` in `hibernate-core`;
it lives in `hibernate-community-dialects`.

## Decision

- **Phase C** keeps Derby (with `hibernate-community-dialects`) and the existing schema and key table, so the
  re-platform does not also change the database.
- **Phase E** moves to PostgreSQL with Flyway migrations, replaces the key table with sequences, and removes
  the community dialect.

## Alternatives considered

| Option | Why not |
|---|---|
| Move to PostgreSQL in phase C | Two sources of failure in the largest phase |
| Move to PostgreSQL in phase A or B (still on Liberty) | Changes the legacy baseline that the load test measures |
| H2 instead of Derby during C | Another database change with no lasting value |

## Consequences

- The `c-boot` and `d-web` load-test results are directly comparable with `v0-legacy` (same database).
  The `e-postgres` result shows the database change on its own.
- One extra dependency (`hibernate-community-dialects`) exists for two phases.
