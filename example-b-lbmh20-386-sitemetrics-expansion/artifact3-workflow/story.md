# LBMH20-386 — Portal: expand SiteMetrics on the license record detail Metrics tab

> Original work item as read from Jira on 2026-07-13 (condensed, synthetic reconstruction modeled on the real story shape; the comment is illustrative).

[LBMH20-386](https://eci-solutions.atlassian.net/browse/LBMH20-386)

| Field | Value |
|-------|--------|
| Key | LBMH20-386 |
| Project | LBMH20 |
| Type | Story |
| Status | Ready for Development |
| Summary | Portal: expand SiteMetrics on the license record detail Metrics tab |
| Reporter | Support Engineering lead |
| Repos / surfaces | LBMH.Services.Portal: license detail Metrics tab, existing SiteMetrics API, live `SpruceMetrics` schema |

## User story

**As a** support engineer on the license detail page,  
**I want to** see the SiteMetrics values I already use elsewhere (especially EDI configuration) on the Metrics tab,  
**So that** I can answer routine customer questions without leaving the page for a fleet export or dashboard.

## Problem / context

The Metrics tab only shows a subset of SiteMetrics (snapshot, hard-coded features, remote-client cards). EDI enablement and last-used dates and many other categorized columns already exist in `SiteMetrics` and on the fleet dashboard, but are missing on the license page. A licensing-side `ediNames` string on another tab is a **different source** and does not replace SiteMetrics EDI flags.

## Acceptance criteria

| ID | Criterion | Verifiable by |
|----|-----------|----------------|
| AC1 | Metrics section shows additional SiteMetrics fields already in DB (including EDI values expected by the requester) | Manual + mapping tests |
| AC2 | Values render as simple name/value pairs (no drill-in on this page) | UI |
| AC3 | No database schema changes | Review |
| AC4 | Unit tests for mapping; UI/smoke coverage for expanded grid / styling rules | `dotnet test` / frontend CI |
| AC5 | Team NFRs (coverage on new/changed code >= 90%, zero new warnings) as stated on the ticket | Pipeline / local verify |

## Out of scope

- Schema/migrations, CDK/pipeline changes
- Drill-in from license Metrics into the fleet dashboard
- Editing SiteMetrics; replacing the existing four cards
- Showing internal identity/ingest columns

## Suggested phases

1. Spec + schema discovery
2. Mapping / API payload
3. Metrics UI cards
4. Pre-PR verification
5. PR, approve, deploy

## Comments

**Support Engineering lead, 2026-07-10:**
"We keep opening the fleet dashboard just to check whether a customer has EDI turned on and when they last used it. EDI is the thing I care about most, but if the other category fields are already in SiteMetrics, show them on the license page too. Please do not show the ingest/internal stuff, and 'No' should not shout at us. Empty dates should not look like 1900."

## Why the ticket alone is not enough

"Show additional SiteMetrics fields" leaves five decisions open, each of which changes the code: full set vs EDI only; which columns are internal; what "empty date" means; how booleans look; which coverage number the NFR means. These are settled in [./spec.md](./spec.md) through [./spec-chat-log.md](./spec-chat-log.md).
