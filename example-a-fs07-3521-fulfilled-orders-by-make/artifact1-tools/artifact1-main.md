# Artifact 1 — Project Guidance Setup (Rules · AGENTS.md · Skill)

> Demo example: synthetic evidence modeled on the FS07-3521 story shape (new integration SOAP operation with ambiguous domain rules). Repo names, run output and timings are illustrative.

> Purpose: prove this repo has a reusable AI toolkit — passive rules, always-on project context, and at least one repeatable workflow. Everything here is reusable, maintainable, and shareable, not one-off.

---

## Inventory

| Tool | Type | Location | Used when |
|------|------|----------|-----------|
| sql-server-conventions | Rule | [`./rules/sql-server-conventions.mdc`](./rules/sql-server-conventions.mdc) | Agent opens any `db/**/*.sql` file |
| soap-contracts | Rule | [`./rules/soap-contracts.mdc`](./rules/soap-contracts.mdc) | Agent opens a contract or operation class |
| FS.OnPrem.Soap guide | AGENTS.md | [`./AGENTS.md`](./AGENTS.md) | Every session in this repo |
| add-soap-operation | Skill | [`./skills/add-soap-operation/SKILL.md`](./skills/add-soap-operation/SKILL.md) | A spec for a new SOAP operation is approved |

---

## 1. Rules (`./rules/*.mdc`)

### sql-server-conventions.mdc

- **Why we need it:** the first FS07-3521 prototype used MySQL-style regex ideas and `LIKE '%sharp%'`, both wrong on SQL Server 2019 (no regex, substring matches). The rule puts the SQL Server facts in front of the agent exactly when it edits SQL.
- **Why we avoided adding more:** it holds only constraints an agent gets wrong without being told (no regex, UTC `datetime2`, schema-first, read-only procs, LIKE escaping, test fixture prefix). Style preferences stay in the editor config.
- **Why not AGENTS.md or a skill:** it only matters for `.sql` files, so it is glob-scoped and does not cost context in C# work. A skill would need to be invoked; a rule applies passively.

### soap-contracts.mdc

- **Why we need it:** consumers (the cloud router and external integrators) are bound to the published WSDL. A silent change to an existing DTO breaks them without a failing build.
- **Why we avoided adding more:** six lines, all about backward compatibility and faults. Naming lives in AGENTS.md because it applies outside contracts.
- **Why not AGENTS.md or a skill:** the additive-only rule must be in context while the agent edits a `[DataContract]`, not only when it reads the repo guide at session start.

---

## 2. AGENTS.md

- **Why we need it:** every agent session needs the same map (host / contracts / services / data / `db/`), the Windows build and test commands, and the fact that `CONTRIBUTING.md` is stale.
- **Why we avoided adding more:** no code style, no architecture history, no per-ticket content. Task knowledge goes in the spec, not in a file loaded for every session.
- **Why not a rule or skill:** it is always needed, so it is always loaded. Rules and skills are loaded conditionally.
- **Gotchas:** the shared `FsCore_QA` database (fixtures must be prefixed and cleaned up), per-environment `Web.config` limits that feature PRs must not touch, read-only Jira scoped to `FS07`.

---

## 3. Skill(s) (`./skills/add-soap-operation/SKILL.md`)

### add-soap-operation

- **Why we need it:** "add a new `Get<Noun>By<Filter>` operation" repeats across integration tickets. Each time the same 8 files, the WSDL snapshot and the router hand-off note are created by hand, and the order is easy to get wrong.
- **Why reusable:** inputs are only task ID, operation name, spec path and an optional reference operation. Business rules are read from the approved spec, so the skill is the same for any read-only operation.
- **Why we avoided adding more:** it stops at red tests. Implementation of rules is test-first and phase by phase in the plan, with a human review after the red tests.
- **Why not a rule or AGENTS.md:** it is a multi-step workflow with inputs and outputs, invoked on purpose (`disable-model-invocation: true`).
- **Gotchas:** stale `obj/` can produce a WSDL without the new types; tSQLt stubs need one `NewTestClass` per folder.
- **Usage:**

```
/add-soap-operation FS07-3521 GetFulfilledSalesOrdersByMake docs/FS07-3521-fulfilled-orders-by-make/spec.md --reference GetSalesOrdersByDate
```

- **Log of the call with agent output:** [`./skills/add-soap-operation/log.md`](./skills/add-soap-operation/log.md). It created 9 artefacts, 19 red tests, and a WSDL diff with only the new operation. It also flagged the deprecated `dbo.SalesOrder.LegacyEquipmentId`, the wrong join the first prototype used.

---
