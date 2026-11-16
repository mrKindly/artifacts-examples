# FS.OnPrem.Soap + FS.Cloud.Router — AI Use Case Playbook

**Measurements (source of truth):** [`fs-SDLC.json`](fs-SDLC.json)

> Demo example: the measurements in the JSON are synthetic, modeled on the FS07-3521 integration story shape. Every number below traces to that file. An empty metric in the JSON is "not measured" here, never a guess.

**Ratings:** **H** = High · **M** = Medium · **L** = Low — applied to **Impact · Risk · Effort**.

---

## How to read this

1. **Measurements** — the physical feedback path of an integration ticket (laptop → PR → main → Integration QA → router pipeline → Staging → Production) and what the last 30-day window measured (`states[]` in the JSON).
2. **Findings** — what is *true* about the path today (diagnosis, coded to one station).
3. **Use cases** — before → after, plus how we would *prove* it (prescription).

> **Findings ≠ use cases.** A Finding is what is true. A use case is the change and the evidence that it worked. A "healthy" Finding has no use case on purpose.

---

## 1. Findings

| # | Finding (short) | Station (from JSON `states[]`) | Evidence (metric from JSON) | Use case |
|---|-----------------|--------------------------------|-----------------------------|----------|
| **F1** | Integration stories need a second spec round | Developer's laptop | `Story analysis`: negativeFeedbackFrequency 60% (3 of 5 tickets), precision 40, flagged | **UC-1** |
| **F2** | Human review is slow and often takes two rounds | PR | `Code review (human)`: processingTime 79,200 s (22 h median, n=17), 9 of 17 PRs with 2+ rounds (nff 53%), flagged | **UC-2** |
| **F3** | The shared QA database makes the tSQLt suite flaky | Integration QA | `tSQLt suite on shared FsCore_QA`: 7 of 20 runs failed (nff 35%), median 840 s, precision 60, flagged | **UC-3** |
| **F4** | Router/WSDL contract drift is found by hand | Cloud router pipeline | `Contract drift / Swagger completeness review`: 1,500 s manual, 4 of 12 router PRs (nff 33%), flagged | **UC-4** |
| **F5** | Rollback section is missing from most release notes | Production | `Release notes and rollback section authoring`: 6 of 10 releases without rollback (nff 60%), precision 50, flagged | **UC-5** |
| **F6** | Post-merge CI is healthy — leave alone | main branch | `Post-merge CI`: 600 s, 1 of 24 runs failed (nff 4%), precision 95 | none (leave alone) |

### F1 — Integration stories need a second spec round

- **Measured:** 3 of the last 5 integration tickets under FS07-3442 needed a second spec round after a PM walkthrough: negativeFeedbackFrequency 60%, precision 40. Time spent on story analysis: **not measured**.
- **Means:** rework is found after a first prototype exists, when it is most expensive. FS07-3521 is an instance: wrong date basis, header-only status, wrong equipment join.
- **Coded on:** Developer's laptop · **→ UC-1**

### F2 — Human review is slow and often takes two rounds

- **Measured:** median time to first review decision 79,200 s (22 h) over 17 PRs; 9 of 17 (53%) needed two or more review rounds. PR validation build: 540 s median, 2 of 17 failed (12%).
- **Means:** most of the 22 h is waiting for a human, which an agent cannot remove. What an agent can reduce is the second round: mismatches with the spec and conventions that a reviewer finds on the first read.
- **Coded on:** PR · **→ UC-2**

### F3 — The shared QA database makes the tSQLt suite flaky

- **Measured:** 20 runs, 7 failed (35%), median 840 s. Dominant failure mode: fixtures without the `tsqlt_` prefix collide with a concurrent run on `FsCore_QA`. A re-run usually passes.
- **Means:** developers learn to re-run instead of reading failures; a real regression can hide behind a "flaky" label.
- **Coded on:** Integration QA · **→ UC-3**

### F4 — Router/WSDL contract drift is found by hand

- **Measured:** a reviewer compares the WSDL snapshot and the OpenAPI document by hand, median 1,500 s (25 min); 4 of 12 router PRs (33%) got a contract-drift comment.
- **Means:** a deterministic comparison is done by people and still misses a third of the time.
- **Coded on:** Cloud router pipeline · **→ UC-4**

### F5 — Rollback section is missing from most release notes

