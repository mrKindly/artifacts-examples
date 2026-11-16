# LBMH.Services.Portal — agent guide

Internal support portal for the Spruce licensing platform: support engineers search licenses, open a license record and read its detail tabs (Metrics, Features, Remote Clients). Read-mostly: the Portal reads `SpruceLicensing` and `SpruceMetrics`; it does not own their schema.

Stack: .NET 10 ASP.NET Core API on ECS Fargate · React + Vite + Fluent UI v9 SPA (served from `Api/wwwroot` locally, S3/CloudFront in AWS) · AWS CDK (C#) · Azure DevOps (org `ECI-LBMH`, project `LBMH-Spruce`) · Okta OIDC/JWT · SQL Server (`SpruceLicensing`, `SpruceMetrics`, `DBPushUpdates`, `DevOpsWarehouse`) · Coralogix / OpenTelemetry.

## Map

| Area | Where | Notes |
|------|-------|-------|
| API | `Api/` | Feature folders (`Api/Licenses/...`); mappers turn data models into response DTOs. |
| Data access | `Data/` | Repositories over SQL Server. Read-only for `SpruceMetrics`. |
| SPA | `Web/` | `src/features/<feature>/` per screen; mock fixtures in `src/mocks/fixtures/`. Build output goes to `Api/wwwroot`. |
| Infra | `CDK/` | AWS CDK in C#. Change only when the task says so. |
| Tests | `Api.Tests/`, `Data.Tests/`, `CDK.Tests/`, `Web/src/**/*.test.ts(x)` | xUnit + Coverlet for .NET, Vitest for Web. |
| Pipelines | repo root `*.yaml` | `Deploy.LBMH.Services.Portal.{Api,App,Infrastructure}.yaml`, `QualityCheck.yaml`; `build.yaml` is an orphan (no active definition). |
| AI toolkit | `.cursor/` | `rules/`, `skills/` (incl. `deploy`, `pre-pr-review`), `agents/`, `mcp.json`. |
| Spec/plan docs | `docs/<STORY-KEY>/` | One folder per story: spec, plan, context map, verification. |

## Run

- Build SPA first: `cd Web && npm ci && npm run build` (writes `Api/wwwroot`). Required before the API serves the UI.
- Run API: `dotnet run --project Api --launch-profile http` → **http://localhost:5000**. Uses real Okta and SQL Server on `127.0.0.1,1434` / `127.0.0.1,1435`.
- **Docker Compose is CI/prod scaffolding only. Do not use it for local runs.**

## Verify (scoped first, then wider)

- Build: `dotnet build Api -c Debug` (baseline warnings must not grow).
- Tests: `dotnet test Api.Tests`, `dotnet test Data.Tests`, `dotnet test CDK.Tests`; Web: `cd Web && npm run test:ci`.
- Prefer a scoped run (`--filter "FullyQualifiedName~<Feature>"`, `npx vitest run <path>`) while iterating.
- **`npm run test:ci` exits 1 on `main` today** (pre-existing failures, 2 files). Judge Web changes by the scoped run of files you touched, and say so in the PR.
- Coverage on new/changed code must be >= 90% (see `rules/portal-test-coverage.mdc`).
- Before opening a PR run the `pre-pr-review` skill.

## Conventions

- Mappers are pure functions; keep display rules (Yes/No, `-`) in one place per layer: data normalisation in the API mapper, formatting in the Web formatter.
- No console logging of customer data; use the existing OpenTelemetry setup.
- Web: functional components, Fluent UI v9 tokens for colour (no hard-coded hex), Vitest + React Testing Library.
- Story work starts from a written spec in `docs/<STORY-KEY>/spec.md`. No code, no plan inside a spec.

## Gotchas

- **Schema-first.** Never invent `SiteMetrics` column names from naming patterns. Discover them through the `mssql-readonly` MCP (`list_columns`, `describe_table`) and quote the result in the spec.
- Some `SiteMetrics` date columns hold sentinel values (`1900-01-01`, `0001-01-01`) instead of `NULL`.
- A licensing-side `ediNames` string on another tab is a different source than `SiteMetrics` EDI flags.
- QualityCheck pipeline 445 is manual and usually red: Coverlet line gate is 90%, baseline is 87.72% (build 29300). Red does not mean your change broke it; compare against baseline.
- Deploys are manual, via `.cursor/skills/deploy/SKILL.md` to ADO definitions 440 (Api), 472 (App), 441 (Infra), Dev then Prod. Merge to `main` deploys nothing.

## Approval gates (stop and ask a human)

1. Any change under `CDK/`, pipeline YAML, or SQL scripts/migrations unless the spec lists it.
2. Any deploy, pipeline run, push, merge, or PR approval.
3. Any write to Jira or to a database. Jira and SQL access in this repo are read-only.
4. Scope expansion beyond the spec (extra cards, drill-ins, refactors).
