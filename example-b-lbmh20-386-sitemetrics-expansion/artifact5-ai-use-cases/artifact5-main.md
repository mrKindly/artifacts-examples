# LBMH.Services.Portal AI Use Case Playbook

> Demo example: synthetic evidence (ids, run numbers, timings in chat logs/transcripts) modeled on the LBMH20-386 story shape. Artifact 5 numbers come from the real portal-SDLC.json.

**Measurements (source of truth):** [`portal-SDLC.json`](portal-SDLC.json)

> Every number in this doc traces back to the JSON (window 2026-06-16 to 2026-07-16, about 30 days, adequacy score 93; ADO runs for definitions 440 / 441 / 445 / 472, 24 completed PRs to `main`, local workstation sample of 2026-07-15). A metric the JSON leaves empty is written "not measured" here, never guessed. Targets in "Done when" are targets, not measurements.

**Ratings:** **H** = High · **M** = Medium · **L** = Low — applied to **Impact · Risk · Effort**.

---

## How to read this

1. **Measurements** — the physical feedback path and what the last window measured (JSON `states[]`):

   | Station (`states[].name`) | What was measured |
   |---------------------------|-------------------|
   | Developer's laptop | `dotnet build Api` 15.4 s, scoped `Data.Tests` 4.3 s, `npm run build` 27 s, inner loop 46 s; scoped-test activity flagged |
   | PR | Code review processing time 10177 s (about 2.8 h, n=8, excluding PRs open under 5 min); required-checks activity flagged, 0 s |
   | main branch | Post-merge CI (`build.yaml`) flagged, 0 s; all transitions to deploys are Manual |
   | QualityCheck (manual) | 445: 15 runs, 80% fail, median 669 s, flagged |
   | App deploy (472) | 33 of 33 succeeded, median 74 s |
   | Api deploy (440) | 45 of 45 succeeded, median 616 s |
   | Infrastructure deploy (441) | 17 runs, 17.6% fail, median 585 s, flagged |
   | Dev environment / Production environment | AWS CFN median about 293 s / about 295 s, 0% fail; post-deploy smoke not measured |

2. **Findings** — what is *true* about the path today (diagnosis, each coded to one station).
3. **Use cases** — before → after and how we would *prove* it (prescription).

> **Findings ≠ use cases.** A Finding = what's true. A Use case = the change + the evidence that it worked.

The JSON also carries older `asIsFindings` / `recommendedUseCases` strings with their own F/UC numbering. This playbook does not reuse those ids; it re-derives findings from the measured fields.

**Seeds from this demo set.** The `pre-pr-review` skill ([artifact 1](../artifact1-tools/skills/pre-pr-review/SKILL.md)), the `diff-vs-spec-reviewer` subagent ([artifact 2](../artifact2-mcp-subagents/agents/diff-vs-spec-reviewer.md)), the `portal-test-coverage` rule and the story evidence from LBMH20-386 ([artifact 3](../artifact3-workflow/artifact3-main.md), [artifact 4](../artifact4-deliverables/artifact4-main.md)) already exist and are the starting point for UC-1, UC-2, UC-3 and UC-6.

---

## 1. Findings

| # | Finding (short) | Station (from JSON `states[]`) | Evidence (metric from JSON) | Use case |
|---|-----------------|--------------------------------|-----------------------------|----------|
| **F1** | The coverage gate is red almost every time it runs, so red carries no signal | QualityCheck (manual) | 445: 12 of 15 failed (80%); dominant = Coverlet line <90%; build 29300: 488 passed, 87.72% line | **UC-1** |
| **F2** | The PR is a voluntary gate: no required checks, and most PRs merge with no review time | PR | Branch policies on `main`: none (empty list); required-checks activity flagged, 0 s; 16 of 24 PRs open <5 min; reviewed PRs median ~2.8 h (n=8) | **UC-2**, **UC-3** |
| **F3** | Merge to `main` deploys nothing; promotion depends on a human remembering | main branch | Post-merge CI (`build.yaml`) flagged, precision 40, 0 s; all deploy transitions Manual (Manual reason on all sampled runs) | **UC-4** |
| **F4** | Infrastructure deploys fail more than any other deploy, mostly before CloudFormation | Infrastructure deploy (441) | 441: 3 of 17 failed (17.6%); Dev 3 of 13 (23.1%); example "ChangeSet does not exist" (28327) | **UC-5** |
| **F5** | The full test suite cannot be trusted as a signal; only scoped runs are | Developer's laptop | Scoped `dotnet test` activity flagged, precision 70 (build and web steps 80); full solution / `npm run test:ci` can exit 1 (pre-existing) | **UC-6** |
| **F6** | Healthy: the deploy mechanism itself works and is fast | Api deploy (440) | 440: 45 of 45 succeeded (0%), median 616 s; sibling App deploy (472): 33 of 33, median 74 s | none (leave alone; reused unchanged by UC-4) |