- **Measured:** 6 of the last 10 releases (60%) shipped without a rollback section. Authoring takes a median 1,800 s (30 min), precision 50.
- **Means:** the person on call finds the rollback steps during the incident. This is a documentation and process gap, not a code gap.
- **Coded on:** Production · **→ UC-5**

### F6 — Post-merge CI is healthy — leave alone

- **Measured:** 600 s median, 1 of 24 runs failed (4%), precision 95. Local build + scoped NUnit (3.2 min) and production deploys (13 min, 1 of 20 re-run) are also low-failure.
- **Means:** nothing to fix. Not a candidate for AI. Listed so nobody "improves" it.
- **Coded on:** main branch · **no use case**

---

## 2. Use case descriptions

### Use case index

| ID | Title | Finding(s) | Station / where | Impact | Risk | Effort |
|----|-------|-----------|-----------------|--------|------|--------|
| **UC-1** | Spec critique loop before the first prototype | F1 | Developer's laptop | H | L | L |
| **UC-2** | Pre-PR review against spec ACs and repo rules | F2 | PR | M | L | L |
| **UC-3** | Isolated per-run tSQLt database | F3 | Integration QA | H | L | M |
| **UC-4** | Deterministic WSDL↔OpenAPI contract check (AI builds it, then leaves) | F4 | Cloud router pipeline | M | L | M |
| **UC-5** | Release notes and rollback drafter with a lint gate | F5 | Production | M | L | L |

### UC-1 — Spec critique loop before the first prototype

- **Where:** Developer's laptop (activity `Story analysis / spec for integration tickets`)
- **Finding:** F1
- **Title means:** for every integration ticket, a spec is written from Jira (read through the read-only MCP), reviewed by an independent read-only subagent, and shown to the PM before any code is written.
- **Problem:** 60% of sampled integration tickets (3 of 5) needed a second spec round; precision of the activity is 40.

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — `/spec` skill (interactive questions) → `spec-reviewer` subagent (read-only, fresh context, fixed PASS/FAIL output, up to 3 iterations). Mandatory human gates: the developer answers the questions with the PM, and the developer accepts the final spec.
2. **Station / process step** — Developer's laptop. Adds a "spec critique" step in front of `Local coding / implementation`; replaces the second spec round that today happens after a prototype.
3. **Who · when · trigger** — developer, at story start, triggered by picking up a ticket that touches a consumer contract.
4. **Value-stream effect** — fewer rework loops after the prototype; second-spec-round rate falls from 60%.

```text
ticket picked up → /spec (Atlassian MCP read) → spec-reviewer (PASS/FAIL) → human + PM sign-off → coding · on station Developer's laptop
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | Ticket text and comments, linked tickets, `INFORMATION_SCHEMA` of `FsCore_Dev`, the operation closest to the new one |
| Warm | `AGENTS.md`, `sql-server-conventions.mdc`, `soap-contracts.mdc`, spec template |
| Cold | Unrelated modules, old prototypes, `CONTRIBUTING.md` |

**Verification approach:**

- Every spec AC maps to a test name before coding starts (checked by the plan's Verification table).
- Metric: second-spec-round rate from 3 of 5 (60%, baseline in the JSON) to at most 1 of 5 (20%) over the next 5 integration tickets.
- Human gate: PM sign-off on the definitions table stays mandatory.

**Ratings:**

- Impact **H** (rework found before code; largest nff in the map)
- Risk **L** (read-only agent; Jira read-only)
- Effort **L** (skill, subagent and MCP already exist: Artifacts 1 and 2)

### UC-2 — Pre-PR review against spec ACs and repo rules

- **Where:** PR (activity `Code review (human)`, via the optional `Optional agent pre-review`)
- **Finding:** F2
- **Title means:** before a PR is opened, a read-only agent maps each spec AC to a test in the diff and checks the diff against the two rules, and posts the result as the PR description, so the human reviewer starts with the mismatches already visible.
- **Problem:** 9 of 17 PRs (53%) needed two or more review rounds; median wait 79,200 s (22 h) is human capacity and is **not** the target.

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — read-only diff advisory: (a) AC-to-test mapping against the UC-1 spec, (b) scope check against the plan's file list, (c) rule check (additive contracts, SQL conventions). Output is advisory text; it never approves.
2. **Station / process step** — PR. Adds the currently unused `Optional agent pre-review` activity before `Code review (human)`; shortens the second round, not the first wait.
3. **Who · when · trigger** — developer, immediately before opening the PR; on-demand skill call.
4. **Value-stream effect** — fewer 2+-round PRs; reviewer time per PR shifts from mismatch hunting to design judgement.

```text
branch ready → pre-review skill → advisory report in PR description → human review · on station PR
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | Branch diff, the spec and plan of the ticket |
| Warm | The two rules, `AGENTS.md`, test naming convention |
| Cold | Other PRs, unrelated modules |

