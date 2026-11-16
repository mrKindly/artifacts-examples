---
name: add-soap-operation
description: Scaffolds a new read-only SOAP operation end to end (contract DTOs, operation class, repository method, stored proc, unit tests, tSQLt tests, WSDL snapshot, router hand-off note). Use when a story asks for a new Get<Noun>By<Filter> operation, "a new SOAP op", or "follow the pattern of operation X".
disable-model-invocation: true
---

# Add SOAP operation

Creates the skeleton and the failing tests for one new read-only operation. It does not decide business rules: those come from an approved spec.

## When to use

- A spec exists in `docs/<TASK-ID>-<slug>/spec.md` with accepted ACs and a request/response field list.
- The work is a **new** operation. For a change to an existing operation, stop and ask: contracts are additive only.

## Inputs

| Input | Required | Example |
|-------|----------|---------|
| Task ID | Yes | `FS07-3521` |
| Operation name | Yes | `GetFulfilledSalesOrdersByMake` |
| Spec path | Yes | `docs/FS07-3521-fulfilled-orders-by-make/spec.md` |
| Reference operation | No | `GetSalesOrdersByDate` (pattern to mirror) |

If the spec path is missing or the spec has open questions, stop and report. Do not guess fields.

## Outputs

| Artefact | Path |
|----------|------|
| Request/response DTOs | `src/FS.Soap.Contracts/<Group>/<Operation>Request.cs`, `...Response.cs` |
| Operation class | `src/FS.Soap.Services/<Group>/<Operation>.cs` |
| Repository method | `src/FS.Soap.Data/<Group>Repository.cs` |
| Stored proc (empty body + header comment of the rule list) | `db/procs/usp_<Operation>.sql` |
| Unit test stubs, one per AC | `tests/FS.Soap.Tests/<Group>/<Operation>Tests.cs` |
| tSQLt test stubs | `db/tests/usp_<Operation>/` |
| Updated WSDL snapshot | `tests/FS.Soap.Tests/Snapshots/service.wsdl` |
| Router hand-off note | `docs/<TASK-ID>-<slug>/router-handoff.md` (operation, request/response JSON shape, fault codes) |

## Workflow

1. Read the spec and `AGENTS.md`. List the ACs and the request/response fields. Read the reference operation if one is given.
2. Check every column the spec names against `INFORMATION_SCHEMA.COLUMNS` or `db/migrations/`. Report any name that does not exist; do not substitute.
3. Create the DTOs and the operation class. Validation faults come from the spec's error-handling section.
4. Create the proc file with the rule list as a header comment and a stub body that returns the correct columns with zero rows.
5. Create one failing test per AC (NUnit and tSQLt). Test names state the behavior, for example `Get_ShouldExcludeSubstringMakes_WhenMakeIsASuperstring`.
6. Regenerate the WSDL snapshot and show the diff. It must contain only the new operation and its types.
7. Run build and the new tests. Expected result: build green, new tests red for the right reason (not compile errors).
8. Write `router-handoff.md`.

## Hard rules

- Read-only database work. The skill never runs a script that writes to `FsCore_QA` or any shared database.
- No business logic beyond validation. Proc bodies and mapping are implemented in later, test-first steps.
- Do not edit existing contracts, existing procs, or `Web.config`.
- Stop after step 8 and wait for the human to review the red tests.

## Gotchas

- `msbuild` on a stale `obj/` can emit a WSDL without the new types. Clean the Host project first.
- tSQLt stubs must call `tSQLt.NewTestClass` once per proc folder or the run script skips them.
