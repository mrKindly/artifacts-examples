# Implementation Plan — LBMH20-386

Generated in Plan Mode from [./spec.md](./spec.md) (course skill from MK6, template `plan.template.md`); session in [./plan-chat-log.md](./plan-chat-log.md); human decisions in [./review-notes.md](./review-notes.md). Phases match the five phases of the story card ([./story.md](./story.md)). No code in this plan.

### Confirmed Facts

- The Metrics tab shows four cards and the endpoint returns data for four SiteMetrics columns today (spec, Current Behavior).
- `SpruceMetrics.dbo.SiteMetrics` has 51 columns: 9 internal (excluded), 4 already shown, 38 new; `dbo.SiteMetricsCategory` has 42 rows in 6 categories (EDI 10, Integrations 7, Communications 5, Mobile & Remote 8, Reporting 5, Environment 7) ([../artifact2-mcp-subagents/mcp/transcript.md](../artifact2-mcp-subagents/mcp/transcript.md)).
- New fields after de-dupe: 10 + 7 + 5 + 6 + 5 + 5 = 38; types: 18 Yes/No, 17 date, 2 text, 1 number.
- Sentinel dates in data: `1900-01-01`, `0001-01-01`, plus `NULL`.
- Local run is `http://localhost:5000` after `npm run build`; tests are `dotnet test` (Api.Tests, Data.Tests, CDK.Tests) and `npm run test:ci` in Web, which exits 1 on `main` today ([../artifact1-tools/AGENTS.md](../artifact1-tools/AGENTS.md)).
- No schema, CDK or pipeline change is allowed (AC-8).
- Quality bar: >= 90% line coverage on new/changed code, zero new warnings (AC-10); QualityCheck 445 baseline is 87.72% line (build 29300) and is not the target.

### Assumptions

- The existing Snapshot card date formatter can be reused for the new date fields.
- Category metadata can be read through the existing data access (same database as the SiteMetrics row) without new objects.
- The API response can gain a new list of groups without breaking the four existing cards (additive change).
- Data.Tests uses an in-memory provider; column names are trusted from the schema discovery, not from those tests.
- Sentinel dates are normalised to empty once, in the API mapper; the Web formatter renders empty as `-`.
- The internal-column exclusion list is applied in the mapper (not in SQL), so that tests cover it.

### Files / Modules Involved

Paths are modeled on the Portal layout and confirmed by the developer in the codebase before implementation.

| Area | Path | Change |
|------|------|--------|
| API | `Api/Licenses/Metrics/SiteMetricsMapper.cs` | Modify: build category groups, apply exclusion and de-dupe, normalise sentinel dates |
| API | `Api/Licenses/Metrics/SiteMetricsResponse.cs` | Modify: add group and item shapes to the response |
| API | `Api/Licenses/Metrics/MetricValueKind.cs` | New: value kinds (Yes/No, date, text, number) |
| Data | `Data/Metrics/SiteMetricsRepository.cs` | Modify: read category metadata and the license's row (SELECT only) |
| Data | `Data/Metrics/SiteMetricsColumnCatalog.cs` | New: internal exclusion list and already-shown list |
| Web | `Web/src/features/license-detail/metrics/MetricsTab.tsx` | Modify: render category cards after the four existing cards |
| Web | `Web/src/features/license-detail/metrics/MetricCategoryCard.tsx` | New: one card, name/value rows |
| Web | `Web/src/features/license-detail/metrics/formatMetricValue.ts` | New: single value formatter (Yes/No, `-`, number, date) |
| Web | `Web/src/features/license-detail/metrics/metrics.types.ts` | Modify: group and item types |
| Web | `Web/src/mocks/fixtures/siteMetrics.fixture.ts` | Modify: 6 groups, 38 items |
| Tests (.NET) | `Api.Tests/Licenses/Metrics/SiteMetricsMapperTests.cs` (new), `Api.Tests/Licenses/Metrics/LicenseSiteMetricsEndpointTests.cs` (modify), `Data.Tests/Metrics/SiteMetricsRepositoryTests.cs` (modify) | New/extended tests |
| Tests (Web) | `formatMetricValue.test.ts`, `MetricCategoryCard.test.tsx`, `MetricsTab.test.tsx`, `siteMetrics.fixture.test.ts` (same folder as sources) | New/extended tests |

Not touched: `CDK/`, pipeline YAML, SQL scripts, migrations, the four existing cards' components, the fleet dashboard.

### Implementation Plan

Each phase leaves the system working and testable. Tests are written before the code they check (TDD). Gates marked **[GATE]** need explicit human approval before the next phase starts.

**Phase 1 — Spec and schema discovery** (done before this plan)
1. Discover schema through the read-only MCP; lock include/exclude lists, display rules and numbers (51 = 9 + 4 + 38; 6 categories).
2. Run the spec reviewer until PASS.
3. **[GATE G1]** Human approves the spec and the scope cap (no drill-in, no EDI-only slice).

