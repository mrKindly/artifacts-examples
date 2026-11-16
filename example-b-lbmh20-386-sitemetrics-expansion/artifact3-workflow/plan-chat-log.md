# Implementation plan for LBMH20-386
_Exported on 7/13/2026 at 18:10:37 GMT+3 from Cursor (3.11.6)_

---

**User**

Create the implementation plan for `docs/LBMH20-386/spec.md`. Five phases like the story card. TDD, human gates, no code in the plan.

---

**Cursor**

Read the spec, `AGENTS.md`, the two rules, and the code the spec touches: `SiteMetricsMapper.cs`, `SiteMetricsResponse.cs`, `SiteMetricsRepository.cs`, `MetricsTab.tsx`, the Snapshot card, the mock fixture, and the existing tests next to them. Nothing under `CDK/` or the pipeline YAML is involved.

Confirmed: four cards and four columns today; an additive response change is possible; the Snapshot date formatter exists. No blocking question. Assumptions are stated in the plan (formatter reuse, metadata readable through existing access, normalisation location).

Draft 1 of the plan has five phases, each leaving a working state, tests before code in each step, and gates G1 to G5. Three proposals I want to flag:

1. Order: mapper and API first (Phase 2), UI second (Phase 3), so the payload is reviewed before any UI is built.
2. Apply the internal-column exclusion in the repository SQL (`WHERE` on the allowed list), so internal columns never leave the database.
3. As a small extra in Phase 3, add a "View in fleet dashboard" link on each category card, because the dashboard already has the same categories.

---

**User**

1. Phase order and TDD structure — fine, keep. Add a gate after the payload is defined (before UI work).
2. Not in SQL. Put the exclusion list in a catalog next to the data layer and apply it in the mapper, so a mapper test covers all 9 columns, including the case where someone adds a category row for an internal column. A SQL-only filter would be untested by the xUnit suite.
3. No. The story says no drill-in on this page, and the fleet dashboard link is explicitly out of scope. Remove it and list it under Out Of Scope.

---

**Cursor**

Updated the plan:

- Phase order kept; added gate **G2** after the response shape is reviewed, before UI work.
- Exclusion moved to `SiteMetricsColumnCatalog` plus the mapper; the repository reads only the metadata and the license's row. Added the mapper theory test for all 9 columns "even when a category row exists" as the first test in Phase 2.
- Drill-in link removed; added to Out Of Scope with the reason.
- Added the explicit AC-to-verification table (template format) and the warning-count and changed-code coverage steps to Phase 4.

Wrote `docs/LBMH20-386/plan.md`.

---

**User**

Approved for implementation, Phase 2 first. Stop at G2.

---

**Cursor**

Plan saved. Decisions are recorded in `docs/LBMH20-386/review-notes.md`: one accepted (phase order and TDD), one changed (where exclusion lives), one rejected (drill-in link). Starting Phase 2; I'll stop at G2.
