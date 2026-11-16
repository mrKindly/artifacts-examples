# Examples for demo — Cursor AI Bootcamp artifacts

Two complete example sets, each with all five artifacts. They follow the templates in `resources-main/atifacts/` and the depth of the real submissions in `artifacts-examples/`. Each set tells one story end to end, so the thread "what I built in Artifact 1 shows up in Artifact 3" is visible.

> The evidence is **synthetic** (modeled on the two story shapes in `resources-main/curated-backlog/`). Ids, run numbers, PR links, chat logs and timings are illustrative. The one exception: Example B's Artifact 5 uses the real `SDLC.example.json` from `resources-main`.

## The two examples

| | Example A | Example B |
|---|-----------|-----------|
| Folder | [`example-a-fs07-3521-fulfilled-orders-by-make/`](./example-a-fs07-3521-fulfilled-orders-by-make/) | [`example-b-lbmh20-386-sitemetrics-expansion/`](./example-b-lbmh20-386-sitemetrics-expansion/) |
| Story shape | New integration API with ambiguous domain rules (FS07-3521) | Expand an existing read model with strict display rules (LBMH20-386) |
| Stack | .NET Framework WCF SOAP + SQL Server procs + cloud router | .NET API + React SPA + AWS CDK + SQL Server |
| Why a spec is mandatory | "Fulfilled", watermark and row grain are easy to implement plausibly and wrongly | Which columns to show, hide and de-dupe; schema-first names; display rules |
| Artifact 1 | 2 rules, `AGENTS.md`, `add-soap-operation` skill | 2 rules, `AGENTS.md`, `pre-pr-review` skill |
| Artifact 2 | `spec-reviewer` subagent + read-only Atlassian MCP | `diff-vs-spec-reviewer` subagent + read-only SQL Server MCP |
| Artifact 3 | Story → spec → plan → context map → verification (14 ACs) | Story → spec → plan → context map → verification (10 ACs) |
| Artifact 4 | Decision: **Merge** (staged deploy, route switch) | Decision: **Hold** (missing off-machine coverage evidence) |
| Artifact 5 | Synthetic `fs-SDLC.json`, 6 findings, 5 use cases | Real `portal-SDLC.json`, findings and use cases derived from it |

Having one **Merge** and one **Hold** outcome shows both sides of the evidence-based decision.

## Layout of each example

```text
example-*/
├── artifact1-tools/            AGENTS.md · rules/*.mdc · skills/<name>/SKILL.md · artifact1-main.md
├── artifact2-mcp-subagents/    agents/ (config, transcript, refusal log) · mcp/ (config, transcript, refusal log) · artifact2-main.md
├── artifact3-workflow/         story · spec · plan · context-map · verification · review-notes · chat logs · artifact3-main.md
├── artifact4-deliverables/     verification · review · release · chat logs · artifact4-main.md
└── artifact5-ai-use-cases/     *-SDLC.json (source of truth) · artifact5-main.md (playbook)
```

Start each demo walk-through from the `artifactN-main.md` file: it is the index with links to every piece of evidence.
