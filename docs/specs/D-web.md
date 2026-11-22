# Spec D – Web layer: Thymeleaf, Spring WebSocket, executable jar

| | |
|---|---|
| Status | Planned |
| Branch | `phase/d-web` |
| JDK | 25 |
| Depends on | C |
| ADRs | [0007](../adr/0007-keep-jsp-flow-drop-jsf.md) |
| Contracts | `legacy-app.md`, `marketsummary-websocket.md` (frozen) |
| Exit tag | `d-web` |

## Goal

Replace the servlet-plus-JSP front end with Spring MVC and Thymeleaf, and the JSR-356 endpoint with Spring
WebSocket, keeping every URL, form field and WebSocket message exactly as the contracts froze them. Then
package as an executable `jar`.

## Scope

- `TradeAppServlet` + `TradeServletAction` (one servlet dispatching on `action=`) → one Spring MVC controller
  that keeps `/app?action=…` working (dispatch by `params = "action=buy"`), so links, forms and the load-test
  plan stay valid.
- Each JSP → a Thymeleaf template, one page per commit, suite green after each. Shared header, footer and menu
  become fragments. `OrdersAlertFilter` → a `HandlerInterceptor` or model attribute.
- `/marketsummary` → Spring WebSocket handler with the same path, request and response JSON.
- `jar` packaging; JSP and `tomcat-embed-jasper` removed.

## Acceptance criteria

- [ ] The characterization suite passes unchanged (its locators never relied on JSP markup).
- [ ] The `/app` JMeter plan runs unchanged.
- [ ] `java -jar daytrader-app.jar` starts the application; no `.jsp` file remains.

## Risks

| Risk | Mitigation |
|---|---|
| JSP scriptlets hold logic (formatting, totals) | Move it to the controller or a view helper with unit tests before converting the page |
| WebSocket push threading changes with Spring | Keep the session registry thread-safe; CT-16/CT-17 cover the observable behavior |
