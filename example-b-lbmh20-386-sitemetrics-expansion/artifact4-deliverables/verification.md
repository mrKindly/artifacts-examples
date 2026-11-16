# Verification — LBMH20-386 (PR 9220)

> Condensed illustrative reconstruction (synthetic ids and numbers modeled on the story shape). Snapshot taken 2026-07-15, before any deploy.

Working result: PR [9220](https://dev.azure.com/ECI-LBMH/LBMH-Spruce/_git/LBMH.Services.Portal/pullrequest/9220) (demo reference link; org `ECI-LBMH`, project `LBMH-Spruce`), branch `feature/LBMH20-386-sitemetrics-expansion` into `main`, head `a41c7e9` (synthetic), author `dmares`. Local proof is in [../artifact3-workflow/verification.md](../artifact3-workflow/verification.md); this file adds the evidence collected **off the developer machine** and states what is missing.

## 1. Local proof (summary)

| AC | Proof | Result |
|----|-------|--------|
| AC-1 | Mapper tests (6 groups, 38 items) + manual M1 | pass |
| AC-2 | Theory over 9 internal columns + manual M2 | pass |
| AC-3 | De-dupe test + manual M1 | pass |
| AC-4 | Formatter and card tests + manual M3 | pass |
| AC-5 | Sentinel date theory + manual M4 | pass |
| AC-6 | Disabled / all-`NULL` tests + manual M5 | pass |
| AC-7 | Endpoint 200 empty groups, 404 unknown + manual M6 | pass |
| AC-8 | 17-file diff, none in `CDK/`, pipelines, SQL | pass |
| AC-9 | 36 new test cases (Api 20, Data 3, Web 13) | pass |
| AC-10 | changed-code coverage 94.1% (.NET, 192/204) and 95.2% (Web, 118/124); warnings 17 vs 17 on `main` | pass (local only) |

## 2. Evidence off the local machine

| Check | Where | Result | Evidence |
|-------|-------|--------|----------|
| PR build (restore, `dotnet build Api`, `dotnet test Api.Tests` and `Data.Tests`, `npm ci`, `npm run build`) | ADO, run 29561 (synthetic id), queued by hand against `refs/pull/9220/merge` on 2026-07-14 | **ok**, 7 min 48 s | Run link on the PR. Not a required check: the repository has no branch policies (empty policy list in [portal-SDLC.json](../artifact5-ai-use-cases/portal-SDLC.json)), so this run is evidence, not a gate |
| QualityCheck 445 on the branch (Sonar, Mend, Coverlet gate 90%) | ADO | **not run** | Manual pipeline; nobody queued it on this branch. Last known run for context: build 29300, 488 passed, 87.72% line coverage, gate failed |
| Whole-solution line coverage after this change | ADO / QualityCheck | **unknown** | Baseline 87.72% (<90%) on build 29300; no run on the branch |
| Changed-code line coverage as an artefact on the PR | PR | **not attached** | 94.1% exists only in local `pre-pr-review.md` and local command output |
| Dev environment smoke | AWS `Spruce-Next Dev` | **not done** | No deploy before the merge decision, by design (see release notes) |

Automated checks: PR build 29561 (ok); no other CI evidence exists.
Manual QA: [../artifact3-workflow/verification.md](../artifact3-workflow/verification.md) manual steps M1-M6 (local only).

## 3. Gaps

1. QualityCheck 445 was not queued on the branch, so there is no off-machine reading of the Coverlet gate for this change.
2. Changed-code coverage (94.1%) is not attached to the PR, so a reviewer cannot confirm it without re-running locally.
3. No Dev smoke yet (expected before merge; becomes a release step).

No product defect was found by any check. The gaps are missing evidence, not failing evidence.
