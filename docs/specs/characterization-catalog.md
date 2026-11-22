# Characterization catalog

The behavior that every phase must preserve, written as black-box scenarios against the running application.
The suite lives in `characterization-tests/` (phase A) and runs **unchanged** from `a-safety-net` to `v2.0.0`.
The only edits allowed later are the ones listed in the "Changed in" column, each authorized by an ADR.

Why black-box and invariant-based: [ADR-0004](../adr/0004-black-box-invariant-characterization-tests.md).

## Rules

1. **No value that DayTrader randomizes or generates is asserted literally.** Every buy and sell moves the
   quote price by a random factor, and IDs come from a key table with `allocationSize = 1000`. Assertions use
   values read from the page under test (the order confirmation shows the execution price and fee) and check
   relations between them.
2. **Money** is compared as `BigDecimal` at scale 2, `HALF_UP`, which is what the legacy code uses
   (`FinancialUtils`). The order fee is `24.95` for buy and sell (`TradeConfig.getOrderFee`).
3. **Isolation**: each scenario that changes data registers its own user (`TestData.newUserId()`), except the
   login scenarios that use the seeded `uid:0` / `xxx`. No scenario depends on another one's leftovers.
4. **Locators**: labels, field names, visible text and ids only. No positional XPath, no CSS classes. The markup
   changes completely in phase D (JSP → Thymeleaf).
5. **Timing**: no `Thread.sleep`. Async scenarios poll with a timeout (30 s) and fail with the last observed
   state.
6. **Mode changes** (`/config`) are made in the scenario's setup and restored in its teardown, in a test class
   that does not run in parallel with others.

## Scenarios

`Changed in` is empty when the behavior must stay exactly the same up to `v2.0.0`.

### Authentication and registration

| ID | Scenario | Assertion | Changed in |
|---|---|---|---|
| CT-01 | Log in as `uid:0` / `xxx` | Home page shows the user ID and an account balance | – (must still pass after BCrypt, see F) |
| CT-02 | Log in with a wrong password | Stays on the welcome page with an error message; no session is created | – |
| CT-03 | Log in with an unknown user ID | Same as CT-02 | – |
| CT-04 | Log out | Welcome page; the next `/app?action=home` does not show account data | – |
| CT-05 | `/app?action=portfolio` without logging in | Welcome page, no portfolio data | – |
| CT-06 | Register a new user with an opening balance | Logged in; balance = opening balance; open balance = opening balance | – |
| CT-07 | Register with `password` ≠ `cpassword` | Error message; logging in with the new ID fails | – |
| CT-08 | Register an existing user ID | Error message; the existing account is unchanged | – |

### Account

| ID | Scenario | Assertion | Changed in |
|---|---|---|---|
| CT-09 | Open the account page | Shows full name, address, e-mail, credit card, balance, open balance, login and logout counts | F (credit card masked, ADR-0010) |
| CT-10 | Update the profile (`update_profile`) | New values shown; still there after log out and log in | – |
| CT-11 | Log in twice | Login count increases by exactly 1 per login; logout count by 1 per logout | – |

### Quotes and market summary

| ID | Scenario | Assertion | Changed in |
|---|---|---|---|
| CT-12 | Quote `s:0` | One row: symbol, company name, price > 0, volume ≥ 0 | – |
| CT-13 | Quote `s:0,s:1,s:2` | Three rows, one per symbol, in the requested order | – |
| CT-14 | Quote a symbol that does not exist | Behavior captured in phase A (error message or empty row) | – |
| CT-15 | Market summary page (`mksummary`) | Shows the index (TSIA), its change, volume, and top gainers and losers; captured counts. The content is filled in by the page's WebSocket client, so this is a Playwright scenario | – |
| CT-16 | WebSocket `/marketsummary`: send `{"action":"update"}` | A JSON message with the market summary fields arrives within 30 s | – (implementation changes in D; assertions do not) |
| CT-17 | WebSocket: buy a symbol while connected | A recent-quote-change message for that symbol arrives within 30 s | – (as CT-16) |

### Trading, synchronous mode (default)

| ID | Scenario | Assertion | Changed in |
|---|---|---|---|
| CT-18 | Buy `N` shares of a symbol | Confirmation shows type `buy`, quantity `N`, price `P`, fee `24.95`; balance after = before − (N × P + 24.95) | – |
| CT-19 | After CT-18, open the portfolio | One new holding for that symbol with quantity `N` and purchase price `P` | – |
| CT-20 | Sell that holding | Confirmation shows type `sell`, price `P'`; holding gone; balance after = before + (N × P' − 24.95) | – |
| CT-21 | Account orders list after CT-18 and CT-20 | Both orders listed with status `closed` | – |
| CT-22 | Completed-order alert | After a completed order, the next `/app` page shows the alert once; the page after that does not | – |
| CT-23 | Portfolio totals | Sum of holding market values equals the displayed total; gain = current value − purchase value (scale 2) | – |
| CT-24 | Buy with quantity `0`, negative, and not a number | Behavior captured in phase A; balance unchanged | – |
| CT-25 | Sell a holding ID that belongs to another user | Behavior captured in phase A. If the sale succeeds, it becomes finding S-08 | F, only if S-08 is confirmed |

### Trading, asynchronous modes

| ID | Scenario | Assertion | Changed in |
|---|---|---|---|
| CT-26 | Mode `Async_ManagedThread`: buy | Order eventually `closed` (≤ 30 s); balance delta as in CT-18 | – (implementation changes in C; assertions do not) |
| CT-27 | Mode `Async_2-Phase` (JMS queue + MDB): buy | Order eventually `closed` (≤ 30 s); balance delta as in CT-18 | C (mode renamed `Async_JMS`, no XA, ADR-0008); H (Kafka + outbox) |

### Administration

| ID | Scenario | Assertion | Changed in |
|---|---|---|---|
| CT-28 | `/config`, reset trade statistics (`resetTrade`) | Run statistics page with counts | F (admin role required) |
| CT-29 | `/config` as an anonymous user | Phase A captures the legacy behavior (expected: reachable) | F (redirect to login / 403, ADR-0010) |
| CT-30 | `/config` changes the order processing mode | Next order uses the new mode (observable through CT-26/27 timing of `open` → `closed`) | C (mode list changes, ADR-0008) |

### Miscellaneous

| ID | Scenario | Assertion | Changed in |
|---|---|---|---|
| CT-31 | `/app?action=<unknown>` while logged in | Behavior captured in phase A | – |
| CT-32 | Home page | Shows holdings count, balance, and the embedded market summary | – |

## Captured values

Filled in during phase A from the running `v0-legacy` application.

| Item | Value |
|---|---|
| Error text, wrong password (CT-02) | TBD |
| Behavior, unknown symbol (CT-14) | TBD |
| Number of top gainers / losers (CT-15) | TBD |
| Behavior, invalid quantities (CT-24) | TBD |
| Behavior, selling another user's holding (CT-25) | TBD |
| `/config` anonymous access (CT-29) | TBD |
| Behavior, unknown action (CT-31) | TBD |
| WebSocket message examples (CT-16, CT-17) | See `docs/contracts/marketsummary-websocket.md` |
