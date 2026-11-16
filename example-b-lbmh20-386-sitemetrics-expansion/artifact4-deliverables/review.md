# Review notes — LBMH20-386 (PR 9220)

> Condensed illustrative reconstruction (synthetic ids and numbers). Review snapshot 2026-07-15 of head `a41c7e9`. Inputs read in this order: spec, plan, verification, release notes, then the diff (last). See [./review-chat-log.md](./review-chat-log.md).

PR: [9220](https://dev.azure.com/ECI-LBMH/LBMH-Spruce/_git/LBMH.Services.Portal/pullrequest/9220) (demo reference). Spec: [../artifact3-workflow/spec.md](../artifact3-workflow/spec.md). Evidence: [./verification.md](./verification.md). Release: [./release.md](./release.md).

## Summary

The diff delivers the spec and nothing else: 17 files, six category cards (38 fields) added to the Metrics tab, the 9 internal columns hidden, the 4 existing cards unchanged, no schema, CDK or pipeline file touched. All 10 ACs have local proof (36 new test cases, manual steps M1-M6), changed-code coverage is 94.1% (.NET) and 95.2% (Web) with zero new warnings, and `diff-vs-spec-reviewer` returned PASS. The PR build (run 29561) is green.

It is not yet merge-ready because two pieces of off-machine evidence are missing, and one of them (QualityCheck 445 on the branch) is the only automated reading of the team's coverage gate. There is no product defect.

## Blockers

- **B1. QualityCheck 445 has not been run on the branch.** The Coverlet line gate (90%) is the only automated coverage check and it has no reading for this branch. The baseline is 87.72% line on build 29300 and the gate fails regardless, so the question for the run is not "green" but "did whole-solution line coverage stay at or above 87.72%, and is the only failure still the Coverlet gate". Needed: queue 445 on `feature/LBMH20-386-sitemetrics-expansion` and record the result in the PR.
- **B2. Changed-code coverage evidence is not attached to the PR.** The 94.1% figure (192 of 204 changed executable lines) lives in a local `pre-pr-review.md`. The AC-10 bar is on changed code, and a reviewer cannot verify it from the PR. Needed: attach the changed-code coverage report (or the `pre-pr-review.md`) to the PR.

## Gaps (missing evidence)

- No Dev smoke before merge (by design; becomes release step 1, see [./release.md](./release.md)).
- No test covers the SPA receiving a response **without** the new groups field (App deployed before Api); only empty groups are tested.

## Findings by class

| # | Class | Finding | Where |
|---|-------|---------|-------|
| B1 | blocker | QualityCheck 445 not run on the branch | ADO pipeline 445 |
| B2 | blocker | Changed-code coverage not attached to the PR | PR description / attachments |
| N1 | non-blocking | A `SiteMetrics` column added later is not shown until it has a category row (documented limitation, matches spec) | `SiteMetricsMapper.cs`, spec Edge Cases |
| N2 | non-blocking | Fixture parity test asserts counts (6 groups, 38 items), not labels | `siteMetrics.fixture.test.ts` |
| N3 | non-blocking | `npm run test:ci` still exits 1 (same 2 pre-existing failing files as on `main`); not caused by this PR, not fixed here | Web CI |
| N4 | non-blocking | No test for a response lacking the groups field during the App-before-Api deploy window | `MetricsTab.test.tsx` |
| I1 | info | No file under `CDK/`, no pipeline YAML, no SQL script or migration in the 17-file diff | diff file list |
| I2 | info | PR build 29561 is green but was queued by hand; no branch policy makes it required | ADO |
| I3 | info | Sentinel dates normalised for `1900-01-01` and `0001-01-01` only; production data was not inspected | spec Assumptions |
| I4 | info | `pre-pr-review` PASS on local gates and `diff-vs-spec-reviewer` PASS (2 invocations, first HOLD on AC-7 fixed) | [../artifact1-tools/skills/pre-pr-review/log.md](../artifact1-tools/skills/pre-pr-review/log.md) |
| I5 | info | PR open since 2026-07-14, one reviewer requested | PR |

## Final decision

**HOLD.** Do not merge or deploy until (1) QualityCheck 445 has been queued on the branch and its whole-solution line coverage and dominant failure mode are recorded against the 87.72% baseline, and (2) the changed-code coverage report is attached to the PR.

The behaviour, the tests and the scope are fine. The hold is missing off-machine evidence, not an open defect. Once both items are recorded and show no regression, the recommendation changes to Merge (a human action), followed by the release steps in [./release.md](./release.md): App 472 then Api 440, Dev first, then Prod.