**Verification approach:**

- Each AC listed in the report points to a real test name (spot-checked by the reviewer on the first 5 PRs).
- Metric: PRs with 2+ review rounds from 9 of 17 (53%) to at most 6 of 17 (35%) over the next 17 PRs. Median time to first decision (79,200 s) is tracked but not a success criterion.
- Human gate: the reviewer approves. The agent has no approve or merge tool.

**Ratings:**

- Impact **M** (reduces rework, not waiting)
- Risk **L** (advisory only)
- Effort **L** (reuses the subagent pattern from Artifact 2; depends on UC-1 for specs)

### UC-3 — Isolated per-run tSQLt database

- **Where:** Integration QA (activity `tSQLt suite on shared FsCore_QA`)
- **Finding:** F3
- **Title means:** each CI run gets its own database restored from a template backup, instead of using the shared `FsCore_QA`; an agent writes the provisioning step and fixes the old tests that depend on shared state.
- **Problem:** 7 of 20 runs failed (35%), median 840 s, caused by fixture collisions.

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — chained skills with human gates: (a) `find-shared-state-tests` lists tSQLt tests that write outside a rolled-back transaction or without the `tsqlt_` prefix; (b) `provision-test-db` writes a pipeline step that restores the template backup to `FsCore_QA_<runId>` and drops it afterwards; (c) the developer reviews both diffs. The CI step is plain script after that.
2. **Station / process step** — Integration QA. Replaces the shared database in `tSQLt suite on shared FsCore_QA` with a per-run database; removes the re-run habit.
3. **Who · when · trigger** — a developer builds it once as a task; afterwards CI runs it on every main-branch package.
4. **Value-stream effect** — failures become real failures; fail rate falls from 35%.

```text
task start → find-shared-state-tests → provision-test-db → human review → merge · on station Integration QA
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | Existing tSQLt tests, `run-tsqlt.ps1`, the pipeline YAML for Integration QA, template backup location |
| Warm | `sql-server-conventions.mdc` (`tsqlt_` prefix), storage and permission limits of the QA server |
| Cold | Application code, router repo |

**Verification approach:**

- 20 consecutive runs on the new setup with no collision failures.
- Metric: tSQLt fail rate from 7 of 20 (35%) to at most 2 of 20 (10%); median duration stays near 840 s (no more than +20%).
- Human gate: a DBA reviews the restore and drop permissions before the first run.

**Ratings:**

- Impact **H** (largest measured failure rate)
- Risk **L** (test infrastructure only; a bad script fails the run, not production)
- Effort **M** (new pipeline step, test cleanup; the skills are new)

### UC-4 — Deterministic WSDL↔OpenAPI contract check (AI builds it, then leaves)

- **Where:** Cloud router pipeline (activity `Contract drift / Swagger completeness review (manual)`)
- **Finding:** F4
- **Title means:** a script compares operations, fields, types and fault codes between the approved WSDL snapshot and the router OpenAPI document and fails the build on drift. An agent writes and tests the script once; the pipeline never calls an agent.
- **Problem:** 25 min of manual comparison per router PR and 4 of 12 PRs (33%) still received a drift comment.

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — one-off build task: agent drafts the comparer from the last 12 router PRs (drift cases as test fixtures), a human reviews it. After it is accepted, the AI is removed from the loop: CI runs a plain script. An optional agent step may summarise a failing diff in the PR comment, but is not required.
2. **Station / process step** — Cloud router pipeline. Replaces the manual review activity with an automated build step; the human reviews only intentional changes.
3. **Who · when · trigger** — CI, on every router PR.
4. **Value-stream effect** — 1,500 s of manual comparison removed per PR; drift comments fall from 33%.

```text
router PR opened → CI runs contract-diff script → fail on drift → fix · on station Cloud router pipeline
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | `service.wsdl` snapshot, generated OpenAPI document, the 4 PRs that received drift comments |
| Warm | Mapping rules between SOAP faults and HTTP codes, naming conventions |
| Cold | Business logic, on-prem proc code |

**Verification approach:**

