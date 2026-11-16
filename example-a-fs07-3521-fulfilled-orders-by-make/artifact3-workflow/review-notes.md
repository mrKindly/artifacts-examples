# Review notes — FS07-3521

Human review of the spec and the plan. The spec questions were answered through the PM walkthrough ([`spec-chat-log.md`](./spec-chat-log.md)). Plan changes come from the review in [`plan-chat-log.md`](./plan-chat-log.md).

## Outcome

[`spec.md`](./spec.md) (v3) is the accepted spec: `spec-reviewer` returned PASS on iteration 2. [`plan.md`](./plan.md) is the accepted plan after five changes. Local verification of AC-1 to AC-14 passed ([`verification.md`](./verification.md)).

## Decisions

| Decision | Accepted / Changed / Rejected | Why |
|----------|-------------------------------|-----|
| "Fulfilled" = line has at least one fulfillment record; header status ignored | Accepted | PM: register what shipped. Header status is unreliable in `FsCore` (can say `Fulfilled` with no fulfillment records). |
| "Fulfilled" = header status `Fulfilled` only | Rejected | That was the prototype's rule. It misses partially fulfilled orders and includes orders that never shipped. |
| Watermark = latest fulfillment time of the line, strictly greater, UTC | Accepted | The consumer registers on shipment and keeps its own clock. Strict `>` avoids re-sending the last row. |
| Watermark = order create date with a lookback window | Rejected | Prototype rule. It misses old orders fulfilled late. |
| One row per equipment serial, latest fulfillment wins | Accepted | The consumer registers devices, not orders. The first draft said both "twice" and "once"; resolved to once. |
| Ship-to: fulfillment party first, order party as fallback; bill-to from the order | Accepted | PM rule. 38% of rows carry a fulfillment ship-to, so it changes real results. |
| Whole word = split at non-alphanumerics; multi-word value matches adjacent words | Accepted | Needed for `SHARP-MX` and `Sharp Electronics`. |
| Hard cap of 5000 rows with `TooManyRows`, no partial result | Changed | Jira had no limit. Plan Mode proposed paging; the PM wants no paging, only a clear stop. |
| `PageSize` / `PageToken` in the request | Rejected | Scope creep. Not in the story or the spec. |
| Refactor the party lookup shared with `GetSalesOrdersByDate` | Rejected | Touches an operation other consumers use. Contracts and procs of existing operations stay unchanged. |
| Whole-word match as `LIKE '%x%'` plus a C# filter | Rejected | Substring rows reach memory, and the cap would count rows that should not match. Replaced by a tested SQL function. |
| Single large "implement and test" step | Changed | Replaced by test-first steps with gates G1, G2, G3. |
| One PR for service and router | Changed | Two repos, two PRs. On-prem deploys first. |
| Route switch off by default in Production | Accepted | Lets the service deploy first and the consumer smoke pass before exposure. |
| Add `spec-reviewer` iteration 2 non-blocking finding (name `TooManyRows` in AC-11) | Accepted | Traceability from AC to fault code. |
| Spec assumption that ~40% of rows have a fulfillment ship-to | Changed | Unmeasured. Replaced by the measured 38% (8,134 of 21,406). |
