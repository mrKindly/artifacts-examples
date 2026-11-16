# Verification — LBMH20-386

> Condensed illustrative reconstruction (synthetic). Counts and timings are modeled; the build, `Data.Tests` and Web build timings (15.4 s, 4.3 s, 27 s) reuse the 2026-07-15 workstation sample in [../artifact5-ai-use-cases/portal-SDLC.json](../artifact5-ai-use-cases/portal-SDLC.json).

Collected locally on 2026-07-15 on branch `feature/LBMH20-386-sitemetrics-expansion` (head `a41c7e9`, synthetic). Automated proof is xUnit and Vitest. Manual proof is the running app at `http://localhost:5000` (real Okta, dev databases), after `npm run build`. Spec: [./spec.md](./spec.md). Plan: [./plan.md](./plan.md). Off-machine evidence is in [../artifact4-deliverables/verification.md](../artifact4-deliverables/verification.md).

## Commands and results

| Step | Command | Exit | Result |
|------|---------|------|--------|
| Build | `dotnet build Api -c Debug` | 0 | 0 errors, 17 warnings (17 on `main`), 15.4 s |
| Api tests | `dotnet test Api.Tests` | 0 | 508 passed, 0 failed (20 new cases) |
| Data tests | `dotnet test Data.Tests` | 0 | 64 passed, 0 failed (3 new), 4.3 s |
| CDK tests | `dotnet test CDK.Tests` | 0 | 22 passed (unchanged; confirms no CDK impact) |
| Web build | `cd Web && npm run build` | 0 | 27 s, output in `Api/wwwroot` |
| Web scoped | `npx vitest run src/features/license-detail/metrics src/mocks/fixtures` | 0 | 4 files, 13 tests passed |
| Web full | `npm run test:ci` | 1 | Test files: 2 failed, 61 passed (63). On `main`: 2 failed, 57 passed (59). Same 2 failing files, no new failures |
| Coverage (.NET) | `dotnet test Api.Tests --collect:"XPlat Code Coverage"` + `Data.Tests`, intersected with `git diff -U0` | 0 | changed lines 204, covered 192 = **94.1%** |
| Coverage (Web) | `npx vitest run --coverage` (scoped), intersected with changed lines | 0 | changed lines 124, covered 118 = **95.2%** |
| TypeScript / lint | `npx tsc --noEmit`, `npx eslint src/features/license-detail src/mocks` | 0 | 0 new warnings |

The 12 uncovered C# lines are the defensive `SqlException` catch paths in `SiteMetricsRepository`; the spec adds no new behaviour for a database outage.

## New tests (36 cases)

| Project | File | Test (case count) |
|---------|------|-------------------|
| Api.Tests | `SiteMetricsMapperTests.cs` | `SiteMetricsMapper_ShouldMapSixCategoriesWith38Items_WhenRowIsFull` (1); `SiteMetricsMapper_ShouldKeepEdiGroupFirstWithTenItems` (1); `SiteMetricsMapper_ShouldHideInternalColumns_WhenMappingRow` (9, theory); `SiteMetricsMapper_ShouldNotRepeatColumnsFromExistingCards` (1); `SiteMetricsMapper_ShouldMapSentinelDatesToNull` (2, theory); `SiteMetricsMapper_ShouldKeepDisabledFeaturesAsFalseWithNullDates` (1); `SiteMetricsMapper_ShouldReturnNullValues_WhenAllCategoryColumnsAreNull` (1); `SiteMetricsMapper_ShouldReturnEmptyGroups_WhenRowIsMissing` (1); `SiteMetricsMapper_ShouldSkipColumnsWithoutCategory` (1) = 18 |
| Api.Tests | `LicenseSiteMetricsEndpointTests.cs` | `GetSiteMetrics_ShouldReturn200WithEmptyGroups_WhenNoRow` (1); `GetSiteMetrics_ShouldReturn404_WhenLicenseUnknown` (1) = 2 |
| Data.Tests | `SiteMetricsRepositoryTests.cs` | `SiteMetricsRepository_ShouldReadCategoryMetadataAndRow_ForLicense`; `SiteMetricsRepository_ShouldReturnNull_WhenNoRowForLicense`; `SiteMetricsRepository_ShouldNotIssueWriteStatements` = 3 |
| Web | `formatMetricValue.test.ts` | "renders true as Yes"; "renders false as No and marks it muted"; "renders null Yes/No, text, number and date as dash"; "renders a real date in the Snapshot card format"; "renders numbers with thousands separators and zero as 0" = 5 |
| Web | `MetricCategoryCard.test.tsx` | "renders one name/value row per item"; "mutes only No values"; "renders no links or buttons" = 3 |
| Web | `MetricsTab.test.tsx` | "renders six category cards after the four existing cards"; "does not render internal column labels"; "shows the existing empty state when groups are empty"; "renders disabled features as muted No and dash dates" = 4 |
| Web | `siteMetrics.fixture.test.ts` | "mock fixture has the same six groups and 38 items as the API contract" = 1 |

