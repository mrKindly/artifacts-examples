# Artifact 4 — Ship It (Working Result + Release Readiness)

> Demo example: synthetic evidence modeled on the FS07-3521 story shape. PR links, run numbers and timings are illustrative reference values; chat logs are synthetic sessions in Cursor's chat-export format.

> Purpose: prove a working, review-ready and release-ready result that matches the spec for FS07-3521. A reviewer can find the PRs, the verification, the review, and the release decision from the links below.

The feature is `GetFulfilledSalesOrdersByMake` (SOAP) and `POST /v1/sales-orders/fulfilled-by-make` (router): one compact row per fulfilled equipment serial for a make and a fulfillment watermark.

---

## Working result (at a glance) and Inventory list

| Item | Location / link |
|------|-----------------|
| PR — on-prem service | [PR 2481, `FS.OnPrem.Soap`](https://dev.azure.com/ECI-FS/FS07/_git/FS.OnPrem.Soap/pullrequest/2481) (demo reference) |
| PR — cloud router | [PR 377, `FS.Cloud.Router`](https://dev.azure.com/ECI-FS/FS07/_git/FS.Cloud.Router/pullrequest/377) (demo reference) |
| Verification | [`./verification.md`](./verification.md) |
| Review notes | [`./review.md`](./review.md) |
| Release notes | [`./release.md`](./release.md) |
| Chat logs | [`./review-chat-log.md`](./review-chat-log.md), [`./release-chat-log.md`](./release-chat-log.md) |
| Final decision | **Merge** both PRs. Deploy on-prem first; route stays off in Production until the Staging consumer smoke passes. |

---

### Verification evidence

Local proof (AC-1 to AC-14) is in [`../artifact3-workflow/verification.md`](../artifact3-workflow/verification.md). The proof below is from environments other than the developer machine.

| AC / behaviour | Proof (test name / manual step) | Result |
|----------------|--------------------------------|--------|
| AC-1 to AC-12 (on-prem) | PR validation build 8841: NUnit 439/439; tSQLt 151/151 on the `FsCore_QA` run 5120 (second attempt, see below) | pass |
| AC-13, AC-14 (router) | PR validation build 1893: xUnit 223/223 and OpenAPI snapshot | pass |
| Behavior on a deployed stack | Manual QA on Dev, 2026-10-29: consumer sample call through the router, 94 rows, SQL count 94 | pass |
| Route switch and auth on a deployed stack | Manual QA on Dev: 401 without token, 403 wrong consumer, 404 switch off | pass |
| Consumer registration against Staging | Not yet run (needs the consumer owner, after merge) | gap, non-blocking |

- **Automated checks:** PR build 8841 (on-prem), PR build 1893 (router), tSQLt run 5120 on the shared QA database. Run 5120 failed once on two tests that belong to another branch's fixtures (shared-database collision) and passed 151/151 on re-run. Detail in [`./verification.md`](./verification.md).
- **Manual QA:** [`./verification.md`](./verification.md#manual-qa-on-dev), steps 1-6.

---

### Review notes

- **Summary:** both PRs match the accepted spec; AC-1 to AC-14 have proof on CI and on Dev. The change is additive and the route is behind a switch.
- **Blockers:** none.
- **Missing evidence / gaps:** consumer-side registration test on Staging; timing on production volume; the one flaky tSQLt run caused by the shared QA database.
- **Final decision:** **Merge.** Deploy on-prem to Production first, router second, enable the route only after the Staging consumer smoke.

Full notes: [`./review.md`](./review.md).

---

### Release notes

- **Release notes:** new read-only operation and route; new proc `usp_GetFulfilledSalesOrdersByMake` and function `fn_ContainsWholeWord`, created by migration `0152`. No table or existing contract changes.
- **Risk control:** first switch the route off (configuration change), then redeploy the previous router build, then the previous on-prem build. The new DB objects can stay.
- **Observability / flag decision:** route-level switch `Routes:FulfilledByMake:Enabled`, off in Production. Log operation name, row count and elapsed time. Alert on router 5xx for the route and on `TooManyRows` rate.
- **Known risks & limitations:** 5000-row cap, daily sync assumed, header status is not used, no paging.

Full notes: [`./release.md`](./release.md).
