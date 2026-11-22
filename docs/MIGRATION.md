# Migration journal

What actually happened in each phase: every change made to get something working, every surprise, what
OpenRewrite automated and what was done by hand. Specs say what was planned; this file says what was done.
Entries are added as each phase runs, newest phase last.

## Phase 0 – Repository setup

- Cloned `WASdev/sample.daytrader7` with full history (121 commits, 2015-10-13 to 2025-10-16). Tagged the last
  upstream commit `643d04c` as `v0-legacy`. Upstream tags `v1.0`–`v1.4` kept.
- Moved `README.md`, `README_LOAD_TEST.md` and the upstream `docs/*.md` pages to `docs/legacy/`.
- Planning made by reading the code; **the legacy build has not been run yet**.
- The working copy had lost its `.git` directory and 117 legacy files (the whole `daytrader-ee7` module and
  seven EJB classes, among them `TradeSLSBBean` and `TradeDirect`). The history was taken again from upstream
  and the files restored unchanged from `643d04c`; no source file differs from upstream.
- Added `.idea/` to `.gitignore` so IDE settings stay out of the repository.
- Published as a standalone public repository (not a fork) with `master` as the default branch, the upstream
  remote kept as `upstream`, and tags `v0-legacy` and `v1.0`–`v1.4` pushed. The plan was merged through PR #1
  with a merge commit, so its three commits stay separate.
- `master` protection: pull request required with **0 approvals** (single maintainer, so a required approval
  would block every merge), enforced for administrators, force pushes and deletion blocked. No required status
  check yet; phase A adds CI and makes it required.

## Phase A – Baseline

### Build changes

| Change | File | Reason |
|---|---|---|
| _none yet_ | | |

### Security and correctness findings (ADR-0010)

| ID | Confirmed / refuted | Notes (prose only, no exploit steps) |
|---|---|---|
| S-01 | TBD | |
| S-02 | TBD | |
| S-03 | TBD | |
| S-04 | TBD | |
| S-05 | TBD | |
| S-06 | TBD | |
| S-07 | TBD | |
| S-08 | TBD | |
| R-01 | TBD | |
| R-02 | TBD | |
| R-03 | TBD | |

### Two-phase commit check (ADR-0008)

_Does Liberty run a real XA transaction between embedded Derby and the JMS engine in `Async_2-Phase` mode?
Evidence from the transaction trace goes here._

## Phase B – Trim

## Phase C – Spring Boot core

### OpenRewrite

| Recipe | Files changed | Manual follow-up |
|---|---|---|

## Phase D – Web layer

## Phase E – Data

## Phase F – Security

## Phase G – Release
