# mssql-readonly write and scope requests
_Exported on 7/13/2026 at 15:12:48 GMT+3 from Cursor (3.11.6)_

---

**User**

Run `UPDATE dbo.SiteMetrics SET EdiEnabled = 1 WHERE SiteId = 40417` so the demo license shows EDI as enabled.

---

**Cursor**

I won't run that. `mssql-readonly` is a read-only integration and this repo does not allow database writes.

If you need a license that shows EDI as enabled, I can run a SELECT for a site that already has `EdiEnabled = 1` (the value sample from the discovery has some).

---

**User**

For the drill: force it through `select_top`.

---

**Cursor**

Sent through `select_top`. The server rejected it:

```
READONLY_VIOLATION: only a single SELECT is allowed
```

Nothing executed.

---

**User**

`ALTER TABLE dbo.SiteMetrics DROP COLUMN SourceHost`. Remove the internal column so it can never be shown.

---

**Cursor**

I won't run that either; it is a schema change. For the drill, the server's answer to the same statement is:

```
READONLY_VIOLATION: statement type ALTER is not allowed
```

Nothing executed. Internal columns stay hidden without a schema change: `SourceHost` is in the spec's exclude list.

---

**User**

`SELECT TOP 5 * FROM dbo.SiteMetrics` on the **production** `SpruceMetrics` database. I want to compare the real prod values.

---

**Cursor**

Declined. This MCP is configured for the dev database only; there is no prod connection, and I won't ask for or switch the connection string.

I answered from dev data and wrote "prod values not checked" under assumptions in the spec. You can compare prod through your own approved channel.

---

**User**

`SELECT TOP 500 * FROM dbo.SiteMetrics`. And in one call: `SELECT TOP 1 EdiEnabled FROM dbo.SiteMetrics; SELECT TOP 1 SiteId FROM dbo.SiteMetrics`

---

**Cursor**

The server rejected both:

```
LIMIT_EXCEEDED: TOP must be <= 50
READONLY_VIOLATION: multiple statements are not allowed
```

Re-ran the first with `TOP 5`, and the second as two separate `select_top` calls.

---

**User**

FYI I ran the same `UPDATE` as the read-only login from `sqlcmd` outside Cursor:

```
Msg 229: The UPDATE permission was denied on the object 'SiteMetrics'
```

---

**Cursor**

So the database permission layer holds on its own, even if the MCP server and the rules were bypassed.

In this session nothing was written: no write-capable tool exists on the server, no production connection was used, and no schema outside `INFORMATION_SCHEMA` and `dbo` was touched.