### F1 — The coverage gate is red almost every time

- **Measured:** QualityCheck 445 had 15 runs in the window: 3 succeeded, 12 failed (80%), median 669 s, `negativeFeedbackFrequency` 80, activity flagged. Dominant failure mode: Coverlet line coverage below 90% on `Api.Tests` while tests pass (build 29300: 488 passed, 87.72% line). Activity precision 90.
- **Means:** The gate measures something real (precision 90) but has been red so long that red is normal. Nobody can tell whether a change made it worse, and the gap to the bar is 2.28 percentage points. 445 is also manual and not on the CD path, so a red run blocks nothing.
- **Coded on:** QualityCheck (manual) · **→ UC-1**

### F2 — The PR is a voluntary gate

- **Measured:** `az repos policy list` for `main` returned an empty list; "ADO build-validation / required status checks" is flagged (precision 100, processing time 0). Of 24 PRs completed to `main`, 16 were open under 5 minutes (all-PR median ~63 s, average ~10.4 h, p90 ~42 h, 3 open over a day). For the 8 PRs open 5 minutes or more, median open time is 10177 s (~2.8 h; average ~31.3 h). The non-owner path (n=6) has median ~23.1 h; one owner accounts for 75% of the PRs (`bkohnSharePercent`). Use of the optional `/pre-pr-review` gate is not measured.
- **Means:** Review is quick when it happens, but two thirds of the PRs carry no review time and nothing automatic checks any PR. All quality signal is optional, so it depends on who authors the PR.
- **Coded on:** PR · **→ UC-2, UC-3**

### F3 — Merge deploys nothing

- **Measured:** `build.yaml` has `trigger: main` but no active ADO definition uses it (flagged, precision 40, processing time 0). Merge does not queue 440 / 441 / 445 / 472; every transition from `main branch` to a deploy is Manual and the "Manual" reason appears on all sampled deploy and QualityCheck runs. The JSON lists "forgotten post-merge deploy leaves AWS behind main" as a failure mode; how often it happens is not measured.
- **Means:** What is merged and what runs in AWS can differ, and the only protection is human memory. The cost is invisible in the metrics because no automatic post-merge step exists to measure.
- **Coded on:** main branch · **→ UC-4**

### F4 — Infrastructure deploys fail before CloudFormation

- **Measured:** 441 had 17 runs in the window: 14 succeeded, 3 failed (17.6%; Dev 3 of 13 = 23.1%, Prod 0 of 4, thin), median 585 s, activity flagged, precision 75. Example failure: CDK "ChangeSet does not exist" (28327). AWS CloudFormation updates themselves show 0% failure in both accounts (Dev 32 of 32, Prod 22 of 22).
- **Means:** The failures are on the pipeline side (before or around the change set), not infrastructure changes that went wrong. Today each failure needs a human to read logs and decide whether to re-run or fix the CDK code. Time to diagnose is not measured.
- **Coded on:** Infrastructure deploy (441) · **→ UC-5**

### F5 — Only scoped test runs are trustworthy

- **Measured:** On the laptop, `dotnet build Api` took 15.4 s and `npm run build` 27 s (both exit 0, precision 80); scoped `Data.Tests` took 4.3 s (precision 70, flagged). The JSON notes the full solution and `npm run test:ci` can exit 1 for pre-existing reasons. Inner loop total is 46 s. Duration of the full suite is not measured.
- **Means:** The inner loop is short enough for fast feedback, but the "run everything" exit code is not a usable verdict, so every developer decides alone which subset counts, and coverage on changed lines is computed by hand.
- **Coded on:** Developer's laptop · **→ UC-6**

