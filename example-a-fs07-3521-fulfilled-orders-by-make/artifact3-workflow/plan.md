# Implementation Plan

Source: `docs/FS07-3521-fulfilled-orders-by-make/plan.md` in `FS.OnPrem.Soap`. Written 2026-10-20 in Plan Mode from [`spec.md`](./spec.md) (v3) and [`context-map.md`](./context-map.md), then edited by a human ([`review-notes.md`](./review-notes.md)).

### Confirmed Facts

- The spec has 14 acceptance criteria: AC-1 to AC-12 for the on-prem operation, AC-13 and AC-14 for the router.
- `GetSalesOrdersByDate` is the pattern: operation class in `src/FS.Soap.Services/Sales/`, repository method in `SalesRepository`, proc in `db/procs/`. Contracts are additive.
- Columns named in the spec were checked against `INFORMATION_SCHEMA` in `FsCore_Dev` (see the log of the scaffold skill).
- 38% of fulfillment rows have a ship-to party (8,134 of 21,406), so the party fallback is not an edge case.
- SQL Server 2019 has no regex. `db/functions/` has helpers; none does whole-word matching.
- The WSDL snapshot test fails on any contract diff until the snapshot is regenerated.
- The router registers routes per consumer with a policy; Swagger comes from route metadata and has a snapshot test.

### Assumptions

- A word separator is any character that is not `[A-Za-z0-9]`; normalization happens in a scalar function so the proc stays readable.
- Fulfillment time per line is `MAX(FulfilledOn)`; one-row-per-serial is a second `MAX` across lines.
- A temp table or CTE is acceptable inside the proc. Index changes are not needed at the dev volume; the performance check in Phase 4 decides.
- The route switch is a router configuration value, read at request time.

### Files / Modules Involved

**`FS.OnPrem.Soap`**

- `src/FS.Soap.Contracts/Sales/GetFulfilledSalesOrdersByMake{Request,Response}.cs` (new), `FulfilledEquipmentRow.cs`, `AddressInfo.cs` (new)
- `src/FS.Soap.Services/Sales/GetFulfilledSalesOrdersByMake.cs` (new)
- `src/FS.Soap.Data/SalesRepository.cs` (method added)
- `db/procs/usp_GetFulfilledSalesOrdersByMake.sql`, `db/functions/fn_ContainsWholeWord.sql` (new)
- `tests/FS.Soap.Tests/Sales/GetFulfilledSalesOrdersByMakeTests.cs`, `db/tests/usp_GetFulfilledSalesOrdersByMake/`, `db/tests/fn_ContainsWholeWord/` (new)
- `tests/FS.Soap.Tests/Snapshots/service.wsdl` (regenerated)

**`FS.Cloud.Router`**

- `src/Routes/SalesOrders/FulfilledByMakeRoute.cs`, `src/Clients/SalesSoapClient.cs` (method), `src/Policies/ConsumerPolicies.cs` (policy), `appsettings*.json` (switch), OpenAPI snapshot, `tests/Router.Tests/SalesOrders/`

### Implementation Plan

**Phase 1 — Spec and PM lock.** Done: spec v3, `spec-reviewer` PASS.

**Phase 2 — On-prem contract and data path**

1. *Scaffold (skill `add-soap-operation`).* Result: DTOs, operation class, proc stub, 19 red tests, WSDL diff. **Gate G1: human reviews the red tests and the WSDL diff.**
2. *Test, then implement rules for fulfilled and watermark (AC-1, AC-6).* tSQLt fixtures: header `Fulfilled` with no fulfillment; header `Open` with one fulfilled line; equal-to-watermark; two fulfillments on a line. Then the proc joins line → fulfillment → equipment and filters on `MAX(FulfilledOn) > @FulfilledSince`.
3. *Test, then implement whole-word matching (AC-2 to AC-5).* tSQLt tests for `fn_ContainsWholeWord` using the five examples in the spec, hyphen separators, wildcard characters, and a multi-word value. Then the function; then the proc applies it to `Make` OR `MakeDescription`.
4. *Test, then implement row grain, parties and cap (AC-7, AC-8, AC-9, AC-11).* Tests: same serial twice, fulfillment-level ship-to wins, missing parties, 5001 rows. Then reduce to one row per serial, add party selection, and the cap. **Gate G2: human runs the proc on `FsCore_Dev` for `Sharp` and compares five rows by hand with the source tables.**
5. *Test, then implement the operation class (AC-10, AC-12).* NUnit tests for validation faults, empty result and row mapping, then the class.
6. *Regenerate the WSDL snapshot and check size and time.* Diff must contain only the new operation. Time the 5000-row case on the dev copy (target under 3 seconds).

**Phase 3 — Cloud router and docs**

1. *Tests, then route (AC-13).* xUnit tests: unauthorized consumer, wrong consumer, fault mapping (400 with code, 422 for `TooManyRows`), switch off returns 404. Then the route and policy.
2. *Tests, then Swagger (AC-14).* OpenAPI snapshot includes request, response, fault codes and the `Sharp` example.
3. *Route switch.* Default off in Production configuration. **Gate G3: human confirms deploy order and the Staging smoke owner.**

**Phase 4 — Verification package.** Fill the AC → proof table, run both full suites, run the consumer sample call through the router on Dev, record known limitations.

**Phase 5 — PRs and release notes.** One PR per repo, each referencing FS07-3521. Release notes and rollback in [`artifact4`](../artifact4-deliverables/release.md).

### Risks

- Whole-word logic in SQL is easy to get subtly wrong at word boundaries. Mitigation: a dedicated function with its own test class.
- The shared `FsCore_QA` database can make DB tests flaky. Mitigation: fixtures live inside the test with the `tsqlt_` prefix.
- Router and service deploy separately. If the router goes first, the route returns a SOAP endpoint error. Mitigation: switch off by default; deploy on-prem first.
- Dev data may not contain every edge case. Mitigation: fixtures create their own rows.

### Out Of Scope

- Pagination, cursors, and any server-side memory of the caller's sync time.
- FS07-3530 (REST wrapper), and any refactor of `GetSalesOrdersByDate` or the shared party code.
- Index tuning, schema changes, `Web.config` limits.
- Data fixes for header `Status`.

### Verification

| ID | Acceptance Criterion | Verifiable By |
|----|----------------------|---------------|
| AC-1 | Only lines with a fulfillment record count | Automated Test (tSQLt) |
| AC-2 | Whole-word match on Make | Automated Test (tSQLt) |
| AC-3 | Whole-word match on MakeDescription | Automated Test (tSQLt) |
| AC-4 | Substrings excluded | Automated Test (tSQLt) |
| AC-5 | Case-insensitive, separators | Automated Test (tSQLt) |
| AC-6 | Strictly after watermark, UTC | Automated Test (tSQLt + NUnit) |
| AC-7 | Ship-to and bill-to, fulfillment-first | Automated Test (tSQLt + NUnit), Manual Test on dev data |
| AC-8 | Equipment identity fields | Automated Test (NUnit) |
| AC-9 | One row per serial | Automated Test (tSQLt) |
| AC-10 | Empty result, no fault | Automated Test (NUnit) |
| AC-11 | `TooManyRows` over 5000 | Automated Test (tSQLt + NUnit) |
| AC-12 | Validation faults, additive WSDL | Automated Test (NUnit, WSDL snapshot) |
| AC-13 | Authorized consumer only, fault mapping, switch | Automated Test (xUnit), API Check on Dev |
| AC-14 | Swagger content | Automated Test (OpenAPI snapshot), UI Check |
