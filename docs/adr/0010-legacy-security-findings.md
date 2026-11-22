# ADR-0010: Legacy security and correctness findings, confirmed in A, fixed in C, E and F

- **Status:** Proposed
- **Date:** 2026-11-03
- **Phase:** A (confirm), C, E and F (fix)
- **Supersedes / Superseded by:** none

## Context

Reading the `v0-legacy` code shows the following. Each one is a **finding to confirm** on the running
application in phase A; none is fixed before phase E or F, so the baseline stays the real legacy behavior.
This document records the findings in prose only, with no exploit steps.

| ID | Finding | Where (in `v0-legacy`) |
|---|---|---|
| S-01 | Passwords stored in plain text | `AccountProfileDataBean.passwd`, column `passwd` |
| S-02 | The user's password is written to the trace log on login | `TradeSLSBBean.login` (`Log.trace("…login(" + userID + "," + password + ")…")`) |
| S-03 | `toString()` and the HTML dump of the profile include the password and the credit card number | `AccountProfileDataBean.toString`, `toHTML` |
| S-04 | The full credit card number is shown on the account page and stored in plain text | `account.jsp`, column `creditcard` |
| S-05 | The admin page `/config` (reset statistics, rebuild tables, populate the database, change runtime modes) is reachable without an admin role | `TradeConfigServlet`; `web.xml` declares security constraints, but `server.xml` enables no application-security feature, so they are likely not enforced |
| S-06 | Login is checked by the application, not by a security framework; no CSRF protection on forms | `TradeAppServlet`, `TradeServletAction` |
| S-07 | Database credentials in plain text in `server.xml` | `authData` elements |
| S-08 | To check: selling a holding by ID may not verify that the holding belongs to the logged-in user | `TradeSLSBBean.sell(userID, holdingID, …)`; catalog CT-25 |
| R-01 | Money constants built from `double` (`new BigDecimal(0.01)`, `new BigDecimal(1.0)`), deprecated rounding constants | `TradeConfig`, `FinancialUtils` |
| R-02 | `Async_ManagedThread` waits 500 ms for the creating transaction to commit; a race under load | `CompleteOrderThread` (see ADR-0008) |
| R-03 | Mutable static configuration shared by all requests and threads | `TradeConfig` static fields |

## Decision

- Phase A confirms or refutes each finding on the running application and writes the result in
  `MIGRATION.md`.
- Fixes: S-01 to S-06 and S-08 in **phase F**; S-07 in **phase C** (`server.xml` is removed with Liberty;
  datasource credentials come from configuration and environment variables); R-01 in **phase E**; R-02 and R-03
  in **phase C** (ADR-0008, `@ConfigurationProperties`).
- Visible behavior changes authorized by this ADR, and only these: CT-09 (masked card), CT-25 (if S-08 is
  confirmed), CT-28 and CT-29 (`/config` requires `ADMIN`).
- Existing plain-text passwords are migrated with upgrade-on-login (BCrypt through
  `DelegatingPasswordEncoder`), so seeded and load-test users keep working.

## Alternatives considered

| Option | Why not |
|---|---|
| Fix everything in phase A | The baseline would no longer be the legacy application |
| Fix each finding in the phase that touches the code | Security changes spread over many PRs are harder to review and to explain |
| Force a password reset for all users | Breaks the seeded users and the load test; upgrade-on-login reaches the same state |

## Consequences

- The README gets a "security findings" section with each finding and its status, which is a clear
  before/after for reviewers.
- BCrypt makes login slower; the `f-secure` load-test result shows by how much.
