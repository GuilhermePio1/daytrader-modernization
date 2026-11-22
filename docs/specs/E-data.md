# Spec E – PostgreSQL, Flyway, Testcontainers, money types

| | |
|---|---|
| Status | Planned |
| Branch | `phase/e-data` |
| JDK | 25 |
| Depends on | D |
| ADRs | [0009](../adr/0009-database-derby-then-postgresql.md), [0010](../adr/0010-legacy-security-findings.md) |
| Exit tag | `e-postgres` |

## Goal

Move from embedded Derby to PostgreSQL with a versioned schema, and fix the money and quantity types, without
changing what the user sees.

## Scope

- Flyway `V1__baseline.sql` derived from `dbscripts/derby/Table.ddl`, translated to PostgreSQL; `V2__seed.sql`
  or the existing build-DB action for test data. `hibernate-community-dialects` removed.
- Key table `KEYGENEJB` with `@TableGenerator` → PostgreSQL sequences (`allocationSize` kept, so the
  application behaves the same under load).
- Money: constants built with `new BigDecimal(double)` (for example `new BigDecimal(0.01)`) → string or
  `valueOf` constructors; `RoundingMode.HALF_UP` instead of the deprecated `BigDecimal.ROUND_HALF_UP`.
  `quantity` stays `double` in the API (fractional shares are a legacy feature) but is stored as `NUMERIC`.
- Docker Compose for local PostgreSQL; Testcontainers for integration tests.
- The characterization suite runs against the app backed by PostgreSQL.

## Acceptance criteria

- [ ] `./mvnw verify` runs integration tests against PostgreSQL in Testcontainers.
- [ ] Flyway validates on startup; `ddl-auto` is `validate`.
- [ ] The characterization suite passes unchanged.
- [ ] Money rounding unit tests: buy and sell totals equal the legacy results for a table of inputs captured
      from `v0-legacy` behavior.

## Risks

| Risk | Mitigation |
|---|---|
| Changing `BigDecimal` constructors changes results by a fraction of a cent | Golden-value unit tests written before the change; any difference is documented, not hidden |
| Derby and PostgreSQL differ on types, case and timestamp precision | Compare schema with a script; CT scenarios cover the visible fields |