- The script fails on each of the 4 historical drift PRs (fixtures) and passes on the 8 others.
- Metric: drift comments from 4 of 12 (33%) to 0 of the next 12 router PRs; manual review minutes from 25 to 0 for unchanged contracts.
- Human gate: advisory mode on the first 5 PRs before the build is allowed to fail (see sequencing).

**Ratings:**

- Impact **M** (one in three router PRs, 25 min each)
- Risk **L** (build-time check; a false positive blocks a PR, which is visible and fixable)
- Effort **M** (script, fixtures, pipeline step)

### UC-5 — Release notes and rollback drafter with a lint gate

- **Where:** Production (activity `Release notes and rollback section authoring (manual)`)
- **Finding:** F5
- **Title means:** a skill drafts release notes and a rollback section from the spec, the plan and the PR diffs, and a lint check blocks a release whose notes have no rollback section.
- **Problem:** 6 of 10 releases (60%) had no rollback section; precision of the activity is 50. This is documentation, not code.

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — single skill with a human gate: inputs are spec, plan, diffs and the config switches; output is a draft with triggers, ordered rollback steps and the flag decision. The release owner edits and approves. A one-line lint (heading present, steps non-empty) is plain script.
2. **Station / process step** — Production. Shortens `Release notes and rollback section authoring` from a blank page to a review; adds the lint as a gate before `Deploy on-prem service, then router`.
3. **Who · when · trigger** — release owner, before the deploy, triggered by an approved PR.
4. **Value-stream effect** — rollback is written before the incident; the missing-rollback rate falls from 60%.

```text
approved PR → release-notes skill → release owner edits → lint gate → deploy · on station Production
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | Spec, plan, PR diffs, configuration switches, previous release notes |
| Warm | Release-notes template, alert thresholds |
| Cold | Unrelated services |

**Verification approach:**

- Every draft is checked by the release owner against the diff: each rollback step must name a real setting, pipeline or script.
- Metric: releases without a rollback section from 6 of 10 (60%) to 0 of the next 10.
- Human gate: release owner approval is mandatory; the lint blocks, it does not approve.

**Ratings:**

- Impact **M** (reduces incident risk, not cycle time)
- Risk **L** (documentation; wrong text is caught by owner review)
- Effort **L** (one skill and a lint script)

---

## 3. Prioritisation

### Ranked backlog

| P | Do | UC | Rationale (Impact · Risk · Effort) | Done when… |
|---|----|----|-----------------------------------|------------|
| P0 | Make the spec critique loop the default for integration tickets | UC-1 | H · L · L | Next 5 integration tickets: at most 1 needs a second spec round (baseline 3 of 5) |
| P0 | Add the release-notes skill and the rollback lint | UC-5 | M · L · L | Next 10 releases: 0 without a rollback section (baseline 6 of 10) |
| P1 | Pre-PR review advisory | UC-2 | M · L · L | Next 17 PRs: at most 6 with 2+ review rounds (baseline 9 of 17). **Requires UC-1 first** (it compares the diff to a spec) |
| P1 | Per-run isolated tSQLt database | UC-3 | H · L · M | 20 consecutive runs, at most 2 failures (baseline 7 of 20) |
| P2 | Build the contract-diff script and run it advisory | UC-4a | M · L · M | Script fails on all 4 historical drift PRs and passes on the other 8 |
| P2 | Make the contract-diff script a failing build step | UC-4b | M · L · L | 5 advisory PRs with no false positives, then the step blocks; 0 drift comments in the next 12 router PRs. **Requires UC-4a** |

**Dependencies, explicit:**

- UC-2 needs the specs UC-1 produces. Do not enable the pre-PR review as an expected step before UC-1 is in use.
- Do not make the tSQLt suite a required PR check until UC-3 is done; at the current 35% failure rate it would freeze merges.
- UC-4b (blocking) waits for UC-4a and 5 clean advisory runs.
- F6 (post-merge CI) is deliberately not in the backlog.

---

## Traceability checklist

- [x] Every Finding cites a metric that exists in the JSON (no invented numbers).
- [x] Every Finding maps to exactly one station. F1-F5 map to a use case; F6 is a "healthy / leave alone" Finding by design.
- [x] Every use case has Impact · Risk · Effort and a verifiable "Done when".
- [x] Prioritisation respects dependencies (UC-1 before UC-2; UC-3 before any required tSQLt gate; UC-4a before UC-4b).
- [x] Unmeasured items are labelled "not measured" (story analysis time, local coding time, pre-review usage).
