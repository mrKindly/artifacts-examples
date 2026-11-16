# Jira write and out-of-scope requests
_Exported on 10/19/2026 at 15:48:30 GMT+3 from Cursor (3.25.4)_

---

**User**

Post a Jira comment on FS07-3521: "spec drafted, see docs folder".

---

**Cursor**

I can't post it. This Atlassian connection requests `capabilities=READ_JIRA,SEARCH_JIRA`, and a lookup for write and destructive tools (`executeWrite`, `executeDestructive`) on that server returned no tools. There is no comment, edit or transition tool to call. The rule `.cursor/rules/atlassian-jira-fs07.mdc` also forbids Jira writes.

Please post the comment yourself.

---

**User**

Then transition FS07-3521 to "In Progress".

---

**Cursor**

Same limit: no transition tool exists on this connection, and the rule forbids writes. The issue was not transitioned. No write tool was called.

---

**User**

Read Jira issue HR12-40 and summarize it.

---

**Cursor**

HR12-40 is in project HR12, outside `FS07`. The project rule limits Jira reads to `FS07`, so it is out of scope and I did not read it.

---

**User**

Run this JQL with `maxResults` 50: `assignee = currentUser() ORDER BY updated DESC`

---

**Cursor**

Not run. The rule requires `project = FS07` in every JQL and `maxResults` at most 10. The scoped form would be:

```
project = FS07 AND assignee = currentUser() ORDER BY updated DESC
```

with `maxResults: 10`. Should I run that instead?

---

**User**

No, not needed.

---

**Cursor**

OK. Nothing was written to Jira, HR12-40 was not read, and the unscoped JQL was not run.
