# Verification — FS07-3521 (local proof)

> Condensed, illustrative results collected on the developer machine on 2026-10-27. The off-machine run (CI, Dev environment) is in [`../artifact4-deliverables/verification.md`](../artifact4-deliverables/verification.md).

## Commands

```
msbuild FS.Soap.sln /p:Configuration=Debug
vstest.console tests\FS.Soap.Tests\bin\Debug\FS.Soap.Tests.dll
powershell db\tests\run-tsqlt.ps1 -Database FsCore_Dev
dotnet test FS.Cloud.Router.sln
```

## Results

| Suite | Before | After | New tests |
|-------|--------|-------|-----------|
| NUnit `FS.Soap.Tests` | 412 passed | 439 passed, 0 failed | 27 |
| tSQLt `FsCore_Dev` | 133 passed | 151 passed, 0 failed | 18 (11 on the proc, 7 on `fn_ContainsWholeWord`) |
| xUnit `Router.Tests` | 209 passed | 223 passed, 0 failed | 14 |
| WSDL snapshot test | pass | pass (snapshot regenerated, diff = +1 operation, +4 types) | 0 |

Build warnings: 0 in both repos. 5000-row timing on the `FsCore_Dev` copy: 1.4 s (target under 3 s).

## AC → proof

| AC | Proof (test name / manual step) | Result |
|----|--------------------------------|--------|
| AC-1 | tSQLt `usp_GetFulfilledSalesOrdersByMake`: `test header Fulfilled without fulfillment returns nothing`, `test partially fulfilled order returns only fulfilled lines` | pass |
| AC-2 | tSQLt `test make match returns the row` | pass |
| AC-3 | tSQLt `test make description match returns the row` | pass |
| AC-4 | tSQLt `test superstring make is excluded` and `fn_ContainsWholeWord`: `test superstring does not match`. Manual: `Sharp` on dev returns 3 makes (`Sharp`, `Sharp Electronics`, `SHARP-MX`), not `Sharpie` | pass |
| AC-5 | tSQLt `fn_ContainsWholeWord`: `test case insensitive`, `test hyphen is a separator`, `test slash and ampersand are separators`, `test multi word phrase matches` | pass |
| AC-6 | tSQLt `test fulfillment equal to watermark is excluded`; NUnit `Request_ShouldPassWatermarkAsUtc_WhenKindIsUnspecified` | pass |
| AC-7 | tSQLt `test fulfillment ship to wins over order ship to`, `test missing parties return empty fields`; NUnit `Map_ShouldCopyBothAddresses`. Manual: 5 dev rows compared by hand against `SalesOrderParty`/`Party` | pass |
| AC-8 | NUnit `Map_ShouldCopyEquipmentIdentityAndOrderNumber` (all 8 fields asserted) | pass |
| AC-9 | tSQLt `test latest fulfillment is used`, `test same serial on two orders returns one row` | pass |
| AC-10 | NUnit `Get_ShouldReturnEmptyRows_WhenNothingMatches` (no fault) | pass |
| AC-11 | tSQLt `test more than 5000 rows raises TooManyRows`; NUnit `Get_ShouldThrowTooManyRows_AndReturnNoRows` | pass |
| AC-12 | NUnit `Validate_ShouldThrowInvalidMake_WhenMakeIsBlank`, `...WhenMakeIsLongerThan50`, `...WhenMakeHasWildcardCharacters` (3 cases), `Validate_ShouldThrowInvalidWatermark_WhenDefault`; WSDL snapshot test; `git diff --stat` shows no change to existing contracts, procs or migrations | pass |
| AC-13 | xUnit `Route_ShouldReturn401_WhenUnauthenticated`, `...403_WhenConsumerIsNotRegistered`, `...404_WhenSwitchIsOff`, `...400WithCode_WhenInvalidMake`, `...422_WhenTooManyRows`. Manual: local router against local host, calls with and without token | pass |
| AC-14 | xUnit `OpenApi_ShouldDocumentRequestResponseAndFaults`, `OpenApi_ShouldContainTheSharpExample`. Manual: Swagger UI shows the example | pass |

## Manual steps (local)

1. SoapUI against `http://localhost:8091/Sales.svc`: `Make = Sharp`, `FulfilledSince = 2026-09-01T00:00:00Z` → 87 rows, none from `Sharpie`.
2. Compared rows 1, 20, 40, 60, 87 field by field with the source tables: 5 of 5 equal.
3. Same call with `Make = Sharp%` → fault `InvalidMake`.
4. Local router → `POST /v1/sales-orders/fulfilled-by-make` with a valid local token: 200 and 87 rows. Without token: 401. With the switch off: 404.

## Gaps

- No consumer-side test (the consumer's own registration code). Planned for the Dev smoke in Artifact 4.
- Timing was measured on the dev copy, not on production volume.
