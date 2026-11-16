# Review notes — LBMH20-386

Human review of the spec and the plan. The five spec choices were answered in the interactive `/spec` session ([./spec-chat-log.md](./spec-chat-log.md)); the spec edits that followed the `spec-reviewer` FAIL were kept. The plan session ([./plan-chat-log.md](./plan-chat-log.md)) produced three flagged proposals; the human accepted one, changed one and rejected one.

## Outcome

The spec at [./spec.md](./spec.md) is the accepted spec (`spec-reviewer` PASS on iteration 2, no blocking or non-blocking findings). The plan at [./plan.md](./plan.md) is the accepted plan with five phases and gates G1 to G5. Local verification of AC-1 to AC-10 passed ([./verification.md](./verification.md)). The merge decision is in [../artifact4-deliverables/review.md](../artifact4-deliverables/review.md).

## Decisions

| Decision | Accepted / Changed / Rejected | Why |
|----------|-------------------------------|-----|
| Scope: all 38 categorised fields in 6 cards, EDI as one card | Accepted | The requester asked for EDI first but also for "the other category fields". The data is already categorised for the fleet dashboard, so the marginal cost is low and a second story would repeat the discovery. |
| Scope: EDI only (10 fields), rest in a follow-up | Rejected | Produces the "partial EDI card that still disappoints the requester" the story warns about, and splits one mapping into two PRs. |
| Internal columns: hide the 8 ingest-related columns | Changed | The discovery found 9 uncategorised columns. `SiteId` is the join key, not a metric, so it was added to the exclusion list (9 total). |
| Internal columns: show everything uncategorised | Rejected | Would expose ingest and identity columns the story lists as out of scope. |
| Rule: a column is shown only if it has a category row and is not on the exclusion or already-shown lists | Accepted | New columns appear only when someone categorises them, which is the existing fleet-dashboard convention. Known limitation recorded in release notes. |
| Empty dates: `NULL`, `1900-01-01`, `0001-01-01` all show `-` | Accepted | The value sample from the dev database showed both sentinels next to real `NULL`s; the requester asked that empty dates should not look like 1900. |
| Empty dates: `NULL` only, or `NULL` and `1900-01-01` | Rejected | Would leave year 1 or 1900 on screen for rows holding the other sentinel. |
| Booleans: `Yes`/`No`, `No` muted, `NULL` shows `-` | Accepted | Matches "No should not shout" in the Jira comment; `NULL` is unknown, not "No". |
| Booleans: icons instead of text | Rejected | Needs new design work and an accessibility label per icon; not in the story. |
| Coverage NFR: line coverage >= 90% on new/changed code only | Changed | Ticket text said "90% coverage". Whole-solution coverage is 87.72% on QualityCheck build 29300, so the literal reading would block this story for a reason it cannot fix. Reworded after `spec-reviewer` blocking finding 3. |
| Spec wording of "muted" as secondary text colour from the design system; Yes not muted | Changed | `spec-reviewer` iteration 1 found "No is muted" untestable. Wording now makes AC-4 verifiable by a test and a visual check. |
| Added edge case "category row pointing at an internal column" and fixture parity in AC-9 | Accepted | Non-blocking reviewer findings 4 and 5; both are cheap and close real gaps (exposure of internal columns, fixture drift). |
| Plan: phase order mapper/API first, UI second, TDD in every step | Accepted | Lets the payload be reviewed (gate G2) before UI work; each phase leaves a working state. |
| Plan: internal-column exclusion applied in the repository SQL | Changed | Moved to a catalog plus the mapper so an xUnit theory covers all 9 columns, including a planted category row. A SQL-only filter would not be covered by the .NET test suite. |
| Plan: "View in fleet dashboard" link on each category card | Rejected | Drill-in is explicitly out of scope in the story and spec (AC-4: plain name/value pairs). Scope creep; recorded under Out Of Scope. |
| Plan: add gate G2 after the response shape is defined | Accepted (human-added) | Cheapest point to catch a leaked internal column or a wrong contract before UI work. |
| Plan: no feature flag | Accepted | Read-only display, no write path, additive response; a flag would add a second code path for no rollback benefit beyond redeploy. Revisited in [../artifact4-deliverables/release.md](../artifact4-deliverables/release.md). |