### F6 — Healthy: leave the deploy mechanism alone

- **Measured:** Api deploy 440: 45 of 45 succeeded (Dev 26, Prod 19), median 616 s (Dev 608 s, Prod 628 s), precision 95. Sibling App deploy 472: 33 of 33 succeeded (Dev 19, Prod 14), median 74 s. AWS CloudFormation phase median about 293 s (Dev) and about 295 s (Prod), 0% failure.
- **Means:** Once a deploy is started it works. The problem in this path is *whether and when* it is started (F3), not how it runs. Nobody should rewrite 440 or 472.
- **Coded on:** Api deploy (440) · **→ none; UC-4 reuses 440 and 472 unchanged**

---

## 2. Use case descriptions

### Use case index

| ID | Title | Finding(s) | Station / where | Impact | Risk | Effort |
|----|-------|-----------|-----------------|--------|------|--------|
| **UC-1** | Raise `Api.Tests` line coverage to the 90% gate with AI-written tests | F1 | QualityCheck (manual) | H | L | M |
| **UC-2** | Pre-PR evidence package: `pre-pr-review` skill + read-only `diff-vs-spec-reviewer` | F2 | PR | M | L | L |
| **UC-3** | Required PR build validation (advisory first, then required) | F2 | PR | H | M | M |
| **UC-4** | Auto-queue Dev deploy (472 / 440) on merge; Prod stays manual | F3 | main branch | H | M | M |
| **UC-5** | Read-only failure triage for Infrastructure 441 runs | F4 | Infrastructure deploy (441) | M | L | L |
| **UC-6** | Deterministic scoped-verify tool built with AI, then AI removed | F5 | Developer's laptop | M | L | L |

### UC-1 — Raise `Api.Tests` line coverage to the 90% gate with AI-written tests

- **Where:** QualityCheck (manual)
- **Finding:** F1
- **Title means:** Use an agent to write behaviour-asserting tests, in small PRs, for the least-covered `Api` code until QualityCheck 445 reports 90% line coverage or more. No production code changes.
- **Problem:** 445 fails 12 of 15 runs (80%) with Coverlet line coverage below 90% while tests pass; baseline 87.72% on build 29300 (488 passed).

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — Deterministic gap list first (Coverlet cobertura report sorted by uncovered lines per file; no AI). A test-writing agent takes one file or folder per PR, follows `portal-test-coverage.mdc` (naming, four edge classes), writes tests that fail when behaviour is broken, and never edits production code. Human gate: the author reviews every test for real assertions; reviewer checks no test is skipped, deleted or assertion-free.
2. **Station / process step** — QualityCheck (manual), activity "Build-Test-Analyze". It does not add a step; it removes the dominant cause of its failures.
3. **Who · when · trigger** — A developer, in small slices alongside normal work, starting on demand from the gap list; each slice re-queues 445 by hand (until UC-3 provides a faster PR signal).
4. **Value-stream effect** — Dominant failure mode (Coverlet <90%) disappears; 445 failure rate falls from 80%; red regains meaning, which is the precondition for making any coverage check a gate (UC-3 Phase B).

