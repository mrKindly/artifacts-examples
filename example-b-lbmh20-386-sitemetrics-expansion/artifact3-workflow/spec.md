# Specification

Story: [LBMH20-386](https://eci-solutions.atlassian.net/browse/LBMH20-386) — see [./story.md](./story.md). Schema facts come from the read-only discovery in [../artifact2-mcp-subagents/mcp/transcript.md](../artifact2-mcp-subagents/mcp/transcript.md). Locked through [./spec-chat-log.md](./spec-chat-log.md); reviewer verdict PASS on iteration 2.

## Business Goal

Support engineers answer routine customer questions on the license record ("does this site use EDI, and when did it last send?") by opening the Metrics tab. Today the tab shows only a small part of the metrics the platform already collects, so they leave the page for the fleet dashboard or an export. That costs time per question and produces answers from a different view than the one they are looking at.

Proposed solution: on the license record detail Metrics tab, show the remaining categorised SiteMetrics values as read-only name/value cards, one card per category, with consistent rules for booleans and empty dates. EDI values are included as one of the categories. No data is changed and nothing new is collected.

## Current Behavior

- **Web (license detail, Metrics tab).** Four cards: Snapshot (snapshot date, active users), Features (a hard-coded list derived from the license record, not from SiteMetrics columns), Remote Clients (client count) and Remote Client Versions (version list). Four SiteMetrics columns are used in total. Nothing shows EDI enablement or last-used dates.
- **API.** The existing site SiteMetrics endpoint for a license returns the values for those four cards only. When a license has no SiteMetrics row the endpoint responds successfully and the tab shows its existing empty state. An unknown license returns not-found.
- **Data.** `SpruceMetrics.dbo.SiteMetrics` has 51 columns: 9 internal identity/ingest columns, 4 columns already shown, and 38 other columns. A metadata table (`dbo.SiteMetricsCategory`) already maps 42 columns (the 38 plus the 4 shown) to 6 categories with a display name and sort order; the fleet dashboard already reads it. The 9 internal columns have no category row.
- **Other tabs.** A licensing-side `ediNames` string on another tab comes from the licensing database. It is a different source and is not changed.
- **Data quality.** Date columns can hold `NULL`, or the sentinel values `1900-01-01` and `0001-01-01`, when a feature has never been used.

## Expected Behavior

On the Metrics tab, below the four existing cards (unchanged), six new cards appear in this order. Each card lists its fields as plain name/value rows in the order given. Total new fields: **38**.

| # | Card (category) | Fields (display label, type) | Count |
|---|-----------------|------------------------------|-------|
| 1 | EDI | EDI Enabled (Yes/No), EDI Last Used (date), EDI Purchase Orders Enabled (Yes/No), EDI Purchase Orders Last Used (date), EDI Invoices Enabled (Yes/No), EDI Invoices Last Used (date), EDI ASN (856) Enabled (Yes/No), EDI ASN (856) Last Used (date), EDI Catalog Enabled (Yes/No), EDI Catalog Last Used (date) | 10 |
| 2 | Integrations | E-commerce Enabled, E-commerce Last Sync (date), Payments Enabled, Payments Last Used (date), Shipping Integration Enabled, Shipping Last Used (date), Accounting Export Enabled | 7 |
| 3 | Communications | Email Enabled, Email Last Sent (date), Text Messaging Enabled, Text Last Sent (date), Fax Enabled | 5 |
| 4 | Mobile & Remote | Mobile App Enabled, Mobile Last Login (date), Handheld Enabled, Handheld Last Sync (date), Driver App Enabled, Driver App Last Used (date) | 6 |
| 5 | Reporting | Reporting Enabled, Reporting Last Run (date), Dashboards Enabled, Dashboards Last Viewed (date), Export to Excel Enabled | 5 |
| 6 | Environment | SQL Server Version (text), OS Version (text), Database Size (MB) (number), Last Backup (date), Last Upgrade (date) | 5 |

Labels and order come from the category metadata (`DisplayName`, `SortOrder`), not from column-name guessing. Type split: 18 Yes/No, 17 dates, 2 text, 1 number.

**Inclusion rule.** A field is shown if and only if it is categorised in the metadata, and it is not already shown on one of the four existing cards, and it is not on the internal exclusion list.

**Excluded internal columns (9), never shown in the UI, the response payload or mock data:** `SiteMetricsId`, `SiteId`, `IngestBatchId`, `IngestedAtUtc`, `SourceHost`, `SourceRowHash`, `CollectorVersion`, `RowVersion`, `IsDeleted`.

**De-dupe (4).** `SnapshotDate`, `ActiveUserCount`, `RemoteClientCount`, `RemoteClientVersionList` appear only on their existing cards, not again in Environment or Mobile & Remote. The four existing cards keep their current look and content.

**Value display rules** (one definition, applied to every new field):

- Yes/No fields: show `Yes` or `No`. `No` is visually muted (secondary text colour from the design system). `Yes` is normal text.
- Date fields: show the date in the same format the Snapshot card uses. A `NULL`, or a `1900-01-01` or `0001-01-01` sentinel, shows `-`.
- Text fields: show the text; `NULL` or empty shows `-`.
- Number fields: show the number with thousands separators; `NULL` shows `-`; `0` shows `0`.
- A `NULL` in a Yes/No field shows `-` (not `No`).
- All values are plain text. No links, buttons or navigation on these cards.

### User Flow

1. Support engineer opens a license record and selects the Metrics tab.
2. The four existing cards show as today.
3. Below them, six category cards show name/value rows. The engineer reads, for example, EDI Enabled `Yes` and EDI Last Used `2026-07-11` without leaving the page.
4. Features that a site has turned off show `No` in muted text, with `-` for the date.

### Edge Cases

- **Feature disabled.** `No` (muted) and last-used date `-`; the card still renders.
- **Row exists but all category columns are `NULL`.** All six cards render with every value `-`.
- **Sentinel dates.** `1900-01-01` and `0001-01-01` show `-`, never a year 1900 or 0001.
- **Partial row.** Each value is judged on its own; one empty field does not hide a card or a row.
- **No SiteMetrics row for the license.** The endpoint still succeeds, the new cards are not shown, and the tab shows its existing empty state.
- **Category column added to the table later.** Shown only once it has a category row; without one it is not shown (listed as a known limitation).
- **A category row that points at an internal column.** The column stays hidden (exclusion list wins).
- **Long text** (version strings) wraps within the card and does not break the layout.
- **Mock data (local mocks and tests).** Mock fixtures expose the same 6 cards and 38 fields as the live contract.

### Error Handling

- Unknown license: existing not-found behaviour, unchanged.
- No SiteMetrics row: success with empty groups (see above); not an error.
- Metrics source unavailable: existing handling of the Metrics endpoint applies; no new error states, messages or retries are introduced.
- Unauthorised user: existing Okta authentication and authorisation, unchanged.

## Out Of Scope

- Any database schema change, migration, new table, view or stored procedure.
- CDK, infrastructure or pipeline changes.
- Drill-in or links from the license Metrics tab to the fleet dashboard.
- Editing SiteMetrics values or adding filters, sorting, export or search on the tab.
- Replacing or restyling the four existing cards.
- Showing the 9 internal identity/ingest columns.
- Replacing the licensing `ediNames` string on the other tab.
- Feature flag or per-user configuration of which categories appear.

## Technical Scope

- **Data layer:** read the category metadata and the license's SiteMetrics row using the existing read access; no new database objects.
- **API:** extend the existing site SiteMetrics response with category groups (card name, ordered items each with a stable key, display label, value type and value); apply the inclusion and exclusion rules and normalise sentinel dates to empty; return empty groups when no row exists.
- **Web:** render the groups as category cards on the Metrics tab using one shared value formatter; keep the four existing cards.
- **Mock data and tests:** update mock fixtures to the same contract; add unit tests for mapping and repository behaviour, endpoint behaviour for found/empty/not-found, and UI tests for card rendering and display rules.

## Non-functional Requirements and Constraints

- Stack: .NET 10 ASP.NET Core API, React + Vite + Fluent UI v9; follow existing project structure and the repository rules.
- **Schema-first:** column names come from the live schema discovery (51 columns, 6 categories) and are not inferred from naming patterns.
- **Database access is read-only** (SELECT only). No changes under CDK, pipeline YAML, SQL scripts or migrations.
- **Coverage on new/changed code >= 90%** (line coverage, .NET and Web measured on changed lines). The whole-solution baseline (87.72% line on QualityCheck build 29300) is not the target of this story.
- **Zero new warnings** in `dotnet build`, TypeScript compilation and lint compared with `main`.
- Display rules live in one place per layer (date normalisation in the API, formatting in the Web); no duplicated rule tables.
- Performance: no additional database round trips beyond reading the metadata and the one SiteMetrics row for the license.
- No customer data in logs added by this work.

## Acceptance Criteria

- **AC-1 Mapping.** Given a license whose SiteMetrics row has values in every column, the Metrics tab data contains exactly 6 category groups in the order EDI, Integrations, Communications, Mobile & Remote, Reporting, Environment, containing 38 items in total (10, 7, 5, 6, 5, 5), with labels and order from the category metadata. The EDI group lists EDI Enabled and EDI Last Used first.
- **AC-2 Internal columns hidden.** None of the 9 internal columns (`SiteMetricsId`, `SiteId`, `IngestBatchId`, `IngestedAtUtc`, `SourceHost`, `SourceRowHash`, `CollectorVersion`, `RowVersion`, `IsDeleted`) appears in the API response, the rendered tab or the mock fixture, even if a category row exists for one of them.
- **AC-3 No duplicates.** None of the 4 already-shown columns appears in a new card, and the four existing cards render unchanged.
- **AC-4 Display rules.** In the rendered tab, Yes/No fields show `Yes`/`No` with `No` muted and `Yes` not muted; empty text, number and Yes/No (`NULL`) values show `-`; every row is a plain name/value pair with no link or button.
- **AC-5 Sentinel dates.** A date of `1900-01-01` or `0001-01-01`, or `NULL`, shows `-`; a real date shows in the same format as the Snapshot card.
- **AC-6 Disabled and empty.** A site with features disabled shows `No` (muted) and `-` for their dates, and a site whose category columns are all `NULL` shows all six cards with every value `-`; no card is hidden and no error appears.
- **AC-7 Not found.** A license with no SiteMetrics row gets a successful response with empty groups and the tab shows its existing empty state; an unknown license still returns not-found.
- **AC-8 No schema change.** The change set contains no database script, migration, CDK or pipeline file, and the data layer issues only read statements.
- **AC-9 Tests.** Automated tests exist for: the mapping rules (AC-1 to AC-6), the endpoint (AC-7), the repository read, the card rendering and display rules, and mock-fixture parity with the contract (6 cards, 38 items).
- **AC-10 Quality bar.** Line coverage on new and changed code is at least 90% and the build adds zero new warnings (.NET build, TypeScript, lint) compared with `main`.

## Assumptions

- The category metadata table is the source of truth for labels and order, and it is already maintained for the fleet dashboard (confirmed by schema discovery; not changed by this story).
- The Snapshot card's existing date formatter can be reused for the new date fields.
- Every SiteMetrics row is looked up through the license's existing site key, as today.
- `NULL` in a Yes/No column is possible in the schema even though sample rows show only 0/1.
- Production data was not inspected; behaviour for production rows is inferred from dev rows (see risks).
- The Features card stays hard-coded and is not reconciled with the new EDI card.
- English labels only; no localisation.

## Risks

- Sentinel values other than the two listed may exist in production data; they would show as real dates. Mitigation: the sentinel list is a single constant in the API mapper.
- A new `SiteMetrics` column added later is not shown until it receives a category row. Accepted and documented in release notes.
- Internal columns could be exposed if someone adds a category row for them; mitigated by the exclusion list and AC-2.
- Mock fixtures can drift from the live contract; mitigated by the parity test in AC-9.
- Whole-solution coverage is under 90% (87.72%), so a QualityCheck run on the branch will not turn green because of this story alone.

## Open Questions (all resolved)

| # | Question | Resolution | Where |
|---|----------|------------|-------|
| 1 | Full set or EDI only? | Full categorised set; EDI is one of six cards | [./spec-chat-log.md](./spec-chat-log.md) Q1 |
| 2 | Which columns are internal? | The 9 columns listed above; every other column must have a category row to be shown | Q2 |
| 3 | What counts as an empty date? | `NULL`, `1900-01-01`, `0001-01-01` | Q3 |
| 4 | How are booleans shown? | `Yes`/`No`, `No` muted, `NULL` shows `-` | Q4 |
| 5 | Which coverage number does the NFR mean? | >= 90% on new/changed code only | Q5 |
