# Spec F – Security: Spring Security, BCrypt, admin role, no PII in logs

| | |
|---|---|
| Status | Planned |
| Branch | `phase/f-security` |
| JDK | 25 |
| Depends on | E |
| ADRs | [0010](../adr/0010-legacy-security-findings.md) |
| Exit tag | `f-secure` |

## Goal

Fix every security finding that phase A confirmed, with the smallest visible change, and record each
behavior change in the catalog with the ADR that authorizes it.

## Scope

- Spring Security with form login replacing the application-level login check. Same `/app` login form fields
  (`uid`, `passwd`), CSRF tokens added to every form.
- Passwords: `DelegatingPasswordEncoder` with BCrypt. Existing plain-text passwords are re-hashed on the next
  successful login (upgrade-on-login), so seeded users and the load-test users keep working.
- `/config` requires an `ADMIN` role (CT-28, CT-29 change).
- No password in logs: remove the `login(userID, password)` trace; `toString()` and HTML dumps of
  `AccountProfileDataBean` no longer include password or credit card.
- Credit card shown masked on the account page (CT-09 changes).
- Ownership check on sell, if S-08 was confirmed in phase A (CT-25 changes).
- Security headers and session fixation protection (Spring Security defaults, verified by tests).

## Acceptance criteria

- [ ] Every finding of ADR-0010 is marked fixed, or explicitly accepted with a reason, in `MIGRATION.md`.
- [ ] The characterization suite passes with only the edits authorized by ADR-0010 (CT-09, CT-25 if S-08, CT-28, CT-29).
- [ ] A test proves that a seeded plain-text user can log in and is stored hashed afterwards.
- [ ] The load-test plan still runs (it logs in with seeded users); result stored as `docs/metrics/f-secure.json`,
      so the cost of BCrypt on the login path is visible.

## Risks

| Risk | Mitigation |
|---|---|
| BCrypt makes login much slower under load | Measure it (metrics row); tune the cost factor with the number written in the ADR, not guessed |
| CSRF breaks the JMeter plan | Already handled: since phase A the plan extracts a token from each form when one is present |
| Spring Security's default login (`POST /login`, `username`/`password`) breaks the `legacy-app.md` contract | Login processing matches `POST /app` with `action=login` (custom `RequestMatcher`), `usernameParameter("uid")`, `passwordParameter("passwd")`; CT-01 to CT-05 prove it |
