# Spec for FS07-3521 fulfilled orders by make
_Exported on 10/20/2026 at 17:42:19 GMT+3 from Cursor (3.25.9)_

---

**User**

/spec FS07-3521

---

**Cursor**

I'll read the ticket through the Atlassian MCP first, then the closest existing operation and the schema, so I only ask what Jira and the code do not answer.

Read FS07-3521 (issue, 2 comments, parent FS07-3442, blocked sibling FS07-3530). Read `GetSalesOrdersByDate`, the `Sales` contracts, and the table list in `db/migrations/`.

Current behavior: no operation filters by make or fulfillment, and the router has no route for it.

**Q1.** The ticket says "fulfilled" for the order, the line, and the equipment. In `FsCore` the header `Status` can say `Fulfilled` with no fulfillment records, and `Open` with fulfilled lines. Which one decides?

---

**User**

I do not know. The first prototype used the header. I will ask the PM. Mark it open.

---

**Cursor**

Marked open.

**Q2.** The watermark: compared to which date, with which operator and which timezone? The prototype used order create date.

---

**User**

Open as well. The consumer comment says they own the sync clock.

---

**Cursor**

Marked open.

**Q3.** What is one row: an order, a line, or a serial? The payload has serial and MAC, which suggests serial.

---

**User**

Probably serial. Open.

---

**Cursor**

**Q4.** "Whole word" on SQL Server: there is no regex. Should `SHARP-MX` match `Sharp`? What about `Sharp Electronics` as a search value?

---

**User**

Hyphen should separate words. Multi-word search value: undecided.

---

**Cursor**

**Q5.** Is there a limit on the result size, and is pagination wanted?

---

**User**

Nothing in Jira. Mark open.

---

**Cursor**

Six questions are unanswered by Jira or code: the five above, plus which party is the ship-to when a fulfillment record carries its own party. I wrote a first draft with each one as an explicit assumption flagged `NEEDS PM`, and stopped.

Draft: `docs/FS07-3521-fulfilled-orders-by-make/spec.md` (v1).

---

**User**

Notes from the PM walkthrough today:

- Registration intent: register a device when it **ships**. So the fulfillment activity time is the watermark. Create date is wrong.
- Partial fulfillment is fine: register what shipped. Header status is not reliable.
- One row per serial. If a serial shipped twice, send it once with the later time.
- Ship-to: the party on the fulfillment record wins; fall back to the order ship-to. Bill-to is the order's.
- Hyphen, slash, ampersand count as word breaks. A multi-word make like `Sharp Electronics` must match as a phrase.
- Size: the consumer syncs daily. Do not build paging. A hard stop with a clear error is fine.

Update the spec with these notes. Then run the reviewer.

---

**Cursor**

I'll fold the walkthrough into the spec, measure the one assumption I can check on dev data, and then start `spec-reviewer`.

Updated the spec: definitions table, AC-1 to AC-12, error codes. Measured one assumption in `FsCore_Dev`: 8,134 of 21,406 fulfillment rows have a ship-to party (38%).

`spec-reviewer` iteration 1 returned **FAIL** with four blocking findings:

1. "Fulfilled" is undefined in the goal.
2. AC-6 does not define the comparison.
3. AC-8 is missing equipment fields.
4. The one-row rule is contradicted by an edge case.

One non-blocking finding: a design hint in Technical Scope.

---

**User**

Fix all four, remove the hint.

---

**Cursor**

Fixed:

- Added the definitions table and the strict `>` rule in UTC.
- Expanded AC-8 to the full field list.
- Changed the edge case to "appears once with the later time".
- Removed the CTE sentence.
- Replaced an unmeasured "about 40%" with the measured 38% (A3).

`spec-reviewer` iteration 2 returned **PASS** with one non-blocking finding: name `TooManyRows` in AC-11.

---

**User**

Accept.

---

**Cursor**

AC-11 updated. Spec v3 saved to `docs/FS07-3521-fulfilled-orders-by-make/spec.md`. Open questions: none. The human decisions are summarised in `review-notes.md`.