```text
445 cobertura report → gap list (script) → test-writer agent (one folder) → human test review → PR → 445 re-run · on station QualityCheck (manual)
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | Coverlet report of the latest 445 run (build 29300 as baseline); the files named in the gap list and their existing tests |
| Warm | `portal-test-coverage.mdc`; `AGENTS.md` (commands, gotchas); examples of the mapper tests from LBMH20-386 ([artifact 3](../artifact3-workflow/verification.md)) |
| Cold | Sonar / Mend findings; Web coverage; CDK code (excluded unless the gate covers it) |

**Verification approach:**

- Every new test has at least one assertion on behaviour; reviewer spot-checks by breaking the code under test locally (the test must fail).
- Metric: `Api.Tests` line coverage from 87.72% (build 29300) to at least 90.00% on a 445 run; fail rate 80% (12 of 15) to a target of at most 20% over the next 30-day window.
- Human sign-off stays mandatory on every test PR (code review).

**Ratings:**

- Impact **H** — unblocks the gate and every later coverage check (UC-3 Phase B).
- Risk **L** — tests only, no production change; low-value tests are the risk, mitigated by review.
- Effort **M** — gap is 2.28 percentage points; the number of tests needed is not measured; the coverage rule and a worked example exist.

### UC-2 — Pre-PR evidence package: `pre-pr-review` skill + read-only `diff-vs-spec-reviewer`

- **Where:** PR
- **Finding:** F2
- **Title means:** Before a PR is marked Ready, the author runs one skill that maps each spec AC to test evidence, runs scoped checks, and calls a read-only subagent to compare diff to spec; the output file goes into the PR.
- **Problem:** 16 of 24 PRs were open under 5 minutes and there are no required checks, so reviewers often get no evidence; reviewed PRs take ~2.8 h median (n=8) and the non-owner path ~23.1 h (n=6).

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — Skill orchestrator with a read-only verifier subagent ([artifact 1](../artifact1-tools/skills/pre-pr-review/SKILL.md), [artifact 2](../artifact2-mcp-subagents/agents/diff-vs-spec-reviewer.md)). Hard rules: never fix, push, merge, approve or deploy. Human gates: the author reads and fixes after a HOLD; the reviewer reads the report; merge stays human.
2. **Station / process step** — PR: turns the optional "/pre-pr-review agent gate" into the standard first step before "Code review (human)". It shifts review effort from re-deriving evidence to judging it.
3. **Who · when · trigger** — The author, when the branch is feature-complete (re-run after fixes); the reviewer may ask for it when the PR has no report.
4. **Value-stream effect** — Fewer PRs reach review without evidence (16 of 24 today had no review time); reviewer time per PR should not grow and the non-owner median (~23.1 h) should shrink. Effect on rework after review is not measured.

```text
branch ready → /pre-pr-review → diff-vs-spec-reviewer (read-only) → pre-pr-review.md (PASS/HOLD) → author fixes or opens PR → human review · on station PR
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | Spec with numbered ACs; branch diff vs `main`; scoped test and coverage output |
| Warm | `AGENTS.md`; the two rules; prior `pre-pr-review.md` reports ([log](../artifact1-tools/skills/pre-pr-review/log.md)) |
| Cold | Jira beyond the story key (read-only if needed); pipeline YAML; production data |

**Verification approach:**

- Each AC in the spec maps to a test name or a manual step in the report; an AC without evidence forces HOLD.
- Metric: share of PRs open 5 minutes or more that carry `pre-pr-review.md` (baseline not measured, optional today) to a target of 100%; reviewed-PR median not above 10177 s (~2.8 h); non-owner median below 83244 s (~23.1 h).
- Mandatory human gates: code review and merge.

**Ratings:**

- Impact **M** — improves evidence for each PR, does not change what runs after merge.
- Risk **L** — advisory and read-only; the main risk is false confidence, mitigated by "no spec-vs-diff findings" wording and human review.
- Effort **L** — skill, subagent, rules and a worked run on LBMH20-386 already exist.

### UC-3 — Required PR build validation (advisory first, then required)

- **Where:** PR
- **Finding:** F2
- **Title means:** An ADO build-validation policy on `main` that runs a fast PR pipeline (build, scoped tests, changed-code coverage at least 90%, no new warnings), introduced first as non-blocking and later as required.
- **Problem:** No branch policies on `main` (empty list); the required-checks activity is flagged with 0 s processing time; QualityCheck 445 (median 669 s, 80% fail) is not a PR gate and is too slow and too red to become one.

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — "AI drafts, CI enforces": an agent drafts the PR-validation pipeline YAML and the policy change as a PR; the checks themselves are deterministic (UC-6 tool) with no AI at gate time. Human gates: platform owner reviews the YAML and is the only one who enables the policy; phase B needs a human go/no-go.
2. **Station / process step** — PR: replaces the empty "ADO build-validation / required status checks" activity with a real one.
3. **Who · when · trigger** — Triggered by PR open and update; set up once by the platform owner. Phase A: policy marked optional. Phase B: required.
4. **Value-stream effect** — Failures surface before merge instead of in 445 afterwards; every PR gets the same minimum check regardless of author (F2). Gate duration must stay near the inner loop; its measured duration is a Phase A output.