18 + 2 + 3 + 5 + 3 + 4 + 1 = 36 (Api 20, Data 3, Web 13).

## AC to proof

| AC | Proof (test name / manual step) | Result |
|----|---------------------------------|--------|
| AC-1 | `SiteMetricsMapper_ShouldMapSixCategoriesWith38Items_WhenRowIsFull`; `SiteMetricsMapper_ShouldKeepEdiGroupFirstWithTenItems`; `SiteMetricsRepository_ShouldReadCategoryMetadataAndRow_ForLicense`; Web "renders six category cards after the four existing cards". Manual M1: license `LIC-40417` shows 6 cards below the four existing ones; rows counted per card 10, 7, 5, 6, 5, 5 = 38 | pass |
| AC-2 | `SiteMetricsMapper_ShouldHideInternalColumns_WhenMappingRow` (9 cases, each with a category row planted); Web "does not render internal column labels"; fixture test. Manual M2: Network tab response for `GET /api/licenses/LIC-40417/site-metrics` searched for all 9 names: 0 hits | pass |
| AC-3 | `SiteMetricsMapper_ShouldNotRepeatColumnsFromExistingCards`; Web "renders six category cards after the four existing cards". Manual M1: Snapshot, Features, Remote Clients, Remote Client Versions identical to `main`; `SnapshotDate`, `ActiveUserCount`, `RemoteClientCount`, `RemoteClientVersionList` appear once | pass |
| AC-4 | Web: "renders true as Yes", "renders false as No and marks it muted", "renders null Yes/No, text, number and date as dash", "renders numbers with thousands separators and zero as 0", "mutes only No values", "renders no links or buttons". Manual M3: `No` rows render in secondary grey, `Yes` rows in normal text, no row is clickable | pass |
| AC-5 | `SiteMetricsMapper_ShouldMapSentinelDatesToNull` (`1900-01-01`, `0001-01-01`); Web "renders a real date in the Snapshot card format". Manual M4: `LIC-40417` "EDI Catalog Last Used" shows `-` (row holds `1900-01-01`); "EDI Last Used" shows `2026-07-11` like the Snapshot date style | pass |
| AC-6 | `SiteMetricsMapper_ShouldKeepDisabledFeaturesAsFalseWithNullDates`; `SiteMetricsMapper_ShouldReturnNullValues_WhenAllCategoryColumnsAreNull`; Web "renders disabled features as muted No and dash dates". Manual M5: `LIC-40533` EDI Enabled `No` muted, EDI Last Used `-`, all six cards visible | pass |
| AC-7 | `GetSiteMetrics_ShouldReturn200WithEmptyGroups_WhenNoRow`; `GetSiteMetrics_ShouldReturn404_WhenLicenseUnknown`; `SiteMetricsMapper_ShouldReturnEmptyGroups_WhenRowIsMissing`; `SiteMetricsRepository_ShouldReturnNull_WhenNoRowForLicense`; Web "shows the existing empty state when groups are empty". Manual M6: `LIC-40998` shows the existing empty state, response HTTP 200; unknown id `LIC-00000` returns 404 | pass |
| AC-8 | Diff file list: 17 files, none under `CDK/`, none `*.yaml`, no SQL script or migration. `SiteMetricsRepository_ShouldNotIssueWriteStatements`. `CDK.Tests` unchanged (22 passed). MCP server never used to write | pass |
| AC-9 | 36 new cases listed above across Api.Tests, Data.Tests, Web, including the fixture parity test | pass |
| AC-10 | Changed-code line coverage 94.1% (.NET, 192 of 204) and 95.2% (Web, 118 of 124); `dotnet build` 17 warnings vs 17 on `main`; `tsc` and ESLint 0 new warnings | pass (local) |

Local evidence only. Whether QualityCheck 445 on the branch and the changed-code coverage figure are accepted off-machine is tracked in [../artifact4-deliverables/verification.md](../artifact4-deliverables/verification.md).

## Manual steps

1. `cd Web && npm run build`, then `dotnet run --project Api --launch-profile http`; open `http://localhost:5000` and sign in through Okta.
2. **M1** Open license `LIC-40417` and the Metrics tab. Confirm four existing cards, then six new cards in the order EDI, Integrations, Communications, Mobile & Remote, Reporting, Environment. Count rows per card.
3. **M2** In browser DevTools, open the `site-metrics` response and search for each of the 9 internal names.
4. **M3** Check that `No` is muted and `Yes` is not; try clicking rows (nothing happens).
5. **M4** Find "EDI Catalog Last Used"; confirm `-`, not a 1900 date.
6. **M5** Open `LIC-40533` (EDI disabled); confirm muted `No` and `-` dates.
7. **M6** Open `LIC-40998` (no SiteMetrics row) and `LIC-00000` (unknown); confirm empty state and 404 respectively.
8. Compare the four existing cards with a screenshot from `main`.
