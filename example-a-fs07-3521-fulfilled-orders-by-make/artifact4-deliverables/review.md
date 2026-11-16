# Review notes — FS07-3521 ship package

> Illustrative reference values. Reviewed 2026-10-30 against PR 2481 (`FS.OnPrem.Soap`) and PR 377 (`FS.Cloud.Router`), the accepted spec, and [`./verification.md`](./verification.md). An AI review pass ([`./review-chat-log.md`](./review-chat-log.md)) prepared the findings; the human reviewer decided each one.

## Summary

Both PRs match the accepted spec. The on-prem PR adds one operation, one proc, one function and tests; the router PR adds one route, one policy, one switch and tests. All 14 acceptance criteria have proof on CI or on Dev. The change is additive: the WSDL diff contains only the new operation and no existing contract, proc or table changed. The route is off by default in Production.

## Findings

| # | Class | Finding | Resolution |
|---|-------|---------|------------|
| 1 | Info | tSQLt run 5120 failed once on two tests this PR does not touch (shared `FsCore_QA`, fixtures without the `tsqlt_` prefix from another branch). Passed on re-run. | Recorded. Not caused by this PR. Tracked as a use case in the Artifact 5 playbook. |
| 2 | Non-blocking | The service log line writes the `Make` value. It is a product name, not personal data, but the rule says to log operation, row count and elapsed time only. | Accepted: the make value helps support. Added to Known risks. Rule text left as is. |
| 3 | Non-blocking | `TooManyRows` is mapped to HTTP 422; some consumers expect 400 for all validation faults. | Kept 422, documented in Swagger (AC-14). |
| 4 | Non-blocking | No timing on production volume. | Release step: read elapsed time in the first week; trigger in [`./release.md`](./release.md). |
| 5 | Info | `Web.config` and existing procs untouched (verified with `git diff --stat`). | None. |

No blocking findings.

## Blockers

None.

## Missing evidence / gaps

- Consumer registration test on Staging. It needs the consumer owner and a Staging deployment, which happens after merge. It gates enabling the route in Production, not the merge.
- Production-volume timing.

## Final decision

**Merge** PR 2481 and PR 377.

Order after merge: deploy on-prem to Dev (done), Staging, Production; then the router. Enable `Routes:FulfilledByMake:Enabled` in Production only after the consumer owner confirms the Staging smoke. Release and rollback steps: [`./release.md`](./release.md).