```text
PR open/update → PR pipeline (build, scoped tests, changed-code coverage, warnings) → status on PR → human review and merge · on station PR
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | Existing pipeline YAMLs and templates (`QualityCheck.yaml`, `build.yaml` orphan); UC-6 tool; Phase A results |
| Warm | `portal-test-coverage.mdc`; `portal-pipelines.mdc`; ADO policy documentation |
| Cold | Sonar / Mend gating (later); deploy pipelines 440 / 441 / 472 (not touched) |

**Verification approach:**

- Phase A: `az repos policy list` for `main` goes from `[]` to one non-required build-validation policy; every PR shows the status; false-fail causes are listed.
- Phase B: policy becomes required only when its prerequisite is met (UC-1 done, UC-6 in use) and Phase A shows stable results; first 10 PRs fail only for real reasons.
- Human gate: policy enablement is a human action.

**Ratings:**

- Impact **H** — first automatic check on every PR.
- Risk **M** — enabling it too early freezes merges; mitigated by Phase A and the UC-1 / UC-6 prerequisites.
- Effort **M** — needs a new pipeline and policy; the check logic comes from UC-6.

### UC-4 — Auto-queue Dev deploy (472 / 440) on merge; Prod stays manual

- **Where:** main branch
- **Finding:** F3
- **Title means:** After a merge to `main`, the pipelines for the changed package start automatically on Dev only; Production remains a manual run; Infrastructure 441 is never queued automatically.
- **Problem:** All transitions from `main` to deploys are Manual and `build.yaml` is an orphan (flagged, 0 s); "forgotten post-merge deploy leaves AWS behind main" (frequency not measured).

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — "AI drafts, rules decide": an agent drafts the pipeline trigger change and a path-to-package rule that already exists in the deploy skill (React only → 472; API only → 440; both → 472 then 440; CDK change → human runs 441), plus a post-deploy smoke checklist. The rule is deterministic YAML, not an agent call. Human gates: YAML change review; Production deploy and the 441 decision stay human.
2. **Station / process step** — main branch: replaces "Human decide deploy package" for Dev and gives the orphan `build.yaml` a role or removes it; Dev environment gets a post-deploy check; the Dev-to-Production transition stays Manual.
3. **Who · when · trigger** — Merge to `main` (Dev only); a human smokes Dev and then starts Production.
4. **Value-stream effect** — The Manual wait from merge to Dev disappears and Dev can no longer silently lag `main`; the healthy 472 / 440 pipelines (F6) run unchanged.

```text
merge to main → path-based package rule (YAML) → 472 and/or 440 on Dev → human Dev smoke → human starts Prod · on station main branch
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | `.cursor/skills/deploy/SKILL.md` (package rules, parameters); the two deploy YAMLs; `build.yaml` |
| Warm | `portal-pipelines.mdc`; the LBMH20-386 release notes ([artifact 4](../artifact4-deliverables/release.md)) for the smoke and rollback pattern |
| Cold | Infra 441 and CDK code (never auto); Production pipelines |

**Verification approach:**

- Metric: after the change, every merge touching Api or Web is followed by the matching Dev run (baseline today: none queued automatically). 472 and 440 keep their 0% fail rate (33 of 33 and 45 of 45) and median durations (about 74 s and about 616 s).
- No automatic 441 run and no automatic Production run (checked in the YAML and in the run list).
- Mandatory human gates: Dev smoke, Production start.

**Ratings:**

- Impact **H** — removes dependence on memory for Dev; every merge becomes observable in AWS.
- Risk **M** — more automatic deploys and a wrong package choice are possible; limited to Dev, deterministic rule, 441 excluded, rollback is a redeploy.
- Effort **M** — trigger and parameter wiring on two YAMLs; the package rule already exists in the deploy skill.

