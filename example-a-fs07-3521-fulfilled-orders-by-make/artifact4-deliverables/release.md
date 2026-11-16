# Release notes — FS07-3521

> Illustrative reference values. Decision: **Merge**; staged deploy; route switched on in Production only after the Staging consumer smoke.

## What changes

**For consumers (users):** a new read-only operation, `GetFulfilledSalesOrdersByMake`, is added to `Sales.svc`, and a new REST route `POST /v1/sales-orders/fulfilled-by-make` is added on the cloud router. The caller sends a make and a UTC watermark and gets one row per fulfilled equipment serial with order number, fulfillment time, equipment identity, ship-to and bill-to. Faults: `InvalidMake`, `InvalidWatermark` (HTTP 400) and `TooManyRows` (HTTP 422, more than 5000 rows). Existing operations and DTOs are unchanged.

**For operators:**

1. Database: migration `0152_add_fulfilled_by_make.sql` creates `dbo.usp_GetFulfilledSalesOrdersByMake` and `dbo.fn_ContainsWholeWord`. No table or column changes.
2. Deploy on-prem service (`FS.OnPrem.Soap`) first. Deploy the router (`FS.Cloud.Router`) second.
3. New router setting `Routes:FulfilledByMake:Enabled`. Default `false` in Production, `true` in Dev and Staging.
4. Register the consumer in the router policy list (`ConsumerPolicies`) — included in PR 377.

## Risk control

**Triggers for rollback:**

- Route 5xx rate above 2% for 15 minutes.
- Consumer reports rows that do not match the make or are missing since the watermark.
- Service log shows elapsed time above 3 seconds for the 5000-row case more than once.
- Any change in behavior of existing `Sales` operations (should not happen; the WSDL diff is additive).

**Rollback, in order of speed:**

1. Set `Routes:FulfilledByMake:Enabled = false`. Configuration only, no deploy. The route answers 404.
2. Redeploy the previous router build. The route disappears.
3. Redeploy the previous on-prem build. The WSDL loses the new operation; no existing consumer is affected because contracts were additive.
4. Optional: run `0152_down.sql` to drop the proc and the function. They are harmless if left in place.

**Roll forward:** a bad row rule is fixed by a new proc version through a new migration (`0153`) and an on-prem redeploy; the router does not change.

## Observability / flag decision

- **Flag:** yes, route-level switch `Routes:FulfilledByMake:Enabled`. Reason: the service deploys first and the consumer smoke must pass before exposure; the switch is the fastest rollback.
- **Logs:** service logs operation name, row count and elapsed time (no addresses). Router logs status code and the fault code.
- **Metrics and alerts:** router request count and latency for the route; alert on 5xx above 2% for 15 minutes; dashboard panel for `TooManyRows` count.
- **Rollout:** Dev (done) → Staging (consumer smoke) → Production with the switch off → switch on → watch the first week.

## Known risks and limitations

- Cap of 5000 rows. A consumer that has not synced for a long time must retry with a later watermark.
- Daily sync is assumed; there is no paging.
- Header `Status` is not used. Data quality problems in it are out of scope.
- Whole-word match treats every non-alphanumeric character as a separator; makes with unusual punctuation may need review.
- `TooManyRows` is HTTP 422, not 400.
- The service log writes the `Make` value (a product name).
- The one flaky tSQLt run came from the shared QA database, not from this change.
- Timing was measured on the dev copy only.
