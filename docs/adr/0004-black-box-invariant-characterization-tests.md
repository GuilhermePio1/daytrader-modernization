# ADR-0004: Black-box, invariant-based characterization tests

- **Status:** Accepted
- **Date:** 2026-11-03
- **Phase:** A
- **Supersedes / Superseded by:** none

## Context

DayTrader has no automated tests. Every internal API (EJB interfaces, JNDI names, servlets, JSPs) is replaced
during the migration, so tests written against internals would be rewritten along with the code and prove
nothing. Two properties of the application make exact assertions impossible:

- every buy and sell moves the quote price by a random factor (`TradeConfig.getRandomPriceChangeFactor`);
- account, holding and order IDs come from a key table with `allocationSize = 1000`, so they jump.

Order completion can also be asynchronous (managed thread or JMS), and the market summary is recomputed every
20 seconds.

## Decision

We will write a standalone suite (`characterization-tests/`, its own Maven project) that talks to the running
application only through HTTP, the browser (Playwright for Java) and the WebSocket endpoint. Assertions are
**relations between values read from the pages** (for example: balance after a buy = balance before −
(quantity × execution price shown on the confirmation + fee)), never literal prices or IDs. Async behavior is
checked by polling with a timeout. The scenarios and rules are in the
[catalog](../specs/characterization-catalog.md).

A scenario may only change if the catalog's "Changed in" column names the phase and an ADR authorizes it.

## Alternatives considered

| Option | Why not |
|---|---|
| Unit tests on `TradeSLSBBean` first | Need the EJB container or heavy mocking of JNDI and JPA; they are added in phase C, when the code is a plain Spring service |
| Arquillian tests inside Liberty | Coupled to the container that is being removed |
| Fixed random seed to make prices deterministic | Requires changing the legacy code before the baseline exists (`TradeConfig` seeds from the clock) |
| Record-and-replay HTTP traffic | Breaks on every random price and generated ID |

## Consequences

- The same suite certifies Liberty, Spring Boot with JSP, and Spring Boot with Thymeleaf.
- Some properties cannot be checked from outside (for example, that passwords are stored hashed). Those are
  covered by integration tests added in the phase that changes them.
- Async scenarios are slower; they run in their own test class.
