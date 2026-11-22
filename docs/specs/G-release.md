# Spec G – Release v2.0.0: container, CI hardening, final metrics, docs

| | |
|---|---|
| Status | Planned |
| Branch | `phase/g-release` |
| JDK | 25 |
| Depends on | F |
| Exit tag | **`v2.0.0`** |

## Goal

Close the committed scope: the application ships as a container image, CI guards it, and the README tells the
before/after story with measured numbers only.

## Scope

- Container image (Spring Boot buildpacks or a multi-stage Dockerfile on a JRE 25 base); Docker Compose with
  PostgreSQL and the app; Actuator liveness and readiness probes.
- CI: build, unit and integration tests, characterization suite against the container, Trivy image scan
  (fails on critical), Dependabot for Maven and GitHub Actions.
- Final `/app` load test, same parameters as the baseline; `docs/metrics/v2.0.0.json`; README metrics table
  complete. Any metric that got worse is shown and explained.
- README: architecture before and after (one diagram each), the security findings and their fixes, and a
  "what I would do next" section that points to H and I.

## Acceptance criteria

- [ ] `docker compose up` starts the whole system from a clean machine.
- [ ] No `TBD` left in the `v0-legacy` and `v2.0.0` metric columns.
- [ ] All specs A–G marked `Done`; `MIGRATION.md` complete.
- [ ] Release `v2.0.0` published on GitHub with release notes.
