# Read FS07-3521 from Jira
_Exported on 10/19/2026 at 15:21:08 GMT+3 from Cursor (3.25.4)_

---

**User**

Read FS07-3521 from Jira with its comments, subtasks and linked issues. I need the context for the spec.

---

**Cursor**

I'll use the Atlassian MCP. This connection requests `capabilities=READ_JIRA,SEARCH_JIRA` and exposes read tools only. The project rule `.cursor/rules/atlassian-jira-fs07.mdc` also applies: project `FS07` only, no write tools, `maxResults` at most 10.

Calls made:

| # | Tool | Arguments | Result |
|---|------|-----------|--------|
| 1 | `getAccessibleAtlassianResources` | none | one site, `https://eci-solutions.atlassian.net/`, `cloudId` reused below |
| 2 | `getJiraIssue` | `FS07-3521`, `view: evidence` | description, AC table, 2 comments, parent epic `FS07-3442`, 1 link ("blocks FS07-3530") |
| 3 | `searchJiraIssuesUsingJql` | `project = FS07 AND parent = FS07-3521`, `maxResults: 10` | no subtasks |
| 4 | `getJiraIssue` | `FS07-3530`, `view: compact` | REST wrapper ticket, status "Blocked by FS07-3521" |

What matters for the spec:

- The ticket says "fulfilled" three ways: "Fulfilled" status, "fulfillment date", and "fulfilled equipment". There is no definition.
- Comment 1 (consumer team): the caller keeps its own sync clock and passes the last time it synced.
- Comment 2 (dev): the first prototype looked back from the order create date.
- Sibling `FS07-3530` (REST wrapper) is blocked by this ticket. It is out of scope here.

No write tool was called. No issue outside `FS07` was read.
