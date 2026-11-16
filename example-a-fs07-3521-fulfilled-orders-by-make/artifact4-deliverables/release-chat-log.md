# Release readiness for FS07-3521
_Exported on 10/30/2026 at 16:03:12 GMT+3 from Cursor (3.26.2)_

---

**User**

Prepare release readiness for FS07-3521. Inputs: spec, plan, diff of both PRs, `verification.md`. I need release notes, risk control with rollback and roll-forward, the flag and observability decision, and known risks.

---

**Cursor**

I'll read the inputs first. A few things are not in them, so questions before I propose anything:

1. Do the service and the router deploy independently?
2. Can the router config be changed without a deploy?
3. Is there an existing alert on router 5xx per route?
4. Does the proc need a down script?

---

**User**

1. Yes, separate pipelines. Service first.
2. Yes, `appsettings` override through the environment configuration.
3. Yes, per route group, 2% over 15 minutes.
4. Provide one, but it is optional.

---

**Cursor**

Proposed decisions:

- **Flag:** a route switch, off in Production, because the consumer smoke must pass before exposure and the switch is the fastest rollback.
- **Rollback order:** switch off, then the previous router build, then the previous service build, then the optional down script.
- **No feature flag in the service:** the new operation is additive and unused until the route is on.
- **Triggers:** 5xx over 2% for 15 minutes, wrong rows reported, repeated elapsed time over 3 seconds, any change in existing operations.

---

**User**

Add the `TooManyRows` count to the dashboard, and write the known limitation about the 5000-row cap for the consumer owner. Also note that the log line carries the make value.

---

**Cursor**

Added both, plus the note that the log line carries the make value. Wrote `release.md`.

---

**User**

Approved. Enabling the route in Production waits for the Staging smoke by the consumer owner.

---

**Cursor**

Recorded in `release.md`: the route stays off in Production until the consumer owner's Staging smoke passes.
