# FS.OnPrem.Soap — agent guide

On-prem SOAP service (WCF, .NET Framework 4.8) that exposes company data from the `FsCore` SQL Server database to integration consumers. Consumers reach it through the cloud router (`FS.Cloud.Router`, separate repo), which turns REST calls into SOAP calls.

## Map

| Area | Where | Notes |
|------|-------|-------|
| Service host | `src/FS.Soap.Host/` | `*.svc`, `Web.config`. No logic here. |
| Contracts | `src/FS.Soap.Contracts/` | `[ServiceContract]` interfaces and `[DataContract]` DTOs. One folder per operation group. |
| Operations | `src/FS.Soap.Services/` | One class per operation: validation → repository call → mapping. |
| Data access | `src/FS.Soap.Data/` | ADO.NET + stored procedures only. No ORM. |
| SQL | `db/procs/`, `db/functions/`, `db/migrations/` | Procs are the only way the service reads data. |
| Unit tests | `tests/FS.Soap.Tests/` | NUnit + NSubstitute. |
| DB tests | `db/tests/` | tSQLt. Run against a local `FsCore_Dev` copy. |
| WSDL snapshot | `tests/FS.Soap.Tests/Snapshots/service.wsdl` | Approved WSDL. A contract change must update it in the same PR. |

## Run and verify

- Build: `msbuild FS.Soap.sln /p:Configuration=Debug` (Windows or the `fs-build` container).
- Unit tests: `vstest.console tests\FS.Soap.Tests\bin\Debug\FS.Soap.Tests.dll`.
- DB tests: `powershell db\tests\run-tsqlt.ps1 -Database FsCore_Dev`.
- Local host: IIS Express, `http://localhost:8091/Sales.svc?wsdl`.
- Run scoped tests (one class or one tSQLt schema) while working. The full run is for the end of a phase.

## Conventions

- New operation = new request/response DTOs. Never add required members to an existing DTO.
- Operation names follow `Get<Noun>By<Filter>`. Dates cross the wire as UTC `DateTime`.
- Business rules live in procs and the operation class, not in the host.
- Schema first: read `INFORMATION_SCHEMA` (or the migration files) before naming a column. Do not infer a column from a table name.
- SQL Server 2019. There is no regex engine. Use the helpers in `db/functions/`.

## Jira

Atlassian access is read-only and limited to project `FS07` (see `.cursor/rules/` and `.cursor/mcp.json`). Do not create, edit, transition, comment on, or link issues.

## Gotchas

- `CONTRIBUTING.md` still describes the old ASMX layout. Follow the folders above.
- The shared QA database `FsCore_QA` is used by every branch. tSQLt tests that write fixtures must use the `tsqlt_` schema prefix and clean up, or other runs collide.
- `Web.config` binding limits (`maxReceivedMessageSize`) are set per environment by the deploy pipeline. Do not change them in a feature PR.
