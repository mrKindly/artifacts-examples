# Release notes — LBMH20-386

> Condensed illustrative reconstruction (synthetic ids and numbers). Decision: **Hold** until the two blockers in [./review.md](./review.md) are cleared. Source session: [./release-chat-log.md](./release-chat-log.md). Pipeline medians (App 472 about 74 s, Api 440 about 616 s) come from [portal-SDLC.json](../artifact5-ai-use-cases/portal-SDLC.json).

PR: [9220](https://dev.azure.com/ECI-LBMH/LBMH-Spruce/_git/LBMH.Services.Portal/pullrequest/9220) (demo reference link). Branch `feature/LBMH20-386-sitemetrics-expansion`, head `a41c7e9` (synthetic).

## What changes

**Users (support engineers).** On a license record, the Metrics tab keeps its four cards and gains six cards below them: EDI (10 fields), Integrations (7), Communications (5), Mobile & Remote (6), Reporting (5) and Environment (5), 38 fields in all. Each field is a plain name/value row. Yes/No fields read `Yes` or `No` (`No` in muted grey); empty values and empty dates (`NULL`, `1900-01-01`, `0001-01-01`) read `-`. A license with no metrics row shows the same empty state as today. There are no links, filters or editing.

**Operators.** No database change, no migration, no new configuration, no new infrastructure (Infra 441 is not part of this release). The API response gains an additive list of category groups; the SPA reads it. Both ship: **App 472 and Api 440**, in that order per environment, **Dev first, then Prod**, started by a human through the `deploy` skill.

## Risk control

**Triggers to roll back** (any one):

1. Any of the 9 internal column names (`SiteMetricsId`, `SiteId`, `IngestBatchId`, `IngestedAtUtc`, `SourceHost`, `SourceRowHash`, `CollectorVersion`, `RowVersion`, `IsDeleted`) appears in the `site-metrics` response or on the tab.
2. The Metrics tab fails to render (browser error) or the endpoint returns non-200 for a license that rendered before the release.
3. 5xx rate or p95 latency on the `site-metrics` endpoint stays above its pre-release level for 30 minutes after a deploy (compare with the 30 minutes before; no fixed number is invented here).
4. Support reports wrong values: a 1900/0001 date, or an unknown Yes/No shown as `No`.

**Rollback.** There is no data to undo. Redeploy the previous known-good commit (the `main` commit recorded at merge time) with the same pipelines and order as the release: **App 472, then Api 440, Dev first, then Prod**. Expected duration per environment is about 11.5 minutes (74 s + 616 s medians), about 23 minutes for both. Then open a revert PR so `main` matches what is deployed (human merge). If a trigger is seen only in Prod, a human may start with Prod to limit user impact; this is the only allowed deviation from Dev-first.

**Roll forward.** For a small defect (a label, a muted style), fix on a branch, repeat Phase 4 evidence, and redeploy App 472 then Api 440; prefer this to rollback when no trigger 1 or 2 applies.

**Deploy-order window.** App 472 runs before Api 440, so for a few minutes the new SPA talks to the old API, which returns no groups. The tab is expected to show the four existing cards only. This is **not covered by an automated test** (review finding N4); check it manually on Dev by loading the tab between the two runs.

**Not allowed:** deploying from the branch, deploying Infra 441 for this change, any DB action.

## Observability / flag decision

- **Feature flag: no.** The change is read-only display on one tab, additive in the API, and has no write path. A flag would add a second code path to test and would not shorten rollback beyond the roughly 11.5 minutes of a redeploy.
- **Metrics / logging: nothing new.** Use the existing Coralogix / OpenTelemetry telemetry for `LBMH.Services.Portal`. Watch for 30 minutes after each environment: HTTP status mix and p95 latency of the `site-metrics` endpoint, and any exception raised from the site metrics mapper. No customer data is added to logs by this change.
- **Rollout:** Dev first, smoke, then Prod. Smoke after each environment is a human check with the three Dev licenses (`LIC-40417` full data, `LIC-40533` disabled features, `LIC-40998` no row; manual steps M1, M4, M5, M6 in [../artifact3-workflow/verification.md](../artifact3-workflow/verification.md)); on Prod open one known license read-only and confirm six cards and no internal names.
- **Pre-conditions for starting the release:** blockers B1 and B2 cleared and a human merge decision. Not started while the decision is Hold.

## Known risks and limitations

- A `SiteMetrics` column added later is not shown until it receives a category row in the metadata table; this is intentional (the fleet dashboard behaves the same) and not an automatic "show everything".
- Mock fixture parity is checked by counts (6 groups, 38 items) only; label drift would not fail that test.
- `npm run test:ci` exits 1 on `main` and on this branch (same 2 failing files); Web proof is the scoped run (13 tests) plus an unchanged failure set.
- Sentinel handling covers `1900-01-01` and `0001-01-01`; other sentinels in production would show as dates. Production data was not inspected.
- QualityCheck 445 is red on baseline (87.72% line, gate 90%); this release does not fix that.
- App-before-Api window described above is untested automatically.
- Out of scope, from the plan: drill-in to the fleet dashboard, schema/CDK/pipeline changes, editing, filters, export, per-user configuration, restyling the four existing cards.