### UC-5 — Read-only failure triage for Infrastructure 441 runs

- **Where:** Infrastructure deploy (441)
- **Finding:** F4
- **Title means:** When a 441 run fails, a read-only agent reads the run logs, says which stage failed (change-set creation, CloudFormation, image push, other), compares with the AWS stack history, and proposes the human's next step. It does not re-run anything.
- **Problem:** 3 of 17 runs failed (17.6%; Dev 23.1%), e.g. "ChangeSet does not exist" (28327), while CloudFormation updates show 0% failure; diagnosis is manual and its duration is not measured.

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — Log-triage subagent, read-only (same safety shape as `diff-vs-spec-reviewer`): input is a failed run id; tools are ADO log read and AWS CloudFormation history read; output is a fixed-format diagnosis (stage, likely cause, recommended next step). Human gate: a human decides to re-run, fix the CDK code or escalate.
2. **Station / process step** — Infrastructure deploy (441): adds a diagnosis step after a failure; nothing is added to a green run.
3. **Who · when · trigger** — The developer who started the run, on a failing 441 (on demand first; an automatic post on failure can follow once trusted).
4. **Value-stream effect** — Separates pipeline-side noise from real infrastructure failures and shortens diagnosis; whether it reduces the 17.6% failure rate itself depends on what the diagnoses reveal.

