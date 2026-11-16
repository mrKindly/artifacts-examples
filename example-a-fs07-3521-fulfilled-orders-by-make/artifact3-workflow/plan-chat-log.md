# Implementation plan for FS07-3521
_Exported on 10/20/2026 at 19:15:03 GMT+3 from Cursor (3.25.9)_

---

**User**

Create the implementation plan for `docs/FS07-3521-fulfilled-orders-by-make/spec.md` (v3). Use `context-map.md` for context. Phases follow the story card. Test first. Each step leaves the repo green. No scope beyond the spec.

---

**Cursor**

I'll read the spec, the context map, `AGENTS.md` and the two rules, then the code the spec touches, and draft the plan.

Draft plan: 7 steps across the 4 phases. Notable parts:

1. Add `PageSize` and `PageToken` to the request for future-proofing.
2. Refactor the party lookup shared with `GetSalesOrdersByDate` into a common helper.
3. Implement the whole-word match as `LIKE '%' + @make + '%'` with a post-filter in C#.
4. One step: implement the operation and the proc, then test.
5. Router and service in a single PR.

---

**User**

Changes:

- **Pagination (1):** the PM said the consumer syncs daily and does not want paging. The spec says the cap fault is the only size control. Remove it.
- **Shared party helper (2):** it touches an existing operation that other consumers use. Not in the spec; contracts and procs of existing operations stay unchanged. Remove it and note it in Out Of Scope.
- **`LIKE` plus C# filter (3):** the substring `LIKE` pulls `Sharpie` rows into memory, and a C# filter would run after the 5000-row cap, so the cap would count rows that should not match. Replace it with a tested `fn_ContainsWholeWord` used inside the proc.
- **One big step (4):** split it. Each step starts with failing tSQLt or NUnit tests.
- **Single PR (5):** two repos, so two PRs. On-prem is deployed first.

---

**Cursor**

Regenerated the plan with your changes:

- Pagination removed. The cap fault stays the only size control.
- Shared party helper removed and listed in Out Of Scope.
- Whole-word match is now `fn_ContainsWholeWord`, tested and used inside the proc.
- The big step is split into steps 2 to 5 in Phase 2; each starts with failing tSQLt or NUnit tests.
- Two PRs, on-prem first.

I also added three human gates: G1 after the red tests, G2 after the SQL behavior is checked on dev data, and G3 for deploy order. Phase 4 now includes a 3-second timing check for 5000 rows.

Wrote `docs/FS07-3521-fulfilled-orders-by-make/plan.md`.

---

**User**

Approved for phase 2 step 1 only. Scaffold, then stop.

---

**Cursor**

Understood. I'll do phase 2 step 1 (scaffold and red tests) and stop at G1. Later steps wait for your approval after G1 and G2.
