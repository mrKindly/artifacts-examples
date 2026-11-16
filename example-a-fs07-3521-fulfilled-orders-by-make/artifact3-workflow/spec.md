# Specification

Source: `docs/FS07-3521-fulfilled-orders-by-make/spec.md` in `FS.OnPrem.Soap`. Written 2026-10-19 with the `/spec` workflow from [FS07-3521](./story.md), revised after the PM walkthrough on 2026-10-20. `spec-reviewer`: FAIL on iteration 1, PASS on iteration 2 ([transcript](../artifact2-mcp-subagents/agents/spec-reviewer.transcript.md)).

## Business Goal

FS07-3521: an integration consumer can ask for the equipment that was fulfilled since the last time it synced, for one equipment make, and receive what it needs to register the customer and the device.

**Problem / pain description**

Existing sales-order operations return commerce-heavy payloads (prices, full line catalogs) and cannot filter by equipment make or by fulfillment activity. The consumer now reads whole orders, filters on its side, and re-keys ship-to, bill-to and equipment data by hand. The Jira text leaves "fulfilled", the watermark and the row shape undefined, and a first prototype guessed them wrong.

**Proposed solution**

A new read-only SOAP operation, `GetFulfilledSalesOrdersByMake`, and a REST route on the cloud router that calls it. The operation returns one compact row per fulfilled equipment serial whose make matches a whole word and whose latest fulfillment is after the caller's watermark.

## Current Behavior

**Service (`FS.OnPrem.Soap`)**

- Sales operations live in the `Sales` group. `GetSalesOrdersByDate` returns order headers with lines and prices, filtered by order create date. No operation filters by equipment make or fulfillment.
- Contracts are additive only. Existing DTOs cannot gain required members.

**Data (`FsCore`, SQL Server 2019)**

- Fulfillment activity is stored per order line in `SalesOrderLineFulfillment` (`FulfilledOn`, optional `ShipToPartyId`). An order line points to one equipment record (`SalesOrderLine.EquipmentId`). Equipment points to a make (`EquipmentMake.Make`, `EquipmentMake.MakeDescription`).
- Parties are stored in `SalesOrderParty` (`PartyRole` = `ShipTo` or `BillTo`) and `Party`.
- `SalesOrder.Status` is a header value. It can read `Fulfilled` while no line has a fulfillment record (data fix-ups) and can read `Open` while some lines are fulfilled.
- `SalesOrder.LegacyEquipmentId` exists, is deprecated (migration 0147) and is populated only for orders created before 2019.
- The database has no regex support.

**Cloud router (`FS.Cloud.Router`)**

- Routes are registered per operation with an authorization policy per consumer. OpenAPI (Swagger) is generated from route metadata. A route for this operation does not exist.

## Expected Behavior

### Definitions

| Term | Meaning in this spec |
|------|----------------------|
| Fulfilled line | A sales-order line with at least one `SalesOrderLineFulfillment` record. The header `Status` is not used. Partial fulfillment is allowed. |
| Fulfillment time | The latest `FulfilledOn` among the fulfillment records of that line. |
| Watermark | The `FulfilledSince` value sent by the caller, a UTC instant. A line qualifies when its fulfillment time is strictly greater than the watermark. |
| Whole-word match | The make text is split into words at every character that is not a letter or a digit. The search value (normalized the same way) must match one word or a run of consecutive words, ignoring case. |
| Row | One equipment serial. |

### Behavior

1. The caller sends `Make` and `FulfilledSince`.
2. The service returns one row for every equipment serial that has a fulfilled line, whose fulfillment time is after the watermark, and whose `Make` or `MakeDescription` contains the search value as a whole word.
3. Each row carries the order number, fulfillment time, equipment identity (serial, MAC, make, make description, model number, model description), the ship-to address and the bill-to address.
4. Ship-to is the fulfillment record's ship-to party when present, otherwise the order's ship-to party. Bill-to is the order's bill-to party.
5. The service keeps no state about the caller. The caller owns its sync clock.
6. The router exposes the operation as an authorized REST route and documents it in Swagger.

### Whole-word examples (search value `Sharp`)

| Make / Make Description | Matches |
|-------------------------|---------|
| `Sharp` | Yes |
| `Sharp Electronics` | Yes |
| `SHARP-MX` | Yes (hyphen is a separator) |
| `Sharpie` | No |
| `Sharper Image` | No |

### User Flow

1. The consumer stores `lastSync` (UTC).
2. It calls the route with `make = "Sharp"` and `fulfilledSince = lastSync`.
3. It registers the returned devices and customers.
4. It sets `lastSync` to the largest `fulfilledOn` it received (or keeps it when no rows came back) and repeats later.

### Edge Cases

