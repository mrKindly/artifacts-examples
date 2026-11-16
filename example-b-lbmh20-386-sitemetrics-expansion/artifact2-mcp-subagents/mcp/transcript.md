# SiteMetrics schema discovery for LBMH20-386
_Exported on 7/13/2026 at 14:37:22 GMT+3 from Cursor (3.11.6)_

---

**User**

Before we lock the spec scope for LBMH20-386: use `mssql-readonly` to discover what `dbo.SiteMetrics` has, which columns are categorised, and what the values look like. No guessing column names.

---

**Cursor**

I'll use the `mssql-readonly` MCP. It is connected to the **dev** `SpruceMetrics` database with a read-only login.

**Call 1 — `list_columns`** (`schema: "dbo"`, `table: "SiteMetrics"`)

51 columns. Grouped by purpose after cross-checking call 3:

| Group | Count | Columns (type) |
|-------|-------|----------------|
| Internal identity/ingest | 9 | `SiteMetricsId` (bigint, identity), `SiteId` (int), `IngestBatchId` (uniqueidentifier), `IngestedAtUtc` (datetime2), `SourceHost` (nvarchar), `SourceRowHash` (varbinary), `CollectorVersion` (nvarchar), `RowVersion` (rowversion), `IsDeleted` (bit) |
| Already on existing cards | 4 | `SnapshotDate` (datetime2), `ActiveUserCount` (int), `RemoteClientCount` (int), `RemoteClientVersionList` (nvarchar) |
| EDI | 10 | `EdiEnabled`, `EdiLastUsedDate`, `EdiPoEnabled`, `EdiPoLastUsedDate`, `EdiInvoiceEnabled`, `EdiInvoiceLastUsedDate`, `Edi856Enabled`, `Edi856LastUsedDate`, `EdiCatalogEnabled`, `EdiCatalogLastUsedDate` (bit / datetime2, alternating) |
| Integrations | 7 | `EcommerceEnabled`, `EcommerceLastSyncDate`, `PaymentsEnabled`, `PaymentsLastUsedDate`, `ShippingIntegrationEnabled`, `ShippingLastUsedDate`, `AccountingExportEnabled` |
| Communications | 5 | `EmailEnabled`, `EmailLastSentDate`, `TextMessagingEnabled`, `TextLastSentDate`, `FaxEnabled` |
| Mobile & Remote | 6 | `MobileAppEnabled`, `MobileLastLoginDate`, `HandheldEnabled`, `HandheldLastSyncDate`, `DriverAppEnabled`, `DriverAppLastUsedDate` |
| Reporting | 5 | `ReportingEnabled`, `ReportingLastRunDate`, `DashboardsEnabled`, `DashboardsLastViewedDate`, `ExportToExcelEnabled` |
| Environment | 5 | `SqlServerVersion` (nvarchar), `OsVersion` (nvarchar), `DatabaseSizeMb` (int), `LastBackupDate` (datetime2), `LastUpgradeDate` (datetime2) |

Type split of the 38 new columns: 18 bit, 17 date/datetime2, 2 nvarchar, 1 int. All 38 are nullable.

**Call 2 — `describe_table`** (`schema: "dbo"`, `table: "SiteMetricsCategory"`)

This is the metadata table the fleet dashboard already reads. Columns: `ColumnName` (nvarchar(128)), `Category` (nvarchar(64)), `DisplayName` (nvarchar(128)), `SortOrder` (int). No schema change is needed to use it.

**Call 3 — `select_top`**

```sql
SELECT TOP 50 Category, COUNT(*) AS Columns FROM dbo.SiteMetricsCategory GROUP BY Category ORDER BY MIN(SortOrder)
```

| Category | Columns in metadata |
|----------|---------------------|
| EDI | 10 |
| Integrations | 7 |
| Communications | 5 |
| Mobile & Remote | 8 (6 new + `RemoteClientCount`, `RemoteClientVersionList`) |
| Reporting | 5 |
| Environment | 7 (5 new + `SnapshotDate`, `ActiveUserCount`) |
| **Total** | **42** (6 categories) |

42 category-backed columns minus the 4 already on existing cards = **38** new.

**Call 4 — `select_top`** (what is not categorised)

```sql
SELECT TOP 50 c.COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS c LEFT JOIN dbo.SiteMetricsCategory m ON m.ColumnName = c.COLUMN_NAME WHERE c.TABLE_NAME = 'SiteMetrics' AND m.ColumnName IS NULL
```

9 rows, the same 9 internal identity/ingest columns as in call 1. So 51 columns - 42 categorised = 9 internal, none of which has a category row.

**Call 5 — `select_top`** (value shapes)

```sql
SELECT TOP 5 EdiEnabled, EdiLastUsedDate, EdiCatalogEnabled, EdiCatalogLastUsedDate, LastBackupDate FROM dbo.SiteMetrics ORDER BY IngestedAtUtc DESC
```

| EdiEnabled | EdiLastUsedDate | EdiCatalogEnabled | EdiCatalogLastUsedDate | LastBackupDate |
|-----------|-----------------|-------------------|------------------------|----------------|
| 1 | 2026-07-11 | 1 | 2026-06-30 | 2026-07-12 |
| 1 | 2026-07-09 | 0 | 1900-01-01 | 2026-07-12 |
| 0 | NULL | 0 | NULL | 2026-07-12 |
| 1 | 2026-07-10 | 0 | 0001-01-01 | NULL |
| 0 | 1900-01-01 | 0 | 1900-01-01 | 2026-07-11 |

Sentinel dates `1900-01-01` and `0001-01-01` exist next to real `NULL`s; disabled features have `0` plus a sentinel or `NULL` date; booleans are `bit`, never `NULL` in these rows but nullable in the schema.

**What goes into the spec:**

- Category-backed, shown: 38 columns in 6 categories (EDI 10, Integrations 7, Communications 5, Mobile & Remote 6, Reporting 5, Environment 5).
- Already shown, do not repeat: 4.
- Internal, never shown: 9.
- Sentinel dates to treat as empty: `1900-01-01`, `0001-01-01`.

So 51 columns = 9 internal (excluded) + 4 already shown + 38 new, in 6 categories.

Not used: any write tool (none exists), any other database, any table outside `INFORMATION_SCHEMA` and `dbo`.
