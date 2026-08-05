# SCBX TDS AI-Assisted Review Tool

**Status: exploratory design draft — not a decided or committed deliverable.**

This repo works out what an AI-assisted tool for TDS (Technical Design Surgery) could look like, so the option is ready to evaluate if and when the TDS Committee chooses to pursue it. Nothing here reflects a decision to build.

## What this tool would do

Two related capabilities for PortCos going through TDS:

1. **Review** — a PortCo submits an architecture diagram or design document and gets automated, checklist-based feedback (Pass / Gap / Unclear per domain, with citations), with TDS Committee review reserved for ambiguous or high-risk cases.
2. **Design-assist** — a PortCo describes a requirement in plain language and gets matched to a pre-approved reference design template to start from, instead of designing from a blank page. The AI selects from a TDS Committee-owned template library; it does not generate new architectures or infrastructure-as-code.

Both are grounded in a versioned checklist and template corpus owned by the TDS Committee (CCoE Migration Enablement, EA, Cyber, Data) — the AI never relies on memorized policy knowledge; every finding cites a specific checklist item and standard clause.

## Start here

| Doc | What it covers |
|---|---|
| [`docs/TDS_AI_Requirements.md`](docs/TDS_AI_Requirements.md) | Background, goals, stakeholders, functional requirements, success metrics, open questions |
| [`docs/TDS_AI_Architecture.md`](docs/TDS_AI_Architecture.md) | Design principles, system architecture, CSP/AI model/retrieval options, roadmap, risks |
| [`docs/CHECKLIST_ITEM_FORMAT.md`](docs/CHECKLIST_ITEM_FORMAT.md) | Plain-text format for authoring checklist items |
| [`docs/TEMPLATE_FORMAT.md`](docs/TEMPLATE_FORMAT.md) | Plain-text format for authoring reference design templates |
| [`docs/TEMPLATE_LIBRARY_PLAN.md`](docs/TEMPLATE_LIBRARY_PLAN.md) | Which reference design templates to author first, across data sensitivity, regulatory scope, and PCI axes |
| [`docs/CORPUS_BUILD_PIPELINE.md`](docs/CORPUS_BUILD_PIPELINE.md) | Repo structure, implementation plan, and how plain-text checklist/template content becomes an indexed, queryable corpus |

Read the requirements and architecture docs first — the format, library plan, and pipeline docs assume that context.

## Repo structure (planned)

```
/docs/          requirements, architecture, and format/pipeline specs (this is what exists today)
/checklist/     TDS Committee-authored checklist items (plain text) — not yet started
/templates/     TDS Committee-authored reference design templates (plain text) — not yet started
/build/         generated corpus artifacts — not yet started
/src/           application code (ingestion, checklist engine, template matching, etc.) — not yet started
/scripts/       corpus build pipeline — not yet started
/.github/       CI workflow + CODEOWNERS for checklist/template authorization — not yet started
```

Full rationale for this structure is in `docs/CORPUS_BUILD_PIPELINE.md` §1.

## Where things stand

- Requirements and architecture are drafted, including the review flow, design-assist flow, and post-launch checklist/template maintenance workflow.
- Checklist and template authoring formats are defined, so content can start once the TDS Committee is ready.
- An initial template library plan (which patterns to author first, and in what priority order) is drafted.
- No code has been written yet. See `docs/CORPUS_BUILD_PIPELINE.md` §4 for suggested next steps.
- Several decisions remain open — see `docs/TDS_AI_Requirements.md` §9 and `docs/TDS_AI_Architecture.md` §11.
