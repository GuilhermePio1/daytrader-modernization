# Contracts

The exact external surface of the application, frozen in phase A from the running `v0-legacy` app. Later
phases must keep it, except where an ADR authorizes a change.

| Contract | Content | Captured in | Frozen until |
|---|---|---|---|
| `legacy-app.md` | Every `/app?action=…`: method, parameters, page rendered, fields the suite reads | A | `v2.0.0` |
| `marketsummary-websocket.md` | Endpoint `/daytrader/marketsummary`, the `{"action":"update"}` request, the pushed JSON (market summary and recent quote changes), one real example of each | A | `v2.0.0` |
| `config.md` | `/config` actions and parameters (reset, build DB, runtime modes) | A | F (ADMIN role) |
| `api-v1.openapi.yaml` | Public REST API, written contract-first | H | – |

No file in this folder is written from reading the code alone: each one includes values captured from the
running application.
