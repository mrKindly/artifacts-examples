# Pre-PR review for LBMH20-386
_Exported on 7/15/2026 at 11:26:40 GMT+3 from Cursor (3.11.6)_

---

**User**

/pre-pr-review LBMH20-386

---

**Cursor**

I'll run the pre-PR checks on `feature/LBMH20-386-sitemetrics-expansion` against `docs/LBMH20-386/spec.md`: scope, build, tests, AC evidence, changed-code coverage, and the `diff-vs-spec-reviewer` subagent.

Spec found with AC-1 to AC-10. Plan found. Base `origin/main`.

Diff: 16 files (Api 3, Data 2, Web 5, tests 6). No path under `CDK/`, no pipeline YAML, no SQL script. Every file is named in the plan.

Build and tests:

| Step | Command | Exit | Result |
|------|---------|------|--------|
| Build | `dotnet build Api -c Debug` | 0 | 17 warnings on branch, 17 on `main` |
| Api tests | `dotnet test Api.Tests --filter "FullyQualifiedName~SiteMetrics"` | 0 | 27 passed (18 new mapper cases + 9 existing SiteMetrics tests) |
| Data tests | `dotnet test Data.Tests` | 0 | 64 passed |
| Web build | `npm run build` (Web) | 0 | output in `Api/wwwroot` |
| Web tests | `npx vitest run src/features/license-detail/metrics src/mocks/fixtures` | 0 | 13 passed |

AC to evidence: AC-1 to AC-6, AC-8, AC-9 have tests. **AC-7 (license with no SiteMetrics row)** is proved only at mapper and repository level; there is no test that the endpoint returns HTTP 200 with empty groups.

Changed-code coverage (coverlet cobertura intersected with `git diff -U0`; the script is not present, so computed by hand): **88.6%** (178 of 201 changed executable lines). Uncovered: the not-found branch in the endpoint handler and two repository error paths.

Reviewer subagent `diff-vs-spec-reviewer`: **HOLD** — "AC-7: only mapper and repository tests; no endpoint-level test that a missing row returns 200 with empty groups". Coverage is not the reviewer's check; the 88.6% above is this skill's own finding.

Wrote `docs/LBMH20-386/pre-pr-review.md`.

**Verdict: HOLD.** Fix: add an endpoint test for AC-7 (200, empty groups) and one for unknown license (404, unchanged behaviour); re-run.

---

**User**

Added the endpoint tests. /pre-pr-review LBMH20-386

---

**Cursor**

Diff: 17 files (Api 3, Data 2, Web 5, tests 7: the endpoint test file is now included). Scope clean; no `CDK/`, pipeline YAML or SQL script.

| Step | Command | Exit | Result |
|------|---------|------|--------|
| Build | `dotnet build Api -c Debug` | 0 | 17 warnings (same as `main`), 15.4 s |
| Api tests | `dotnet test Api.Tests` | 0 | 508 passed, 0 failed (20 new cases) |
| Data tests | `dotnet test Data.Tests` | 0 | 64 passed, 0 failed (3 new), 4.3 s |
| CDK tests | `dotnet test CDK.Tests` | 0 | 22 passed (unchanged, run to confirm no CDK impact) |
| Web build | `npm run build` | 0 | 27 s |
| Web scoped tests | `npx vitest run src/features/license-detail/metrics src/mocks/fixtures` | 0 | 4 files, 13 passed |
| Web full | `npm run test:ci` | 1 | same 2 failing files as on `main`; no new failures |

AC to evidence: AC-1 to AC-10 each map to at least one test or manual step (`docs/LBMH20-386/verification.md`). AC-7 now has `GetSiteMetrics_ShouldReturn200WithEmptyGroups_WhenNoRow`.

Changed-code coverage: **94.1%** (192 of 204 changed executable lines). The 12 uncovered lines are the defensive `SqlException` catch paths in `SiteMetricsRepository`; the spec does not define behaviour for a database outage beyond existing handling. Web changed lines: 95.2% (118 of 124).

Reviewer subagent: **PASS**, one non-blocking note ("Data.Tests exercises the repository against an in-memory provider, not SQL Server; column casing is covered by the MCP discovery in the spec").

**Verdict: PASS (local gates).**

Human follow-ups, not performed by this skill:

1. Queue QualityCheck 445 on the branch (not run; baseline 87.72% line on build 29300).
2. Attach the changed-code coverage figure to the PR.
3. Request human review; merge and deploy stay with humans.

Wrote `docs/LBMH20-386/pre-pr-review.md`. No file in the repository was changed by this skill other than that report.