- A line fulfilled twice appears once, with the later fulfillment time.
- The same serial on two orders (resale) appears once, with the latest fulfillment across both.
- A fulfillment time equal to the watermark is not returned.
- The header says `Fulfilled` but no line has a fulfillment record: nothing is returned for that order.
- The header says `Open` and one line is fulfilled: that line's equipment is returned.
- A make that matches only in `MakeDescription` is returned. A make that matches only in `Make` is returned.
- No ship-to party for a row: the ship-to fields are empty and the row is still returned. Same for bill-to.
- A search value that contains `%`, `_` or `[` is rejected by validation; it never reaches a pattern match.
- Equipment with no make record is never matched.

### Error Handling

All failures are SOAP faults of type `ValidationFault` with a stable `Code`; the router maps them to HTTP 400 with the same code, except `TooManyRows` (HTTP 422).

| Code | When |
|------|------|
| `InvalidMake` | `Make` is empty, whitespace, longer than 50 characters, or has characters other than letters, digits, space, hyphen, ampersand, period or comma |
| `InvalidWatermark` | `FulfilledSince` is missing or is the default `DateTime` |
| `TooManyRows` | More than 5000 rows would be returned. No partial result is sent. The caller narrows the watermark. |

An unauthorized caller gets HTTP 401 or 403 from the router and never reaches the service.

## Out Of Scope

- FS07-3530 (REST wrapper ticket) and any second consumer.
- Prices, amounts, full line catalogs, shipments and tracking numbers.
- Pagination or cursors (the row cap fault is the only size control).
- Changes to existing operations, DTOs, procs or tables. No schema change.
- Remembering a caller's last sync on the server.
- Fixing header `Status` data quality.

## Technical Scope

- **On-prem service:** one new operation with its own request and response contracts in the `Sales` group; validation; one new stored procedure and one helper function for whole-word matching; unit and database tests; WSDL snapshot update.
- **Cloud router:** one new route with a consumer authorization policy and a route-level enable switch; Swagger entries; router tests.
- **Data:** read-only access to the tables listed under Current Behavior.

## Non-functional Requirements and Constraints

- .NET Framework 4.8 and WCF on-prem; ASP.NET Core 8 on the router. SQL Server 2019 with stored procedures only.
- Contracts are additive. The WSDL snapshot test passes in the same PR.
- All instants are UTC.
- The proc is read-only and uses explicit column lists.
- Response for 5000 rows completes in under 3 seconds on the dev database copy (measured in verification).
- Logs carry operation name, row count and elapsed time, never addresses.
- Router route is behind a switch that is off by default in Production until the consumer smoke test passes.

## Acceptance Criteria

| ID | Criterion |
|----|-----------|
| AC-1 | Only equipment on lines with at least one fulfillment record is returned. A header with status `Fulfilled` and no fulfillment record returns nothing; a partially fulfilled order returns its fulfilled lines. |
| AC-2 | A whole-word match on `Make` returns the row. |
| AC-3 | A whole-word match on `MakeDescription` returns the row when `Make` does not match. |
| AC-4 | Substring matches are excluded: `Sharp` does not return `Sharpie` or `Sharper Image`. |
| AC-5 | Matching ignores case and treats any non-letter, non-digit character as a word separator (`SHARP-MX` matches `sharp`). |
| AC-6 | Only lines whose fulfillment time is strictly after `FulfilledSince` (UTC) are returned. A value equal to the watermark is excluded. |
| AC-7 | Each row has the ship-to and bill-to name and address fields. Ship-to uses the fulfillment record's ship-to party first, then the order's ship-to. Missing parties give empty fields, not an error. |
| AC-8 | Each row has order number, fulfillment time, serial, MAC, make, make description, model number and model description. |
| AC-9 | One row per equipment serial. Repeated fulfillment or the same serial on two orders yields one row with the latest fulfillment time. |
| AC-10 | When nothing matches, the response has an empty row list and no fault. |
| AC-11 | When more than 5000 rows match, the service raises `TooManyRows` and returns no rows. |
| AC-12 | Invalid `Make` or `FulfilledSince` raises `InvalidMake` or `InvalidWatermark`. No existing contract, proc or table changes, and the WSDL diff contains only the new operation. |
| AC-13 | The router route accepts only the registered consumer, maps faults as in Error Handling, and returns 404 while the route switch is off. |
| AC-14 | Swagger documents the route: request, response, fault codes and a worked example using `Sharp`. |

## Assumptions

- A1. `SalesOrderLine.EquipmentId` is the correct path from a line to equipment. `SalesOrder.LegacyEquipmentId` is ignored.
- A2. UTC is the clock of record. A `DateTime` with `Kind = Unspecified` from the wire is treated as UTC and documented in Swagger.
- A3. In `FsCore_Dev` (21,406 fulfillment rows), 8,134 rows (38%) carry a `ShipToPartyId`. The fulfillment-first rule therefore changes the result for a material share of rows and is covered by tests.
- A4. The consumer's expected volume per call is far below 5000 rows when it syncs at least daily. The cap protects the service, not normal use.
- A5. Whole-word matching on a multi-word search value means those words adjacent in that order (`Sharp Electronics`).
- A6. The word "fulfilled" is confirmed with the PM: partial fulfillment counts, header status does not. Recorded in [`spec-chat-log.md`](./spec-chat-log.md).
