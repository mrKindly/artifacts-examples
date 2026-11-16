# Scaffold GetFulfilledSalesOrdersByMake
_Exported on 10/21/2026 at 12:09:55 GMT+3 from Cursor (3.25.9)_

---

**User**

/add-soap-operation FS07-3521 GetFulfilledSalesOrdersByMake docs/FS07-3521-fulfilled-orders-by-make/spec.md --reference GetSalesOrdersByDate

---

**Cursor**

I'll read the spec, `AGENTS.md` and the reference operation, check every column against the schema, then scaffold the operation with red tests and stop.

```
Read: spec.md (14 ACs: AC-1..AC-12 on-prem, AC-13..AC-14 router; 0 open questions), AGENTS.md, reference op GetSalesOrdersByDate.

Schema check (INFORMATION_SCHEMA.COLUMNS, FsCore_Dev):
  OK   dbo.SalesOrderLineFulfillment.FulfilledOn (datetime2)
  OK   dbo.SalesOrderLineFulfillment.ShipToPartyId (int, null)
  OK   dbo.Equipment.SerialNumber, MacAddress, ModelNumber, ModelDescription, MakeId
  OK   dbo.EquipmentMake.Make, MakeDescription
  OK   dbo.SalesOrderParty.PartyRole, PartyId; dbo.Party.Name, Address1, Address2, City, Region, PostalCode, CountryCode
  NOTE dbo.SalesOrder.LegacyEquipmentId exists (nullable, deprecated in migration 0147, populated only for pre-2019 orders).
       Spec joins equipment through SalesOrderLine.EquipmentId. The legacy column is NOT used.

Created:
  src/FS.Soap.Contracts/Sales/GetFulfilledSalesOrdersByMakeRequest.cs
  src/FS.Soap.Contracts/Sales/GetFulfilledSalesOrdersByMakeResponse.cs
  src/FS.Soap.Services/Sales/GetFulfilledSalesOrdersByMake.cs
  src/FS.Soap.Data/SalesRepository.cs                (method added)
  db/procs/usp_GetFulfilledSalesOrdersByMake.sql     (stub, header lists rules R1-R7)
  tests/FS.Soap.Tests/Sales/GetFulfilledSalesOrdersByMakeTests.cs   (12 stubs, one per AC)
  db/tests/usp_GetFulfilledSalesOrdersByMake/        (7 tSQLt stubs)
  tests/FS.Soap.Tests/Snapshots/service.wsdl         (diff: +1 operation, +4 types)
  docs/FS07-3521-fulfilled-orders-by-make/router-handoff.md

Build: succeeded, 0 warnings.
New tests: 19 run, 19 failed (NotImplemented / empty result), 0 compile errors.
WSDL snapshot test: passed after regeneration. Diff contains only the new operation.
```

Stopped. Waiting for your review of the red tests.

---

**User**

The `LegacyEquipmentId` note is right: the first prototype used that join and returned no equipment for orders created after 2019. Red tests look good, I'm not adding anything by hand. Go to the first implementation step from the plan (rules R1-R3).

---

**Cursor**

Starting the first implementation step from the plan: rules R1-R3, tests first.
