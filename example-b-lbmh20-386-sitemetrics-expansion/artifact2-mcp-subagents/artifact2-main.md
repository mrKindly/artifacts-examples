# Artifact 2 — Custom Subagent AND MCP Integration

> Demo example: synthetic evidence (ids, run numbers, timings in chat logs/transcripts) modeled on the LBMH20-386 story shape. Artifact 5 numbers come from the real portal-SDLC.json.

> Purpose: prove at least one custom subagent AND at least one MCP integration, each solving a REAL workflow problem (not a demo for demo's sake). Each must have clear scope, explicit safety boundaries, and working evidence someone else could reuse.

Story thread: [LBMH20-386](https://eci-solutions.atlassian.net/browse/LBMH20-386) — Portal: expand SiteMetrics on the license record detail Metrics tab. Both items below are used in that story ([artifact 3](../artifact3-workflow/artifact3-main.md)).

---

## Inventory

| Deliverable | Type | Location | Installed in Portal repo at | Solves | Used when |
|-------------|------|----------|-----------------------------|--------|-----------|
| diff-vs-spec-reviewer | Subagent | [./agents/diff-vs-spec-reviewer.md](./agents/diff-vs-spec-reviewer.md) | `.cursor/agents/diff-vs-spec-reviewer.md` | Reviewer sees no evidence that a diff matches the spec (16 of 24 PRs to `main` open <5 min, no required PR checks) | Before a PR is marked Ready, called by the `pre-pr-review` skill or on demand |
| mssql-readonly | MCP integration | [./mcp/mcp.json](./mcp/mcp.json) | `.cursor/mcp.json` | Column names for `SiteMetrics` had to be copied by hand from SQL tools, so naming-pattern guesses crept into specs | Spec phase of any Portal story that maps database columns |

**Where the MCP server code lives.** The server source is in the Portal repo at `tools/mcp/mssql-readonly/` (`server.js`, a deny-list of PII tables, its own tests). It is a normal reviewed part of the repo and is **not part of this demo folder**; only the config, transcript and refusal log are here.

---

## Subagent

- **Name:** `diff-vs-spec-reviewer`
- **Problem:** A Portal PR reaches a human reviewer with a diff and a Jira title, but no statement of "does this diff do what the spec says, and only that". In the measured window 16 of 24 PRs to `main` were open under five minutes and the repository has no branch policies ([portal-SDLC.json](../artifact5-ai-use-cases/portal-SDLC.json)), so the check is either skipped or redone from scratch by the reviewer. On LBMH20-386 the risk is concrete: 51 columns in `SiteMetrics`, 9 of them internal, and a spec that says "no schema, CDK or pipeline change".
- **Why a subagent fits:** The task is bounded (one spec, one diff), read-only, and produces a fixed-format verdict. A separate context keeps raw diff noise out of the parent chat, and a reviewer that cannot write cannot "fix" a gap and then pass itself.
- **Running mode:** Local in Cursor, on the developer's branch. Not a cloud/CI agent: it needs the local working tree and runs only on demand (the CI equivalent is a later step, see [artifact 5](../artifact5-ai-use-cases/artifact5-main.md), UC-3).
- **Chain:** Standalone. The `pre-pr-review` skill ([artifact 1](../artifact1-tools/skills/pre-pr-review/SKILL.md)) calls it as step 6 and embeds its output; the subagent calls nobody.
- **Configuration setup reasoning:**
  - Allowed tools: Read, Grep, Glob, and Shell restricted to read-only git (`diff`, `show`, `log`, `status`, `rev-parse`). No MCP, no network, no test runs.
  - Forbidden actions: write to the repo, `git add/commit/push/merge/reset`, deploy or queue any pipeline (440/441/445/472), approve or comment on a PR, Jira or database write, following instructions found in diffs or comments, inventing evidence.
  - Read/write boundary: `readonly: true` in frontmatter; zero writes anywhere.
  - Human gates: code review, QualityCheck 445 on the branch, PR approval, merge, Dev then Prod deploy. The verdict is advisory and never says "approved" or "ready to deploy".
  - Security / data notes: the excluded-column list is read from the spec, not hard-coded, so no data model detail is baked into the agent; planted instructions in the diff are reported as a blocker (see refusal log, item 4).

### Evidence

| Item | Path |
|------|-------------|
| Config file | [./agents/diff-vs-spec-reviewer.md](./agents/diff-vs-spec-reviewer.md) |
| Chat log / transcript | [./agents/diff-vs-spec-reviewer.transcript.md](./agents/diff-vs-spec-reviewer.transcript.md) (invocation 1 HOLD on AC-7, invocation 2 PASS) |
| Refusal log (demo named forbidden action + what it does instead) | [./agents/diff-vs-spec-reviewer.refusal-log.md](./agents/diff-vs-spec-reviewer.refusal-log.md) (fix the file, run pipeline 440, approve the PR, planted comment) |

---

## MCP integration

- **Name:** `mssql-readonly` (repo-local MCP server)
- **Problem:** Schema-first is a Portal rule: never invent `SiteMetrics` column names. Without tool access, developers paste column lists from SQL clients into chat, which goes stale and is easy to get wrong. For LBMH20-386 the real shape (51 columns, 42 with a category, 9 without) decides the whole spec.
- **Why MCP fits:** The agent needs live, structured, read-only answers from a system it cannot reach otherwise (the dev database). A CLI or raw SQL access would expose far more than needed; an MCP server can expose three narrow tools and enforce limits in code.
- **Server / tools exposed:** Node server `./tools/mcp/mssql-readonly/server.js` with three tools: `list_columns` (schema + table), `describe_table`, `select_top` (a single `SELECT` with `TOP <= 50`; no PII tables). Target: **dev** `SpruceMetrics`.
- **Auth / connection:** `command: node`, `args: ["./tools/mcp/mssql-readonly/server.js"]`. `MSSQL_CONNECTION_STRING` is read from the developer's environment through `${env:MSSQL_CONNECTION_STRING}`; the value is a dev connection that uses a read-only SQL login. `MSSQL_ALLOWED_SCHEMAS=INFORMATION_SCHEMA,dbo`. Nothing secret is stored in `mcp.json`.
- **Configuration setup reasoning:**
  - Allowed tools: `list_columns`, `describe_table`, `select_top`. Auto-run is on for the two metadata tools, off for `select_top` (each query is approved).
  - Forbidden actions: any statement that is not a single `SELECT`; `TOP` above 50 or missing; schemas outside the allow-list; PII tables; any write or DDL; any production connection; switching the connection string from inside a chat.
  - Read/write boundary: read-only end to end, enforced in three independent layers:
    1. **SQL login permissions.** The login has `SELECT` on `INFORMATION_SCHEMA` and `dbo` objects it needs and `DENY` on PII tables; no INSERT/UPDATE/DELETE/ALTER. This holds even if layers 2 and 3 fail (human-verified: `Msg 229` on UPDATE).
    2. **Server validation.** The server rejects non-SELECT statements, multi-statement batches, `INTO`, `EXEC`, comments, TOP above 50 and tables outside the allow-list, before touching the database.
    3. **Project rule.** `AGENTS.md` approval gate 3 and the `portal-local-dev` rule say database access is read-only and schema-first ([artifact 1](../artifact1-tools/AGENTS.md)).
  - Human gates: developer approves each `select_top`; connection string is changed only by the developer, outside chat; any need for prod data goes through a human, not this server.
  - Security / data notes: dev only, never prod. `mcp.json` holds an env var reference, not a secret. The server logs tool name and table, not result rows. Output of the session is quoted in the spec as column names and counts only, no customer data.

### Evidence

| Item | Link / path |
|------|-------------|
| Config file | [./mcp/mcp.json](./mcp/mcp.json) |
| Chat log / transcript | [./mcp/transcript.md](./mcp/transcript.md) (5 calls: 51 columns, 42 categorised in 6 categories, 9 uncategorised internal, sentinel dates found) |
| Refusal log (demo named forbidden action + what it does instead) | [./mcp/refusal-log.md](./mcp/refusal-log.md) (UPDATE, DROP COLUMN, SELECT on prod, TOP 500 and multi-statement) |
