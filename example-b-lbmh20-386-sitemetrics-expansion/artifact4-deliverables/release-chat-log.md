# Release strategy for LBMH20-386
_Exported on 7/15/2026 at 15:31:44 GMT+3 from Cursor (3.11.6)_

---

**User**

/release-strategy for LBMH20-386, PR 9220

---

**Cursor**

I'll prepare the release strategy from the spec, the plan, the review and the change itself, read-only. First, what is being released: 17 files, the Api response gains category groups, the SPA renders six new cards. Both App and Api change; Infra does not (no CDK file in the diff).

Facts checked:

- Deploys are manual through the `deploy` skill: App 472, Api 440, Dev then Prod; merge to `main` deploys nothing.
- Pipeline medians in the measurements: App 472 about 74 s, Api 440 about 616 s, both with 0% failures in the window.
- No feature flag exists in the repo for the Metrics tab; the change has no write path.
- Observability: existing Coralogix / OpenTelemetry for the Portal; nothing in the diff adds telemetry.

Questions I need answered by evidence, not guesses:

1. **Flag?** No. Read-only, additive, and redeploy rollback is about 11.5 minutes per environment (74 s + 616 s); a flag would add a second path to test.
2. **Rollback?** No data change, so redeploy the previous known-good commit through App 472 then Api 440, Dev first then Prod; then a revert PR to keep `main` aligned.
3. **What can go wrong that tests do not cover?** The deploy order. App 472 runs before Api 440, so the new SPA briefly talks to the old API with no groups field. Only empty groups are tested, not an absent field. Recorded as a known risk and a manual Dev check.
4. **What to watch?** Status mix and p95 latency of the `site-metrics` endpoint and mapper exceptions for 30 minutes per environment; no new metric.

---

**User**

Add explicit rollback triggers. And keep Prod behind a Dev smoke with the three licenses.

---

**Cursor**

Added four triggers:

1. An internal column name is visible.
2. The tab fails or returns non-200.
3. 5xx or latency above the pre-release level for 30 minutes.
4. Wrong values, such as 1900 or an unknown Yes/No shown as No.

Added the Dev smoke with `LIC-40417`, `LIC-40533`, `LIC-40998`, and the pre-condition that the release does not start while the decision is Hold (blockers B1 and B2).

Wrote `docs/LBMH20-386/release.md`.

---

**User**

Good. Decision stays Hold until 445 and the coverage report are in.
