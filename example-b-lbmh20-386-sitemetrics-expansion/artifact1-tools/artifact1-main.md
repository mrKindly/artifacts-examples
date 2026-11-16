# Artifact 1 — Project Guidance Setup (Rules · AGENTS.md · Skill)

> Demo example: synthetic evidence (ids, run numbers, timings in chat logs/transcripts) modeled on the LBMH20-386 story shape. Artifact 5 numbers come from the real portal-SDLC.json.

> Purpose: prove this repo has a reusable AI toolkit — passive rules, always-on project context, and at least one repeatable workflow. Everything here must be reusable, maintainable, and shareable, not one-off.

Story thread: [LBMH20-386](https://eci-solutions.atlassian.net/browse/LBMH20-386) — Portal: expand SiteMetrics on the license record detail Metrics tab. Repo: `LBMH.Services.Portal`.

---

## Inventory

| Tool | Type | Location | Installed in Portal repo at |
|------|------|----------|-----------------------------|
| Portal local dev | Rule | [./rules/portal-local-dev.mdc](./rules/portal-local-dev.mdc) | `.cursor/rules/portal-local-dev.mdc` |
| Portal test and coverage | Rule | [./rules/portal-test-coverage.mdc](./rules/portal-test-coverage.mdc) | `.cursor/rules/portal-test-coverage.mdc` |
| Portal agent guide | AGENTS.md | [./AGENTS.md](./AGENTS.md) | `AGENTS.md` (repo root) |
| pre-pr-review | Skill | [./skills/pre-pr-review/SKILL.md](./skills/pre-pr-review/SKILL.md) | `.cursor/skills/pre-pr-review/SKILL.md` |
| pre-pr-review run log | Skill evidence | [./skills/pre-pr-review/log.md](./skills/pre-pr-review/log.md) | not installed (evidence only) |

Not in this set on purpose: the repo already has `.cursor/skills/deploy/SKILL.md` and `.cursor/rules/portal-pipelines.mdc` (listed in [portal-SDLC.json](../artifact5-ai-use-cases/portal-SDLC.json) `documentation`). They cover deploys and pipelines and are reused as-is. The course skills for spec, plan, context map, release strategy and review (MK4-MK9) were used unchanged during the story; they are generic and not re-authored here. Paths in `AGENTS.md` beyond those named in the JSON are modeled for this demo.

---

## 1. Rules (`./rules/*.mdc`)

### portal-local-dev.mdc

- **Why we need this rule.** Three local-run mistakes kept recurring: building only the API and wondering why the UI is stale (SPA must be built into `Api/wwwroot`), reaching for Docker Compose (CI/prod scaffolding, wrong auth and DB wiring), and treating the red `npm run test:ci` as caused by the current change (it exits 1 on `main`). The rule fires only when the agent edits Api C# or Web source, which is exactly when these matter.
- **Why we avoided adding more.** Seven bullets. No architecture, no deploy steps, no style guide: those are either in `AGENTS.md` or change too often to be enforced passively.
- **Why not AGENTS.md or a skill.** `AGENTS.md` carries the same commands for orientation, but a glob-scoped rule is injected at the moment of editing source, so the "build the SPA first" and "no Compose" reminders are not lost in a long file. It is a passive constraint, not a workflow, so it is not a skill.

### portal-test-coverage.mdc

- **Why we need this rule.** The team's bar is "coverage on new/changed code >= 90%, zero new warnings", while the shared QualityCheck baseline is 87.72% line and cannot be the target of a story PR. Without the rule agents either chase the global number or write tests with inconsistent names and no edge classes.
- **Why we avoided adding more.** It defines target, naming, the four edge classes (happy, disabled, empty/null, not-found), and the scoped-run order. It does not explain Coverlet or Vitest setup.
- **Why not AGENTS.md or a skill.** It should apply every time test files are touched, by any agent, without being asked: a glob rule on `*Tests*` and `*.test.ts(x)`. The orchestrated check that uses this bar (collect diff, map ACs, compute coverage) is the skill.

---

## 2. AGENTS.md

- **Why we need this.** Every agent in this repo needs the same orientation: what the Portal is, how the .NET API, React SPA and CDK fit together, how to run it on port 5000, and which commands prove a change. See [./AGENTS.md](./AGENTS.md).
- **Why we avoided adding more.** About 70 lines: a depth-2 map, run, verify, conventions, gotchas, approval gates. Story-specific facts (column lists, AC tables) live in `docs/<STORY-KEY>/`, and deploy detail lives in the existing deploy skill.
- **Why not rule or skill.** It must be loaded for every task, not only when certain files are open, and it is reference material, not a procedure.
- **Gotchas / non-obvious context.** Schema-first (never invent `SiteMetrics` column names); sentinel dates instead of `NULL`; licensing `ediNames` is a different source than `SiteMetrics` EDI flags; QualityCheck 445 is manual and usually red (Coverlet gate 90%, baseline 87.72% on build 29300); merge to `main` deploys nothing; Docker Compose is not for local use.

---

## 3. Skill(s) (`./skills/{name}/SKILL.md`)

### pre-pr-review

- **Why we need this skill.** The manual pre-PR checklist (what changed, do tests prove each AC, what is coverage on my lines) is tedious to repeat by hand and easy to skip under pressure. In [portal-SDLC.json](../artifact5-ai-use-cases/portal-SDLC.json) 16 of 24 PRs to `main` were open under five minutes and there is no required check on PRs (empty branch-policy list), so the reviewer often sees no evidence at all.
- **Why reusable.** Inputs are just a story key, a spec with numbered ACs and optionally a plan. Nothing is specific to SiteMetrics; any Portal story with ACs gets the same report with a PASS/HOLD verdict.
- **Why we avoided adding more.** It does not write code, open PRs, run QualityCheck, deploy, or decide merge. Those are human gates and stay out of the skill. It also does not re-implement the spec-vs-diff review; it delegates that to the read-only subagent in [artifact 2](../artifact2-mcp-subagents/artifact2-main.md).
- **Why not rule or AGENTS.md.** It is a multi-step procedure with outputs, run on demand (once per PR, re-run after fixes). Rules and AGENTS.md are passive context.
- **Gotchas / non-obvious context.** `npm run test:ci` is never the verdict (exit 1 on `main`); only scoped runs plus "failure set unchanged". A PASS means local gates only: QualityCheck 445 must still be queued by a human. Changed-code coverage is computed from the diff because the whole-solution number is under the 90% gate regardless of the story.
- **Usage explanation.** In Cursor chat on the story branch: `/pre-pr-review LBMH20-386`. Read `docs/LBMH20-386/pre-pr-review.md`. On HOLD fix the listed items and run again. On PASS follow the human follow-ups list.
- **Logs of calling skill with agent output.** [./skills/pre-pr-review/log.md](./skills/pre-pr-review/log.md): run 1 HOLD (AC-7 only proved below endpoint level, changed-code coverage 88.6%), run 2 PASS on local gates (94.1% changed-code coverage) with human follow-ups.