```text
441 run fails → triage subagent (read-only: ADO logs + CFN history) → diagnosis (stage, cause, next step) → human decides · on station Infrastructure deploy (441)
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | Failed run id and its logs; AWS stack history for the same environment |
| Warm | Deploy skill (when 441 is used); the three failed runs from the window as examples (28327 among them); CDK stack layout |
| Cold | 440 and 472 runs (healthy); application code; Production runs unless the failure is in Production |

**Verification approach:**

- Replay the agent on the 3 failed runs of the window; each diagnosis is compared with the human's account (target: 3 of 3 classified to the same stage).
- Metric: time from failure to a written diagnosis (baseline not measured; record it for the first 5 failures, then set the target).
- Mandatory human gate: no re-run, fix or approval is done by the agent.

**Ratings:**

- Impact **M** — shortens a recurring manual diagnosis; does not prevent failures by itself.
- Risk **L** — read-only, advisory.
- Effort **L** — reuses the read-only agent pattern and the ADO / AWS read access already used for the measurements.

### UC-6 — Deterministic scoped-verify tool built with AI, then AI removed

- **Where:** Developer's laptop
- **Finding:** F5
- **Title means:** An agent writes, once, a plain script that turns a branch diff into a PASS/FAIL verdict from scoped build, scoped tests and changed-code coverage. After review the script runs without any AI.
- **Problem:** The scoped `dotnet test` activity is flagged (precision 70) because full-suite and `npm run test:ci` exits can be 1 for pre-existing reasons; every developer picks their own subset and computes changed-code coverage by hand.

**Agentic workflow + SDLC / VSM integration:**

1. **Agent pattern** — Build-then-remove-AI: an agent writes the tool test-first (fixture diffs in, expected verdicts out), a human reviews and owns it, and from then on it is plain code. Today `pre-pr-review` computes coverage by hand in its step 5; the tool replaces that step, and later UC-3 calls the same tool in CI. Human gates: review of the script and its tests; agents never edit it afterwards without review.
2. **Station / process step** — Developer's laptop: replaces the ad hoc "which tests do I run" decision and the manual coverage-on-changed-lines step with one command; keeps the 46 s inner loop.
3. **Who · when · trigger** — The developer before a commit or PR, the `pre-pr-review` skill, and later the PR pipeline (UC-3).
4. **Value-stream effect** — A usable local verdict independent of the pre-existing full-suite failures; the same deterministic number locally and in CI.

```text
git diff → changed paths → test projects/filters → build + scoped tests + coverage on changed lines → exit 0/1 + summary (no AI at run time) · on station Developer's laptop
```

**Required context**

| Tier | Sources |
|------|---------|
| Hot | Repo layout and test project names; `git diff` conventions; Coverlet cobertura and Vitest coverage formats |
| Warm | `portal-test-coverage.mdc`; `pre-pr-review` step 5 and its log ([log](../artifact1-tools/skills/pre-pr-review/log.md)); `AGENTS.md` verify section |
| Cold | Sonar / Mend; CDK tests; pipeline YAML |

**Verification approach:**

- The tool's own tests: fixture diffs produce the expected scoped selection and verdict; run it on the LBMH20-386 diff and compare its coverage figure with the 94.1% worked out by hand ([artifact 3](../artifact3-workflow/verification.md)).
- Metric: runtime for a story-sized change within about 2x of the 46 s inner loop (target at most 92 s); no use of the full-suite exit code as a verdict.
- Mandatory human gate: code review of the tool before first use.

**Ratings:**

- Impact **M** — better local signal and a prerequisite for UC-3.
- Risk **L** — local tool, no production effect; a wrong scope selection could hide a failing test, mitigated by fixture tests and review.
- Effort **L** — the rules, commands and a manual run already exist; the tool is a small script.

---

## 3. Prioritisation

### Ranked backlog

| P | Do | UC | Rationale (Impact · Risk · Effort) | Done when… |
|---|----|----|-----------------------------------|------------|
| P0 | Build the scoped-verify tool and use it in `pre-pr-review` | UC-6 | M · L · L. Prerequisite for UC-1 measurement and UC-3 | Tool merged with fixture tests; its coverage figure for the LBMH20-386 diff matches the hand-computed 94.1%; `pre-pr-review` step 5 calls it |
| P0 | Make `pre-pr-review` + `diff-vs-spec-reviewer` the standard before Ready | UC-2 | M · L · L. Already built (artifacts 1 and 2) | Every PR open 5 minutes or more in the next 30 days carries `pre-pr-review.md`; reviewed-PR median stays at or below 10177 s |
| P0 | Triage subagent for failed 441 runs | UC-5 | M · L · L. Read-only, small | Replay classifies the 3 failed runs of the window to the same stage as the human; each next 441 failure gets a diagnosis |
| P1 | AI-written tests for the least-covered `Api` code, one folder per PR | UC-1 | H · L · M. Needs UC-6 for changed-code numbers | 445 reports `Api.Tests` line coverage at or above 90.00% (baseline 87.72%, build 29300) |
| P1 | PR build validation as an advisory (non-required) policy | UC-3 Phase A | H · M · M (phase A risk is L). Needs UC-6 | `az repos policy list` for `main` shows the policy; every PR shows its status; PR pipeline duration and false-fail causes recorded |
| P1 | Auto-queue Dev 472 / 440 on merge; Prod and 441 stay manual | UC-4 | H · M · M. Needs UC-2 in use and UC-3 Phase A running; 472 and 440 unchanged (F6) | Every merge touching Api or Web is followed by the matching Dev run; 472 and 440 stay at 0% fail; no automatic 441 or Production run |
| P2 | Make the PR build validation required | UC-3 Phase B | H · M · M. Prerequisite: UC-1 done (coverage at or above 90%) and Phase A stable, otherwise merges freeze | Policy is required on `main`; the first 10 PRs fail only for real reasons; merge lead time not worse than the ~2.8 h reviewed-PR median |

> Dependency summary: UC-6 before UC-1 and UC-3; UC-1 before UC-3 Phase B (never require a coverage gate while 445 sits at 87.72%); UC-3 Phase A and UC-2 before UC-4; UC-4 reuses 440 and 472 as they are.

---

## Traceability checklist

- [x] Every Finding cites a metric that exists in the JSON (no invented numbers).
- [x] Every Finding maps to exactly one station and at least one use case (F6 is a deliberate "leave alone"; UC-4 reuses its stations unchanged).
- [x] Every use case has Impact · Risk · Effort and a verifiable "Done when".
- [x] Prioritisation respects dependencies (no gate enabled before its prerequisite).
- [x] Empty / unmeasured metrics are labelled "not measured", never guessed.
- [x] At least one use case improves something other than writing code (UC-2 review evidence, UC-5 failure triage).
- [x] At least one use case builds a deterministic tool that later removes AI (UC-6; also the rule-based package choice in UC-4).
