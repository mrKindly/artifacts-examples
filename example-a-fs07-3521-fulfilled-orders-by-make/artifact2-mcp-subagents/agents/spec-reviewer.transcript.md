# Run spec-reviewer on FS07-3521 spec
_Exported on 10/20/2026 at 17:36:44 GMT+3 from Cursor (3.25.9)_

---

**User**

Run the `spec-reviewer` subagent on `docs/FS07-3521-fulfilled-orders-by-make/spec.md`. This is the draft after the PM walkthrough.

---

**Cursor**

I'll start `spec-reviewer` (`.cursor/agents/spec-reviewer.md`) in the foreground with a fresh context. Input: the spec path, the template path, and the Jira context summary from the Atlassian read earlier in this session.

`spec-reviewer` returned:

```
Verdict: FAIL
Iteration: 1

Blocking findings
1. 4 · Business Goal / AC-4 · "fulfilled" is used for the order and for the equipment line. The spec never says whether one fulfilled line is enough or the header must be Fulfilled.
2. 3 · AC-6 · "returns only new activity" has no definition of the comparison (> or >=) or of the timezone of the watermark.
3. 2 · AC list · Story AC8 (equipment identity: serial, MAC, make/model fields) has no matching spec AC. The spec lists only serial and MAC.
4. 7 · Edge Cases vs Assumptions · Edge case says an order fulfilled twice appears twice; Assumptions say one row per serial.

Non-blocking findings
1. 6 · Technical Scope · "use a recursive CTE for party fallback" is a design decision. Move it out.

Unverified statements
- "SalesOrderLineFulfillment.ShipToPartyId is populated for about 40% of rows" · no query result or file in the repo supports the number.

Refused
- none
```

Four blocking findings need a decision from you before the next iteration. The unverified 40% claim can be replaced with a measured number from the dev database.

---

**User**

Spec updated: findings 1-4 fixed (definitions section, AC-6 wording, AC-8 field list, one row per serial, latest fulfillment wins), CTE sentence removed, and the 40% claim replaced by the measured number from the dev DB (assumption A3). Run iteration 2.

---

**Cursor**

Starting `spec-reviewer` again with a fresh context and the updated spec.

`spec-reviewer` returned:

```
Verdict: PASS
Iteration: 2

Blocking findings
- none

Non-blocking findings
1. 3 · AC-11 · The cap of 5000 rows is stated, but the fault code name appears only in Error Handling. Add it to the AC for traceability.

Unverified statements
- none

Refused
- none
```

PASS. One non-blocking finding: name the fault code in AC-11.

---

**User**

Accept it.

---

**Cursor**

AC-11 now names `TooManyRows`. No third iteration needed.

The subagent used only read-only file reads: `docs/FS07-3521-fulfilled-orders-by-make/spec.md`, the template, and `db/migrations/0147_deprecate_legacy_equipment.sql`. No other tools.
