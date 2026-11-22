# Spec 00 – Repository setup

| | |
|---|---|
| Status | Done |
| Branch | `docs/modernization-plan`, then `phase/0-setup` |
| JDK | n/a |
| Depends on | – |
| ADRs | [0001](../adr/0001-record-architecture-decisions.md), [0002](../adr/0002-choose-daytrader-as-legacy-base.md) |
| Exit tag | `v0-legacy` (upstream commit `643d04c`) |

## Goal

Publish a repository that starts from the **unmodified** DayTrader 7 history, so every later change is visible
as a diff against real legacy code. Put the planning artifacts (specs, ADRs, catalog) in place.

## Scope

- Keep the full history of `WASdev/sample.daytrader7` (121 commits, 2015–2025) and tag its last commit
  `v0-legacy`. Upstream tags `v1.0`–`v1.4` are kept as they are, which is why this project's release is `v2.0.0`.
- Add README.md, CLAUDE.md, specs, ADRs, the characterization catalog, the metrics and contracts folders.
  Move the upstream README, load-test README and the upstream `docs/*.md` pages (Liberty and WDT setup) to
  `docs/legacy/`; their internal links are left as upstream wrote them.
- Create a **standalone** GitHub repository (not a fork) and push the default branch and the tags.
- Repository settings: description, topics (`legacy-modernization`, `java-ee`, `spring-boot`, `ejb`, `jms`,
  `migration`, `openrewrite`), branch protection on the default branch (PR required; CI required once phase A
  adds it).
- Clone location: a **short path** on Windows (for example `C:\dev\daytrader-modernization`). Source paths
  reach 124 characters, and Maven and `javac` fail beyond the 260-character Windows path limit.

## Out of scope

- Any change to source code, `pom.xml` or `server.xml`. That starts in phase A.
- Scripts. They are written in phase A, where they can be run and tested.

## Tasks

1. Clone with full history, `git tag v0-legacy` on `643d04c`.
2. Commit the planning files on `docs/modernization-plan`.
3. `git remote rename origin upstream` (the clone's `origin` points to WASdev).
4. **Ask the owner for confirmation**, then
   `gh repo create GuilhermePio1/daytrader-modernization --public --source . --remote origin --push`.
5. `git push origin --tags`.
6. Merge `docs/modernization-plan` through a PR (the first PR in the history).
7. Configure description, topics and branch protection (`gh repo edit`, `gh api`).

## Acceptance criteria

- [x] `git log v0-legacy` on the new repository matches the upstream history exactly.
- [x] README shows the status table with the v2.0.0 line and credits IBM and the Apache Geronimo project.
- [x] `LICENSE` and the upstream `NOTICE` files are untouched.
- [x] Every relative link in README, specs and ADRs resolves.
- [x] The default branch is protected.

## Risks

| Risk | Mitigation |
|---|---|
| A fork would hide contributions and point reviewers to an unrelated upstream | Standalone repo, credit in the README, `upstream` remote kept locally |
| Confusion between upstream tags (`v1.x`) and this project's tags | This project's tags are phase names (`a-safety-net` …) and the release starts at `v2.0.0` |
| Apache-2.0 obligations | Keep `LICENSE` and `NOTICE`, mark changed files in their header |
| Windows path length | Clone to a short path. Optionally `git config core.longpaths true` |

## Commands

```powershell
git clone https://github.com/WASdev/sample.daytrader7.git C:\dev\daytrader-modernization
git tag v0-legacy 643d04c
git remote rename origin upstream
gh repo create GuilhermePio1/daytrader-modernization --public --source . --remote origin --push
git push origin --tags
```
