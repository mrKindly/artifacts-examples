# FS07-3521 — Fulfilled sales orders by equipment make

Original work item, read from Jira on 2026-10-19 through the read-only Atlassian MCP ([transcript](../artifact2-mcp-subagents/mcp/transcript.md)). Reference shape: new integration API with ambiguous domain rules.

[FS07-3521](https://eci-solutions.atlassian.net/browse/FS07-3521) · Epic FS07-3442 · blocks FS07-3530 (REST wrapper)

| Field | Value |
|-------|--------|
| Key | FS07-3521 |
| Project | FS07 |
| Type | Story |
| Status | Ready for development |
| Repos | `FS.OnPrem.Soap` (SOAP service, SQL procs), `FS.Cloud.Router` (REST router, Swagger) |
| Created | 2026-10-12 |

## Description

**As a** downstream integration consumer,
**I want to** retrieve fulfilled sales orders by equipment make (whole-word match) and a fulfillment watermark datetime,
**So that** I can register accounts and devices from ship-to / bill-to / equipment identity without dual entry.

## Acceptance criteria (as written in Jira)

| ID | Criterion |
|----|-----------|
| T-AC1 | The operation returns sales orders that have fulfilled equipment. |
| T-AC2 | The make filter matches a whole word in **Make**. |
| T-AC3 | The make filter matches a whole word in **Make Description**. |
| T-AC4 | Substrings do not match (`Sharp` does not match `Sharpie`). The match is case-insensitive. |
| T-AC5 | The caller passes a datetime. Only orders with fulfillment activity after it are returned. |
| T-AC6 | The response includes the ship-to address. |
| T-AC7 | The response includes the bill-to address. |
| T-AC8 | The response includes equipment identity: serial, MAC, make and model fields. |

## Comments

- **Consumer team, 2026-10-14:** "We store the last time we synced and send it back. We do not want the service to remember us."
- **Developer, 2026-10-16:** "Prototype looks back from order create date and checks the header status. Needs review before we go further."

## Related

- Parent: FS07-3442 (equipment registration integration).
- Blocks: FS07-3530 (REST wrapper over this operation). Out of scope here.

## Gaps noted at intake

- "Fulfilled" is used for the order, the line and the equipment. No definition.
- "Datetime" has no timezone and no comparison rule.
- No row grain: one row per order, per line, or per serial?
- No limit on the result size.
