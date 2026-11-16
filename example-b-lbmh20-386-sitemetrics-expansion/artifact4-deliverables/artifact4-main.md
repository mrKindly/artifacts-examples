# Artifact 4 — Ship It (Working Result + Release Readiness)

> Demo example: synthetic evidence (ids, run numbers, timings in chat logs/transcripts) modeled on the LBMH20-386 story shape. Artifact 5 numbers come from the real portal-SDLC.json.

> Purpose: prove a working, review-ready and release-ready result that matches the spec. A reviewer must see the working result, convincing verification, review notes (summary, blockers, gaps), release notes with a rollback plan, known risks, and a final merge / deployability decision.

Story: [LBMH20-386](https://eci-solutions.atlassian.net/browse/LBMH20-386). Spec: [../artifact3-workflow/spec.md](../artifact3-workflow/spec.md). Snapshot date of every file here: 2026-07-15.

**Reading note.** This folder is a teaching reference that stops at a **Hold** decision on purpose, so a mentee can see what a decision with missing evidence looks like. [portal-SDLC.json](../artifact5-ai-use-cases/portal-SDLC.json) was pulled later (2026-07-16) and lists PR 9220 as a deployed exemplar; this demo does not reproduce those later events and its Hold is not meant to contradict them.

---

## Working result (at a glance) and Inventory list

| Item | Location / link |
|------|-----------------|
| PR | [https://dev.azure.com/ECI-LBMH/LBMH-Spruce/_git/LBMH.Services.Portal/pullrequest/9220](https://dev.azure.com/ECI-LBMH/LBMH-Spruce/_git/LBMH.Services.Portal/pullrequest/9220) (demo reference link; org `ECI-LBMH`, project `LBMH-Spruce`) |
| Branch / head | `feature/LBMH20-386-sitemetrics-expansion` into `main`, head `a41c7e9` (synthetic), 17 files, author `dmares` |
| Verification | [./verification.md](./verification.md) |
| Review notes | [./review.md](./review.md) |
| Release notes | [./release.md](./release.md) |
| Chat logs | [./review-chat-log.md](./review-chat-log.md), [./release-chat-log.md](./release-chat-log.md) |
| Final decision | **Hold** (not Blocked by a defect; blocked by missing off-machine evidence) |

What was built: the license record Metrics tab shows six new category cards (EDI 10, Integrations 7, Communications 5, Mobile & Remote 6, Reporting 5, Environment 5 = 38 fields) beside the four existing cards; 9 internal columns hidden; Yes/No with muted `No`; `-` for empty and sentinel dates; no schema, CDK or pipeline change.

---

### Verification evidence

| AC / behaviour | Proof (test name / manual step) | Result |
|----------------|--------------------------------|--------|
| AC-1 Mapping (6 groups, 38 items) | `SiteMetricsMapper_ShouldMapSixCategoriesWith38Items_WhenRowIsFull`; manual M1 | pass |
| AC-2 Internal columns hidden (9) | `SiteMetricsMapper_ShouldHideInternalColumns_WhenMappingRow`; manual M2 | pass |
| AC-3 No duplicates with 4 existing cards | `SiteMetricsMapper_ShouldNotRepeatColumnsFromExistingCards`; manual M1 | pass |
| AC-4 Display rules, no drill-in | `formatMetricValue.test.ts`, `MetricCategoryCard.test.tsx`; manual M3 | pass |
| AC-5 Sentinel dates | `SiteMetricsMapper_ShouldMapSentinelDatesToNull`; manual M4 | pass |
| AC-6 Disabled and empty rows | mapper tests, `MetricsTab.test.tsx`; manual M5 | pass |
| AC-7 Not found | `GetSiteMetrics_ShouldReturn200WithEmptyGroups_WhenNoRow`; manual M6 | pass |
| AC-8 No schema change | 17-file diff review; `SiteMetricsRepository_ShouldNotIssueWriteStatements` | pass |
| AC-9 Tests exist | 36 new cases (Api 20, Data 3, Web 13) | pass |
| AC-10 Changed-code coverage and warnings | local: 94.1% .NET, 95.2% Web, 17 vs 17 warnings | pass (local) |
| Off-machine: PR build | ADO run 29561 (synthetic id), `refs/pull/9220/merge`, 7 min 48 s; not a required check (no branch policies) | ok |
| Off-machine: QualityCheck 445 on the branch | not queued; last known build 29300: 488 passed, 87.72% line, gate failed | **gap** |
| Off-machine: changed-code coverage on the PR | not attached | **gap** |

- **Automated checks:** PR build run 29561 (green); local xUnit and Vitest runs in [../artifact3-workflow/verification.md](../artifact3-workflow/verification.md). Details and gaps: [./verification.md](./verification.md).
- **Manual QA:** local manual steps M1-M6 on `http://localhost:5000` with `LIC-40417`, `LIC-40533`, `LIC-40998` in [../artifact3-workflow/verification.md](../artifact3-workflow/verification.md). No Dev smoke yet.

---

### Review notes

- **Summary:** The diff matches the spec exactly (17 files, six cards, 38 fields, internal columns hidden, scope clean) and has convincing local proof. It is not merge-ready only because off-machine evidence is missing. No product defect found. Full review: [./review.md](./review.md).
- **Blockers:** B1 QualityCheck 445 not run on the branch. B2 changed-code coverage (94.1%) not attached to the PR.
- **Missing evidence / gaps:** no Dev smoke before merge; no test for the SPA receiving a response without the new groups field (App deployed before Api). Non-blocking: later-added columns not auto-shown; fixture parity checks counts only; `npm run test:ci` exit 1 pre-existing (same 2 files).
- **Final decision:** **Hold.** Lift when 445 is queued on the branch with its coverage and failure mode recorded against the 87.72% baseline, and the changed-code coverage report is attached to the PR. Then Merge is a human decision.

---

### Release notes

Full text: [./release.md](./release.md).

- **Release notes:** Support engineers see six new read-only category cards on the license Metrics tab (38 fields); Yes/No with muted `No`; `-` for empty and sentinel dates; no database, CDK or config change. Ships as App 472 then Api 440, Dev first, then Prod.
- **Risk control:** Rollback = redeploy the previous known-good commit via App 472 then Api 440, Dev first then Prod (about 11.5 min per environment from the 74 s and 616 s medians), then a revert PR; four explicit triggers (internal column visible, tab broken or non-200, 5xx/latency above pre-release level for 30 min, wrong values).
- **Observability / flag decision:** No feature flag (read-only, additive, cheap redeploy). No new metrics; use existing Coralogix / OpenTelemetry for the Portal and watch the `site-metrics` endpoint for 30 minutes per environment; human Dev smoke before Prod.
- **Known risks & limitations:** later-added `SiteMetrics` columns not shown until categorised; fixture parity by counts only; `npm run test:ci` exits 1 on `main` (pre-existing); sentinel list covers two values and prod data was not inspected; QualityCheck baseline red (87.72% < 90%); App-before-Api window untested automatically.
