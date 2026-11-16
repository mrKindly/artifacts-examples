# Artifact 3 — Story → Spec → Plan → PR (Full Traceability)

> Demo example: synthetic evidence modeled on the FS07-3521 story shape (new integration API with ambiguous domain rules). Chat logs are synthetic sessions in Cursor's chat-export format; numbers are internally consistent across files.

> Purpose: prove a complete, gap-free chain from an original story to a PR — spec, plan, context map, human decisions, implementation, and verification.

---

## Traceability chain (at a glance) and Inventory list

| Stage | Artefact | Location |
|-------|----------|----------|
| Story | FS07-3521 as read from Jira | [`./story.md`](./story.md) |
| Spec | AI-ready spec (v3, reviewer PASS) | [`./spec.md`](./spec.md) |
| Spec chat logs | exported session + PM walkthrough | [`./spec-chat-log.md`](./spec-chat-log.md) |
| Plan | Implementation plan, 5 phases | [`./plan.md`](./plan.md) |
| Plan chat logs | exported session + review | [`./plan-chat-log.md`](./plan-chat-log.md) |
| Context map | hot / warm / cold / ignore | [`./context-map.md`](./context-map.md) |
| Verification | AC ↔ evidence table | [`./verification.md`](./verification.md) |
| Plan and spec review notes | human review + decisions | [`./review-notes.md`](./review-notes.md) |
| Tools used from Artifact 1 and 2 | `add-soap-operation` skill, rules, `spec-reviewer`, Atlassian MCP | [`../artifact1-tools/artifact1-main.md`](../artifact1-tools/artifact1-main.md), [`../artifact2-mcp-subagents/artifact2-main.md`](../artifact2-mcp-subagents/artifact2-main.md) |

How the chain connects: the Atlassian MCP read the story (including the comment that fixed who owns the sync clock). `/spec` plus the PM walkthrough produced the spec, and `spec-reviewer` forced four fixes before planning. The context map kept the deprecated equipment column out of the session. Plan Mode produced a plan that the human cut (paging, shared refactor) before the `add-soap-operation` skill scaffolded the first red tests.

---

### Verification

Local proof on 2026-10-27: NUnit 439/439 (27 new), tSQLt 151/151 (18 new), router xUnit 223/223 (14 new), WSDL diff = new operation only, 0 warnings. Detail: [`./verification.md`](./verification.md).

| AC | Proof (test name / manual step) | Result |
|----|--------------------------------|--------|
| AC-1 | tSQLt `test header Fulfilled without fulfillment returns nothing`, `test partially fulfilled order returns only fulfilled lines` | pass |
| AC-2, AC-3 | tSQLt `test make match returns the row`, `test make description match returns the row` | pass |
| AC-4, AC-5 | tSQLt `test superstring make is excluded`; `fn_ContainsWholeWord` case, hyphen and phrase tests; manual `Sharp` check (3 makes, no `Sharpie`) | pass |
| AC-6 | tSQLt `test fulfillment equal to watermark is excluded`; NUnit `Request_ShouldPassWatermarkAsUtc_WhenKindIsUnspecified` | pass |
| AC-7, AC-8 | tSQLt party tests; NUnit mapping tests; 5 dev rows compared by hand | pass |
| AC-9 | tSQLt `test latest fulfillment is used`, `test same serial on two orders returns one row` | pass |
| AC-10, AC-11 | NUnit `Get_ShouldReturnEmptyRows_WhenNothingMatches`; tSQLt `test more than 5000 rows raises TooManyRows` | pass |
| AC-12 | NUnit validation tests; WSDL snapshot test; no diff on existing contracts or procs | pass |
| AC-13, AC-14 | xUnit route and OpenAPI tests; local router calls (200 / 401 / 404) | pass |

---

### Human decisions & review notes

Full table: [`./review-notes.md`](./review-notes.md). The decisions that changed the outcome:

| Decision | Accepted / Changed / Rejected | Why |
|----------|-------------------------------|-----|
| Fulfilled = any fulfillment record on the line | Accepted | Header status is unreliable; partial fulfillment must register. |
| Watermark = fulfillment time, strict `>`, UTC | Accepted | Consumer owns the clock and registers on shipment. |
| Pagination proposed by Plan Mode | Rejected | PM wants no paging; scope creep. |
| Hard cap 5000 rows with `TooManyRows` | Changed | Jira had no limit; protects the service without paging. |
| Shared party-lookup refactor | Rejected | Touches an operation other consumers use. |
| `LIKE` plus C# filter for whole-word match | Rejected | Substring rows reach memory and distort the cap. |
| Unmeasured "40% have ship-to" | Changed | Replaced by measured 38% (8,134 / 21,406). |
