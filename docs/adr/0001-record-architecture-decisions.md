# ADR-0001: Record architecture decisions

- **Status:** Accepted
- **Date:** 2026-11-03
- **Phase:** 0
- **Supersedes / Superseded by:** none

## Context

Re-platforming a Java EE application makes dozens of decisions: what replaces each container service, which
legacy features are dropped, which behaviors change on purpose. Reviewers need the reasoning, not only the diff.
A portfolio project also has to show how the work was done.

## Decision

We will record every significant decision as an Architecture Decision Record in `docs/adr/`, in Michael
Nygard's format (Context, Decision, Consequences) plus an "Alternatives considered" table
([template](0000-template.md)).

- Files are named `NNNN-kebab-title.md` and listed in the [index](README.md).
- Status lifecycle: `Proposed` → `Accepted` | `Rejected`. An accepted ADR is never edited except to mark it
  `Superseded by ADR-XXXX`.
- A decision is significant if it is hard to reverse, changes observable behavior, adds or removes a
  dependency, or would surprise a reviewer.
- PRs that implement a decision link to the ADR. Every catalog scenario that changes cites one.

## Alternatives considered

| Option | Why not |
|---|---|
| Decisions in PR descriptions only | Hard to find later, not versioned with the code |
| A wiki | Lives outside the repository and drifts |

## Consequences

Each phase costs a little more time to write down; in exchange the repository explains itself. By `v2.0.0` no
ADR for phases 0–G may remain `Proposed`.
