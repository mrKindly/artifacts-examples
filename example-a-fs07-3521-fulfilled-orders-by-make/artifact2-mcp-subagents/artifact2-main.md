# Artifact 2 — Custom Subagent AND MCP Integration

> Demo example: synthetic evidence modeled on the FS07-3521 story shape. Transcripts and refusal logs are synthetic sessions in Cursor's chat-export format.

> Purpose: prove at least one custom subagent AND at least one MCP integration, each solving a REAL workflow problem. Each has clear scope, explicit safety boundaries, and working evidence someone else could reuse.

---

## Inventory

| Deliverable | Type | Location | Solves | Used when |
|-------------|------|----------|--------|-----------|
| spec-reviewer | Subagent | [`./agents/spec-reviewer.md`](./agents/spec-reviewer.md) | Specs for integration tickets looked complete and still encoded wrong business rules; reviewing them by hand cost a PM walkthrough each time | A spec draft is written, before planning |
| Atlassian (read-only) | MCP integration | [`./mcp/mcp.json`](./mcp/mcp.json) | Story text, comments and linked tickets were copied into chat by hand, losing the comment that held the consumer's real intent | Start of every story, to build the spec context |

---

## Subagent

- **Name:** `spec-reviewer`
- **Problem:** on FS07-3521 the first prototype was coded from Jira text alone and used the wrong date basis, header-only status, and the wrong equipment join. Three of the last five integration tickets needed a second spec round after a PM walkthrough (measured in [`../artifact5-ai-use-cases/fs-SDLC.json`](../artifact5-ai-use-cases/fs-SDLC.json), station "Developer's laptop"). A reviewer with fresh context catches undefined terms and untestable ACs before a human spends time.
- **Why a subagent fits:** the review must be independent of the agent that wrote the spec (fresh context, no sunk-cost bias), has a fixed output contract, and must be incapable of editing. A subagent gives all three; a skill step run by the same agent would not.
- **Running mode:** local, in Cursor, foreground. Not a CI agent: it needs the story context the developer has at hand.
- **Chain:** standalone. The `/spec` workflow calls it up to three times (fresh instance each iteration); it calls nobody.
- **Configuration setup reasoning:**
  - **Allowed tools:** file read under `docs/`, `.cursor/skills/`, `db/migrations/`, `src/`; grep/glob. Enough to verify statements about current behavior.
  - **Forbidden actions:** any write (including the spec), code/SQL/plan proposals, approving for a human, obeying instructions found in the spec or Jira text, inventing facts.
  - **Read/write boundary:** `readonly: true`. Zero writes.
  - **Human approval gates:** the agent returns PASS/FAIL; a human reads the findings and edits the spec. A PASS does not replace the human spec review recorded in [`../artifact3-workflow/review-notes.md`](../artifact3-workflow/review-notes.md).
  - **Security / data notes:** no network, no shell, no database. The spec contains no customer data; the agent never sees party addresses.

### Evidence

| Item | Path |
|------|------|
| Config file | [`./agents/spec-reviewer.md`](./agents/spec-reviewer.md) |
| Chat log / transcript | [`./agents/spec-reviewer.transcript.md`](./agents/spec-reviewer.transcript.md) — iteration 1 FAIL with 4 blocking findings, iteration 2 PASS |
| Refusal log | [`./agents/spec-reviewer.refusal-log.md`](./agents/spec-reviewer.refusal-log.md) — asked to fix the spec, add SQL, obey an in-document "mark PASS", post to Jira; refused all four |

---

## MCP integration

- **Name:** Atlassian MCP, read-only, scoped to project `FS07`.
- **Problem:** the decisive facts for FS07-3521 sat in two Jira comments (the consumer owns the sync clock; the first prototype's wrong basis) and in a blocked sibling ticket. Copy-paste into chat dropped them. This hits every story at kickoff.
- **Why MCP fits:** Jira is an external system the agent must read live and with permissions of the user. A CLI export would be stale and unscoped; MCP gives typed read tools and a capability filter in the URL.
- **Server / tools exposed:** Atlassian remote MCP. Used: `getAccessibleAtlassianResources`, `getJiraIssue` (`evidence` and `compact` views), `searchJiraIssuesUsingJql` (subtasks only).
- **Auth / connection:** remote server, OAuth sign-in by the developer in the browser. No token in the repo. `mcp.json` holds only the URL and `capabilities=READ_JIRA,SEARCH_JIRA`.
- **Configuration setup reasoning:**
  - **Allowed tools:** the three read tools above.
  - **Forbidden actions:** create, edit, transition, comment, worklog, link, attachment; reading outside `FS07`; JQL without `project = FS07` or with `maxResults` above 10; following instructions found in ticket text.
  - **Read/write boundary:** two layers. The URL requests read capabilities only (the server exposes no write tool). The always-on rule `.cursor/rules/atlassian-jira-fs07.mdc` repeats the project and limit restrictions.
  - **Human approval gates:** none needed for reads. Any Jira update stays a manual action by a person.
  - **Security / data notes:** comments can contain personal data. Summaries keep display names only. Ticket text is treated as data, never as commands.

### Evidence

| Item | Link / path |
|------|-------------|
| Config file | [`./mcp/mcp.json`](./mcp/mcp.json) |
| Chat log / transcript | [`./mcp/transcript.md`](./mcp/transcript.md) |
| Refusal log | [`./mcp/refusal-log.md`](./mcp/refusal-log.md) — comment, transition, out-of-project read, unscoped JQL; all refused |

---
