# Review PR 2481 and PR 377 against spec
_Exported on 10/30/2026 at 14:27:51 GMT+3 from Cursor (3.26.2)_

---

**User**

Review PR 2481 and PR 377 against `spec.md`. Use the `spec-reviewer` approach for scope, and the checklist from Artifact 4: summary, blockers, missing evidence, final decision. Read only; do not change files.

---

**Cursor**

I'll read both diffs, the spec and `verification.md`, then check each AC against a test in the diff and the scope against the plan's file list. Read-only.

Read both diffs (`git diff main...HEAD` in each repo), the spec, and `verification.md`.

```
Coverage of ACs: AC-1..AC-14 each map to at least one new test in the diff.
Scope: files outside the plan list: none. Existing contracts/procs/Web.config: unchanged.
Findings:
 1. Info        tSQLt run 5120 attempt 1 failed in tests not touched by this PR.
 2. Non-blocking service log writes the Make value (rule says operation, rows, elapsed only).
 3. Non-blocking TooManyRows -> HTTP 422 (others are 400).
 4. Non-blocking no production-volume timing.
Blockers: none.
Missing evidence: Staging consumer smoke.
Suggested decision: Merge, with the route off in Production until the smoke.
```

---

**User**

On finding 2, is the make value personal data?

---

**Cursor**

No. It is a product name from `EquipmentMake`. The rule exists to keep addresses out of logs. Either leave it and record it, or drop it from the log line.

---

**User**

Leave it, record it under known risks. On finding 1, is it our change?

---

**Cursor**

No. The two failing tests are `usp_GetSalesOrdersByDate` tests whose fixtures lack the `tsqlt_` prefix; they collided with another run. The new tests passed in both attempts.

---

**User**

Accept. Write the review notes. The final decision is mine: Merge.

---

**Cursor**

Wrote `review.md`. Decision recorded as **Merge**, by the developer.