**Phase 2 — Mapping and API payload** (usable state: API returns the new groups; UI unchanged)
1. Test first: mapper tests for full row (6 groups, 38 items, order and counts), internal-column hiding (all 9, including when a category row exists), de-dupe (4 columns), sentinel dates, disabled features, all-`NULL` row, skipped column without category, missing row.
2. Implement the exclusion/already-shown catalog and the mapper until the tests pass.
3. Test first: repository tests (reads metadata and row for a license; returns nothing when no row; issues only read statements); then extend the repository.
4. Test first: endpoint tests (success with groups; success with empty groups when no row; not-found for unknown license); then extend the response.
5. **[GATE G2]** Human reviews the response shape against the spec (names of fields in the payload, no internal column present) before any UI work.

**Phase 3 — Metrics UI cards** (usable state: tab shows six new cards beside the four existing ones)
1. Test first: formatter tests (Yes, No muted, `-` for empty text, number and `NULL` Yes/No, thousands separators, `0`).
2. Implement the formatter; test first the category card (one row per item, muted only on `No`, no links or buttons); implement the card.
3. Test first the tab (six cards after four existing, internal labels absent, empty state when no groups, disabled features shown as muted `No` and `-`); then wire the tab.
4. Update mock fixtures to 6 groups and 38 items; add the parity test (counts match the contract).
5. Build the SPA and look at the tab at `http://localhost:5000` for three prepared licenses (full data, disabled features, no row). **[GATE G3]** Human confirms the look (muted `No`, `-` for empty) with the requester's example.

**Phase 4 — Pre-PR verification** (usable state: evidence package ready)
1. Run scoped build and tests; compare warning count with `main`.
2. Compute changed-code coverage (target >= 90%); fix gaps with tests, not exclusions.
3. Run `pre-pr-review` and the `diff-vs-spec-reviewer` subagent; fill `verification.md` with an AC-to-proof table.
4. **[GATE G4]** Human reviews the evidence and confirms scope: no CDK/pipeline/schema file in the diff.

**Phase 5 — PR, approve, deploy** (usable state: reviewed PR with a release note)
1. Open the PR with the spec, verification and coverage figure attached; queue QualityCheck 445 on the branch (human) and read its coverage line against the 87.72% baseline.
2. Human review and approval; merge is a human action.
3. Deploy is a human action through the `deploy` skill: App (472) then Api (440), Dev first, then Prod; write release notes with rollback.
4. **[GATE G5]** Human merge decision and deploy approval per environment.

### Risks

- Sentinel values other than the two known ones in production data would display as dates.
- Category metadata drift: a new category row for an internal column is blocked only by the exclusion list.
- Contract drift between API, Web types and mock fixtures; mitigated by the parity test.
- Whole-solution coverage (87.72%) keeps QualityCheck red regardless of this story; evidence must therefore be changed-code coverage plus the QualityCheck delta.
- `npm run test:ci` exits 1 on `main`; Web evidence must be scoped runs plus an unchanged failure set.
- PR can sit open while waiting for a single reviewer; keep the diff small and evidence attached.

### Out Of Scope

- Schema, migrations, CDK, pipeline changes.
- Drill-in from the license Metrics tab to the fleet dashboard (proposed by the planning agent and **rejected**, see [./review-notes.md](./review-notes.md)).
- Feature flag, per-user card configuration, editing, filters, export.
- Restyling or replacing the four existing cards.
- Fixing the pre-existing `npm run test:ci` failures or the QualityCheck baseline.
- Showing the 9 internal columns; replacing the licensing `ediNames` string.

### Verification

| ID | Acceptance Criterion | Verifiable By |
|----|----------------------|---------------|
| AC-1 | 6 groups, 38 items, order and labels from metadata | Automated Test (mapper, repository), Manual Test (tab at localhost:5000) |
| AC-2 | None of the 9 internal columns in payload, UI or fixture | Automated Test (mapper theory), API Check, UI Check |
| AC-3 | 4 already-shown columns not repeated; existing cards unchanged | Automated Test (mapper, tab), Manual Test |
| AC-4 | Yes/No with muted No; `-` for empty; plain name/value rows | Automated Test (formatter, card), UI Check |
| AC-5 | Sentinel and `NULL` dates show `-`; real dates formatted like Snapshot | Automated Test (mapper theory, formatter), Manual Test |
| AC-6 | Disabled features and all-`NULL` row render correctly | Automated Test (mapper, tab), Manual Test |
| AC-7 | No row: success with empty groups and existing empty state; unknown license: not-found | Automated Test (endpoint, repository, tab), API Check |
| AC-8 | No schema/CDK/pipeline/SQL change; read-only statements | Review (diff file list), Automated Test (repository) |
| AC-9 | Tests exist for mapping, endpoint, repository, UI and fixture parity | Automated Test (test inventory in `verification.md`) |
| AC-10 | >= 90% changed-code coverage; zero new warnings | Automated Test (coverage run), Manual Test (warning count vs `main`) |
