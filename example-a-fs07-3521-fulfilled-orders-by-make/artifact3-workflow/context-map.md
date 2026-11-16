# Context Map

Case: `FS07-3521`

Written 2026-10-20 with the `/context-map` workflow from [`spec.md`](./spec.md) and used as input to Plan Mode; updated 2026-10-21 after [`plan.md`](./plan.md) was approved. The map says what the agent was allowed to load per tier and why. More context was not better: the first session loaded the whole `Sales` folder and picked the wrong equipment join.

## Hot context

Directly required to implement and verify this case now.

| Resource | Why |
|----------|-----|
| `docs/FS07-3521-fulfilled-orders-by-make/spec.md` | Definitions, ACs, fault codes. |
| `docs/FS07-3521-fulfilled-orders-by-make/plan.md` | Step order, gates, files to touch. |
| `src/FS.Soap.Services/Sales/GetSalesOrdersByDate.cs` | Pattern for validation → repository → mapping. |
| `src/FS.Soap.Data/SalesRepository.cs` | Where the new repository method goes. |
| `db/procs/usp_GetSalesOrdersByDate.sql` | Proc layout and parameter style to mirror. |
| `db/migrations/0147_deprecate_legacy_equipment.sql` | Proves `LegacyEquipmentId` is deprecated and which join is right. |
| `INFORMATION_SCHEMA.COLUMNS` of `FsCore_Dev` (query result, saved in the spec session) | Schema first: confirms every column name. |
| `tests/FS.Soap.Tests/Snapshots/service.wsdl` | Must change only by the new operation. |
| `FS.Cloud.Router/src/Routes/SalesOrders/` and `src/Policies/ConsumerPolicies.cs` | Route and policy pattern for the router phase. |

## Warm context

Reusable guidance that constrains the work.

| Resource | Why |
|----------|-----|
| `AGENTS.md` | Map, commands, additive-contract convention, shared QA DB gotcha. |
| `.cursor/rules/sql-server-conventions.mdc` | No regex, UTC, read-only procs, LIKE escaping. |
| `.cursor/rules/soap-contracts.mdc` | Additive contracts, faults, WSDL snapshot. |
| `.cursor/rules/atlassian-jira-fs07.mdc` | Read-only Jira for FS07. |
| `.cursor/skills/add-soap-operation/SKILL.md` | The scaffold workflow. |
| `.cursor/agents/spec-reviewer.md` | The independent spec check. |
| PM walkthrough notes (2026-10-20) | Source of the watermark, grain, party and size rules. |

## Cold context

Open only if hot and warm are not enough.

| Resource | Why |
|----------|-----|
| FS07-3521 and FS07-3530 in Jira | Original text and the blocked sibling. |
| `src/FS.Soap.Services/Sales/` (other operations) | Only to compare conventions. |
| `db/functions/` (existing helpers) | To confirm no whole-word helper exists. |
| Router `README.md` | Deployment and switch conventions. |

## Resources to ignore

| Resource | Why |
|----------|-----|
| `CONTRIBUTING.md` | Describes the old ASMX layout. |
| `SalesOrder.LegacyEquipmentId` and anything using it | Deprecated, wrong for orders after 2019. |
| First prototype branch `fs07-3521-proto` | Wrong date basis, header status, wrong join. |
| `bin/`, `obj/`, generated WSDL outputs | Generated. |
| `src/FS.Soap.Services/Inventory/` | Unrelated module. |
