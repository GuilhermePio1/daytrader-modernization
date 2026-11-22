# ADR-0002: Use DayTrader 7 as the legacy code base

- **Status:** Accepted
- **Date:** 2026-11-03
- **Phase:** 0
- **Supersedes / Superseded by:** none

## Context

The goal is a public modernization project that resembles the systems that still run in banks and insurers:
Java EE on an IBM application server, EJBs and MDBs, JNDI configuration, no tests. Proprietary code cannot be
published, so the base has to be open source, recognizable, small enough to finish, and runnable on a laptop.

Candidates looked at (November 2026):

| Candidate | Notes |
|---|---|
| **WASdev/sample.daytrader7** | Apache-2.0. Java EE 7: EJB, MDB, JMS queue and topic, JTA, JPA, JSP, JSF, CDI, WebSocket. 108 Java classes, about half of them benchmark primitives. Ships JMeter plans. IBM's own benchmark for Java EE servers. No Spring Boot version exists; official ports (daytrader8, daytrader10) stay on Liberty |
| Tudu Lists (Spring 3, 2013) | GPL-3.0, smaller, already planned in another repository. Spring-to-Spring upgrade; a to-do domain |
| OpenQuote / OpenUnderwriter | Insurance domain, EJB and Liferay portlets on JBoss 7.1. GPL-2.0, thousands of classes across six versions, SVN replica; getting the legacy build to run is a project of its own |
| JForum 2 | Archived 2020, forum domain, unclear license metadata |
| jPetStore 6, Cargo Tracker, Spring PetClinic, Apache Roller | Actively maintained or already modern |
| AppFuse, Mifos "head" | A framework, not an application; and a 500 MB code base |

## Decision

We will use **DayTrader 7** at upstream commit `643d04c` (tag `v0-legacy`) as the legacy base.

## Alternatives considered

See the table above. Tudu Lists was the closest alternative; it modernizes Spring to Spring, while DayTrader
requires removing an application server, which is the harder and more common real-world case.

## Consequences

- The project uses the full range of Java EE services, so each replacement (EJB, MDB, JTA, `@Schedule`,
  JSR-356) has to be decided and documented.
- Because DayTrader is a benchmark, before/after performance can be measured with a workload that was designed
  for that purpose (with the limits stated in [ADR-0007](0007-keep-jsp-flow-drop-jsf.md)).
- About half the code is benchmark scaffolding that is removed rather than modernized
  ([ADR-0006](0006-remove-benchmark-primitives-and-direct-mode.md)); metrics report that separately.
