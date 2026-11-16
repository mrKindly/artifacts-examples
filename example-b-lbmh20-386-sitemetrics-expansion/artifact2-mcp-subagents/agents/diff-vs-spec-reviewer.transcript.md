# Diff vs spec review for LBMH20-386
_Exported on 7/15/2026 at 11:18:23 GMT+3 from Cursor (3.11.6)_

---

**User**

/pre-pr-review LBMH20-386

---

**Cursor**

Branch `feature/LBMH20-386-sitemetrics-expansion`, base `origin/main`. Build, tests and coverage results go to `docs/LBMH20-386/pre-pr-review.md`. For the spec-vs-diff check I'm starting the `diff-vs-spec-reviewer` subagent (`.cursor/agents/diff-vs-spec-reviewer.md`, read-only) with this prompt:

> Review `docs/LBMH20-386/spec.md` against `origin/main...HEAD`. Plan: `docs/LBMH20-386/plan.md`.

The subagent read the spec (AC-1 to AC-10), the plan, and the diff (16 files). Excluded internal columns taken from the spec: `SiteMetricsId`, `SiteId`, `IngestBatchId`, `IngestedAtUtc`, `SourceHost`, `SourceRowHash`, `CollectorVersion`, `RowVersion`, `IsDeleted`. Existing-card columns: `SnapshotDate`, `ActiveUserCount`, `RemoteClientCount`, `RemoteClientVersionList`.

`diff-vs-spec-reviewer` returned:

```
Verdict: HOLD
Spec: docs/LBMH20-386/spec.md | Diff: origin/main...HEAD | Files changed: 16

## AC coverage
| AC | Evidence (test or manual step) | Status |
|----|--------------------------------|--------|
| AC-1 | SiteMetricsMapper_ShouldMapSixCategoriesWith38Items_WhenRowIsFull; SiteMetricsMapper_ShouldKeepEdiGroupFirstWithTenItems | covered |
| AC-2 | SiteMetricsMapper_ShouldHideInternalColumns_WhenMappingRow (9 cases) | covered |
| AC-3 | SiteMetricsMapper_ShouldNotRepeatColumnsFromExistingCards; MetricsTab.test.tsx "renders six category cards after the four existing cards" | covered |
| AC-4 | MetricCategoryCard.test.tsx, formatMetricValue.test.ts | covered |
| AC-5 | SiteMetricsMapper_ShouldMapSentinelDatesToNull (2 cases) | covered |
| AC-6 | SiteMetricsMapper_ShouldKeepDisabledFeaturesAsFalseWithNullDates; SiteMetricsMapper_ShouldReturnNullValues_WhenAllCategoryColumnsAreNull | covered |
| AC-7 | SiteMetricsMapper_ShouldReturnEmptyGroups_WhenRowIsMissing; SiteMetricsRepository_ShouldReturnNull_WhenNoRowForLicense | gap |
| AC-8 | no path under CDK/, *.yaml, SQL scripts; SiteMetricsRepository_ShouldNotIssueWriteStatements | covered |
| AC-9 | tests present in Api.Tests, Data.Tests, Web; siteMetrics.fixture.test.ts | covered |
| AC-10 | not a diff check (coverage and warnings are computed by the parent) | n/a |

## Scope
- Outside spec scope: none

## Data exposure
- Excluded columns found in payload/UI/fixtures: none

## Display rules
- Yes/No with muted No: formatMetricValue.ts, tested in formatMetricValue.test.ts
- '-' for null/empty/sentinel dates: sentinel normalised in SiteMetricsMapper.cs, '-' rendered in formatMetricValue.ts, both tested
- No drill-in: MetricCategoryCard.test.tsx "renders no links or buttons"

## Findings
- [blocker] AC-7: spec says the endpoint returns HTTP 200 with empty groups when a license has no SiteMetrics row. Only the mapper and repository are tested; no test exercises the endpoint response. Add endpoint-level evidence.
- [info] Api/Licenses/Metrics/SiteMetricsMapper.cs: the internal-column deny-list is applied in addition to missing category rows; good defence in depth, covered by the theory test.

## Refused
- none
```

HOLD recorded in `pre-pr-review.md`. Add an endpoint-level test for a license with no SiteMetrics row, then re-run.

---

**User**

Added the endpoint tests. /pre-pr-review LBMH20-386

---

**Cursor**

Re-running. The diff is now 17 files; `LicenseSiteMetricsEndpointTests.cs` adds two tests. Starting `diff-vs-spec-reviewer` again with the same prompt.

`diff-vs-spec-reviewer` returned:

```
Verdict: PASS
Spec: docs/LBMH20-386/spec.md | Diff: origin/main...HEAD | Files changed: 17

## AC coverage
| AC | Evidence (test or manual step) | Status |
|----|--------------------------------|--------|
| AC-1 | SiteMetricsMapper_ShouldMapSixCategoriesWith38Items_WhenRowIsFull; SiteMetricsMapper_ShouldKeepEdiGroupFirstWithTenItems | covered |
| AC-2 | SiteMetricsMapper_ShouldHideInternalColumns_WhenMappingRow (9 cases) | covered |
| AC-3 | SiteMetricsMapper_ShouldNotRepeatColumnsFromExistingCards; MetricsTab.test.tsx | covered |
| AC-4 | MetricCategoryCard.test.tsx, formatMetricValue.test.ts | covered |
| AC-5 | SiteMetricsMapper_ShouldMapSentinelDatesToNull (2 cases) | covered |
| AC-6 | SiteMetricsMapper_ShouldKeepDisabledFeaturesAsFalseWithNullDates; SiteMetricsMapper_ShouldReturnNullValues_WhenAllCategoryColumnsAreNull; MetricsTab.test.tsx "renders disabled features as muted No and dash dates" | covered |
| AC-7 | GetSiteMetrics_ShouldReturn200WithEmptyGroups_WhenNoRow; GetSiteMetrics_ShouldReturn404_WhenLicenseUnknown; SiteMetricsMapper_ShouldReturnEmptyGroups_WhenRowIsMissing | covered |
| AC-8 | no path under CDK/, *.yaml, SQL scripts; SiteMetricsRepository_ShouldNotIssueWriteStatements | covered |
| AC-9 | Api.Tests, Data.Tests, 4 Web test files, fixture parity test | covered |
| AC-10 | not a diff check (coverage and warnings are computed by the parent) | n/a |

## Scope
- Outside spec scope: none

## Data exposure
- Excluded columns found in payload/UI/fixtures: none

## Display rules
- all rules implemented once per layer and tested

## Findings
- [info] Data.Tests/Metrics/SiteMetricsRepositoryTests.cs: repository tests run on an in-memory provider, not SQL Server. Column names are verified by the schema discovery quoted in the spec, not by these tests.

## Refused
- none
```

Merged this verdict into `pre-pr-review.md`: **PASS (local gates)**. The subagent says nothing about approval or deploy readiness; those stay with you.
