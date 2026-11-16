# Artifact 3 — Story → Spec → Plan → PR (Full Traceability)

> Demo example: synthetic evidence (ids, run numbers, timings in chat logs/transcripts) modeled on the LBMH20-386 story shape. Artifact 5 numbers come from the real portal-SDLC.json.

> Purpose: prove a complete, gap-free chain from an original story to a merged PR — spec, plan, context map, human decisions, implementation, and verification. A reviewer must be able to trace story → spec → plan without gaps.

Story: [LBMH20-386](https://eci-solutions.atlassian.net/browse/LBMH20-386) — Portal: expand SiteMetrics on the license record detail Metrics tab. Chain: story → schema discovery ([MCP](../artifact2-mcp-subagents/mcp/transcript.md)) → spec → plan → implementation (branch `feature/LBMH20-386-sitemetrics-expansion`) → verification → PR 9220 ([artifact 4](../artifact4-deliverables/artifact4-main.md)). Shared numbers: 51 columns = 9 internal (hidden) + 4 already shown + 38 new in 6 cards (10, 7, 5, 6, 5, 5); 10 ACs; 36 new test cases.

---

## Traceability chain (at a glance) and Inventory list

| Stage | Artefact | Location | Notes |
|-------|----------|----------|-------|
| Story | Original work item with requester comment | [./story.md](./story.md) | 5 ticket ACs, out of scope, phases |
| Spec | AI-ready spec (AC-1..AC-10) | [./spec.md](./spec.md) | Story AC1 → AC-1,2,3,6; AC2 → AC-4; AC3 → AC-8; AC4 → AC-9; AC5 → AC-10; edge classes → AC-5, AC-7 |
| Spec chat logs | exported session (5 questions, reviewer FAIL then PASS) | [./spec-chat-log.md](./spec-chat-log.md) | Uses the schema discovery in artifact 2 |
| Plan | Implementation Plan (5 phases, gates G1-G5, TDD) | [./plan.md](./plan.md) | Phases match the story card; verification table maps every AC |
| Plan chat logs | exported session (one accepted, one changed, one rejected) | [./plan-chat-log.md](./plan-chat-log.md) | |
| Context map | hot/warm/cold files | [./context-map.md](./context-map.md) | Links the artifact 1 and 2 tools used |
| Verification | AC↔evidence table | [./verification.md](./verification.md) | Commands, 36 new tests, manual steps M1-M6 |
| Plan and spec review notes | human review + decisions | [./review-notes.md](./review-notes.md) | 17 decisions |

Where the tools from artifacts 1 and 2 appear in this chain: the `mssql-readonly` MCP produced the schema facts in the spec ([transcript](../artifact2-mcp-subagents/mcp/transcript.md)); `AGENTS.md` and the two rules were loaded as warm context ([context map](./context-map.md)); `pre-pr-review` and `diff-vs-spec-reviewer` ran in Phase 4 ([skill log](../artifact1-tools/skills/pre-pr-review/log.md), [subagent transcript](../artifact2-mcp-subagents/agents/diff-vs-spec-reviewer.transcript.md)).

---

### Verification

> Convincing evidence the key behaviour works. Tie each AC to proof collected locally. Full table with test names and manual steps: [./verification.md](./verification.md).

| AC | Proof (test name / manual step) | Result |
|----|--------------------------------|--------|
| AC-1 | `SiteMetricsMapper_ShouldMapSixCategoriesWith38Items_WhenRowIsFull`; manual M1 (6 cards, 38 rows) | pass |
| AC-2 | `SiteMetricsMapper_ShouldHideInternalColumns_WhenMappingRow` (9 cases); manual M2 (0 hits in response) | pass |
| AC-3 | `SiteMetricsMapper_ShouldNotRepeatColumnsFromExistingCards`; manual M1 | pass |
| AC-4 | `formatMetricValue.test.ts`, `MetricCategoryCard.test.tsx`; manual M3 | pass |
| AC-5 | `SiteMetricsMapper_ShouldMapSentinelDatesToNull` (2 cases); manual M4 | pass |
| AC-6 | `SiteMetricsMapper_ShouldKeepDisabledFeaturesAsFalseWithNullDates`, `..._WhenAllCategoryColumnsAreNull`; manual M5 | pass |
| AC-7 | `GetSiteMetrics_ShouldReturn200WithEmptyGroups_WhenNoRow`, `GetSiteMetrics_ShouldReturn404_WhenLicenseUnknown`; manual M6 | pass |
| AC-8 | diff file list (17 files, none in `CDK/`, pipelines, SQL); `SiteMetricsRepository_ShouldNotIssueWriteStatements` | pass |
| AC-9 | 36 new cases (Api 20, Data 3, Web 13) incl. fixture parity | pass |
| AC-10 | changed-code coverage 94.1% (.NET) / 95.2% (Web); warnings 17 vs 17 on `main` | pass (local only; off-machine gap in artifact 4) |

---

### Human decisions & review notes

> Full table with 17 decisions: [./review-notes.md](./review-notes.md). Highlights:

| Decision | Accepted / Changed / Rejected | Why |
|----------|-------------------------------|-----|
| Scope: all 38 categorised fields in 6 cards (not EDI only) | Accepted | The requester wants EDI first but also the other category fields; avoids a second story that repeats discovery. |
| Internal columns: exclude 9 (8 ingest-related + `SiteId`) | Changed | Discovery found 9 uncategorised columns; `SiteId` is the join key, not a metric. |
| Empty dates: `NULL`, `1900-01-01`, `0001-01-01` show `-` | Accepted | Both sentinels exist in dev data; requester: "empty dates should not look like 1900". |
| Coverage NFR: >= 90% on new/changed code only | Changed | Whole-solution baseline is 87.72% (build 29300); literal reading would block the story for an unrelated reason. |
| Plan: apply internal-column exclusion in mapper + catalog, not SQL | Changed | An xUnit theory can then cover all 9 columns; SQL-only filtering is untested by the .NET suite. |
| Plan: "View in fleet dashboard" link on cards | Rejected | Drill-in is out of scope (AC-4: plain name/value pairs); scope creep. |
