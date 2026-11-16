# Spec for LBMH20-386 SiteMetrics expansion
_Exported on 7/13/2026 at 16:24:51 GMT+3 from Cursor (3.11.6)_

---

**User**

/spec LBMH20-386

---

**Cursor**

I'll generate a spec for LBMH20-386. First I'm loading the spec template, reading the Jira issue and its comment (read-only), and reading the Metrics tab code and the existing site metrics endpoint.

Read the issue (no subtasks, no links) and the requester's comment of 2026-07-10. Read the Metrics tab (four cards), the endpoint and the mapper. No existing spec for this story.

The ticket says "show additional SiteMetrics fields" with EDI as the priority. Today the tab uses 4 `SiteMetrics` columns. Schema must be discovered before any scope can be locked, so I'm using the `mssql-readonly` MCP (dev database) rather than guessing names.

Calls: `list_columns` on `dbo.SiteMetrics` (51 columns), `describe_table` on `dbo.SiteMetricsCategory`, `select_top` on the category table (42 rows, 6 categories), a join showing 9 columns with no category, and a 5-row value sample showing sentinel dates `1900-01-01` and `0001-01-01`.

What this means: 42 categorised columns minus the 4 already shown leaves 38 candidates in 6 categories; 9 columns have no category. The licensing `ediNames` string is a different source and does not answer the requester's EDI question.

Five choices stay open.

> **Scope. Show only the EDI fields now, or all categorised fields?**
> - EDI only (10 fields), others in a follow-up
> - All 38 categorised fields in 6 cards ✔

> **Internal columns. Which columns must stay hidden? Proposed: `SiteMetricsId`, `IngestBatchId`, `IngestedAtUtc`, `SourceHost`, `SourceRowHash`, `CollectorVersion`, `RowVersion`, `IsDeleted` (the ingest-related uncategorised ones; the discovery found 9 uncategorised columns, the 9th is `SiteId`, the join key).**
> - Hide exactly those 8
> - Hide those 8 plus `SiteId` (the join key, also uncategorised) ✔
> - Show everything uncategorised too

> **Empty dates. What should a date column show when it is `NULL`, `1900-01-01` or `0001-01-01`?**
> - `-` for `NULL` only
> - `-` for `NULL` and `1900-01-01`
> - `-` for `NULL`, `1900-01-01` and `0001-01-01` ✔

> **Booleans. How should Yes/No be shown?**
> - `Yes`/`No`, both normal text
> - `Yes`/`No`, `No` muted ✔
> - Icons instead of text

> **Coverage NFR. The ticket says "90% coverage". Which number?**
> - Whole solution (QualityCheck baseline is 87.72% line, build 29300)
> - Line coverage on new/changed code only ✔

---

**User**

Two notes: `SiteId` is the key we join on, nobody needs to read it, so it stays hidden. And a Yes/No that is `NULL` should not look like a "No".

---

**Cursor**

Recorded. A `NULL` Yes/No shows `-`. Using the category metadata for labels and order. Out of scope keeps: no schema/CDK/pipeline change, no drill-in to the fleet dashboard, no editing, existing four cards unchanged. Writing the spec now, then running `spec-reviewer`.

Wrote `docs/LBMH20-386/spec.md` (draft 1).

`spec-reviewer` iteration 1 returned:

```
Verdict: FAIL

Blocking
1. AC-4: "No is muted" is not testable. Define muted (secondary text colour from the design system) and say Yes is not muted.
2. Expected Behavior does not say what a NULL Yes/No shows. The chat answer ("not a No") is missing from the spec.
3. Non-functional Requirements say "90% coverage" without scope. Say line coverage, new/changed code only, and that the 87.72% baseline is not the target.

Non-blocking
4. Edge case missing: a category row that points at an internal column.
5. AC-9 does not require mock-fixture parity with the contract (6 cards, 38 items), although the story lists mock fixtures.
```

All five are valid. Updated the spec:

- AC-4 now names secondary text colour and states Yes is not muted.
- Expected Behavior and AC-4 say a `NULL` Yes/No shows `-`.
- The NFR now reads "line coverage on new/changed code, >= 90%, baseline 87.72% not the target".
- Added the edge case "category row that points at an internal column" and AC-2 wording "even if a category row exists".
- AC-9 now includes fixture parity.

`spec-reviewer` iteration 2 returned:

```
Verdict: PASS

Blocking: none
Non-blocking: none
Checks: every AC independently testable; counts consistent (51 = 9 + 4 + 38; 38 = 10+7+5+6+5+5; 18 Yes/No + 17 date + 2 text + 1 number);
no code or plan content; out of scope is explicit; assumptions separated from constraints.
```

Spec accepted by the reviewer. Your decisions on the draft and the review changes are summarised in `docs/LBMH20-386/review-notes.md`. Next step: the plan.
