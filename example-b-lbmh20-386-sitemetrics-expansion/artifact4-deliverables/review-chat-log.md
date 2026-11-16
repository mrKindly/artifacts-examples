# Review PR 9220 for LBMH20-386
_Exported on 7/15/2026 at 13:52:06 GMT+3 from Cursor (3.11.6)_

---

**User**

Review PR 9220 for LBMH20-386 before I ask for human approval. Tests are green locally. Can we merge?

---

**Cursor**

I'll review against the spec, the plan, the verification evidence and the release notes first, and read the diff last. Read-only: I will not edit anything.

Inputs loaded: `spec.md` (AC-1 to AC-10), `plan.md` (5 phases, gates G1-G5), `context-map.md`, local `verification.md` (36 new cases, manual M1-M6), `release.md`, then the diff `origin/main...HEAD` (17 files).

Findings so far:

- **Spec compliance:** all 10 ACs have local evidence. AC-2 is proven with all 9 internal columns, including a planted category row.
- **Plan adherence:** the diff stays within the files named in the plan. No file under `CDK/`, no pipeline YAML, no SQL script or migration (AC-8).
- **Hidden behaviour change:** the four existing cards are untouched; the response change is additive.
- **Test quality:** tests assert visible text, roles and payload shape, not just "no exception". The sentinel dates are a theory test.
- **Release readiness:** the rollback in `release.md` redeploys the previous commit through App 472 then Api 440, no data to undo. A flag is not claimed and none exists.

Evidence off the machine:

- PR build run 29561 is green (7 min 48 s), but it was queued by hand and no policy requires it.
- QualityCheck 445 has **not** been run on this branch. Last known run is build 29300 (488 passed, 87.72% line, Coverlet gate failed).
- The 94.1% changed-code coverage is local only; nothing is attached to the PR.

---

**User**

Is the missing 445 run a real problem? It is red on main anyway.

---

**Cursor**

Yes, as missing evidence, not as a defect. It is red on baseline, so a red result is expected. What matters is the delta: whole-solution line coverage should not drop below 87.72% and the only failure should still be the Coverlet gate. Without a run on the branch nobody can say that. Same for the coverage figure: the AC-10 bar is on changed code, and the only copy of that number is on your machine.

Classification:

- Blocker B1: 445 not run on the branch.
- Blocker B2: changed-code coverage not attached to the PR.
- Non-blocking N1 to N4: later-added columns not shown without a category row; fixture parity checks counts only; `npm run test:ci` exit 1 is pre-existing (same 2 files); no test for a response lacking the groups field during the App-before-Api window.
- Info I1 to I5: no schema/CDK/pipeline files; PR build manual and not required; sentinel list covers two values; both pre-PR checks PASS; PR open since 2026-07-14 with one reviewer.

**Decision: HOLD.** No product defect. Lift the hold when 445 is queued on the branch and its whole-solution coverage and failure mode are recorded against 87.72%, and the changed-code coverage report is attached. Merge and approval remain human actions; I am not approving anything.

Wrote `docs/LBMH20-386/review.md`.

---

**User**

Agreed. I'll queue 445 and attach the report. Next: release strategy.
