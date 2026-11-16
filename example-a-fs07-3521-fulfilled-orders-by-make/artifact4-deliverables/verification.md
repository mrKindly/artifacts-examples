# Verification — FS07-3521 (ship package)

> Illustrative reference values. Two layers: local proof (already written up) and proof from environments other than the developer machine.

## Local proof

[`../artifact3-workflow/verification.md`](../artifact3-workflow/verification.md): NUnit 439/439, tSQLt 151/151, xUnit 223/223, AC-1 to AC-14 pass, 2026-10-27.

## Automated checks outside the developer machine

| Check | Where | Result |
|-------|-------|--------|
| PR validation build 8841 (`FS.OnPrem.Soap`, PR 2481): build, NUnit, WSDL snapshot | Azure DevOps hosted agent | pass: 439/439, 0 warnings |
| tSQLt run 5120 on `FsCore_QA`, attempt 1 | Integration QA (shared database) | fail: 2 tests (`usp_GetSalesOrdersByDate` fixtures from another branch's run) |
| tSQLt run 5120, attempt 2 | Integration QA | pass: 151/151, including the 18 new tests |
| PR validation build 1893 (`FS.Cloud.Router`, PR 377): build, xUnit, OpenAPI snapshot | Azure DevOps hosted agent | pass: 223/223 |

The attempt-1 failure was in tests this PR does not touch. The two failing tests wrote fixtures without the `tsqlt_` prefix on the shared database while another run was active. The new tests use the prefix and passed in both attempts.

## Manual QA on Dev

Environment: Dev stack, on-prem build from PR 2481 and router build from PR 377 deployed on 2026-10-29 by the Dev deploy pipeline. Route switch on in Dev.

| Step | Action | Expected | Result |
|------|--------|----------|--------|
| 1 | `POST /v1/sales-orders/fulfilled-by-make` with `make=Sharp`, `fulfilledSince=2026-09-01T00:00:00Z`, consumer token | 200, rows for `Sharp`, `Sharp Electronics`, `SHARP-MX`; none for `Sharpie` | 94 rows, no `Sharpie` |
| 2 | SQL `COUNT(DISTINCT SerialNumber)` over the same filter on the Dev database | Equals step 1 | 94 = 94 |
| 3 | Same call with `fulfilledSince` set to the largest `fulfilledOn` from step 1 | 200, empty list | 0 rows, 200 |
| 4 | `make=Sharp%` | 400, code `InvalidMake` | 400 `InvalidMake` |
| 5 | No token / token of a different consumer | 401 / 403 | 401 / 403 |
| 6 | Switch off in Dev configuration | 404 | 404 |

## AC → proof (off-machine)

| AC | Proof | Result |
|----|-------|--------|
| AC-1 to AC-9 | tSQLt run 5120 attempt 2 (18 new tests); Manual step 1-2 | pass |
| AC-10 | NUnit in build 8841; manual step 3 | pass |
| AC-11 | tSQLt `test more than 5000 rows raises TooManyRows` in run 5120 | pass |
| AC-12 | NUnit in build 8841; WSDL snapshot test in the same build | pass |
| AC-13 | xUnit in build 1893; manual steps 4-6 | pass |
| AC-14 | OpenAPI snapshot in build 1893 | pass |

## Gaps

- Consumer registration code not exercised: planned as the Staging smoke after merge.
- Timing 1.4 s for 5000 rows is from the dev copy, not from production volume.
