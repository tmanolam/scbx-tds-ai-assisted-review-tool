# Checklist Item Format

Defines the plain-text format TDS Committee members author checklist items in. One file per item, committed to `checklist/<domain>/<ID>.md`. The coding assistant's generation pipeline (see `docs/CORPUS_BUILD_PIPELINE.md`) parses these into the indexed, chunked artifacts the checklist engine (§3.4) actually queries at review time — the TDS Committee never touches the indexed output directly, only these source files.

## File location

```
checklist/
  cloud-architecture/
    CA-001.md
    CA-002.md
  security/
    SEC-001.md
  network/
    NET-001.md
  data-ai/
    DAI-001.md
  iam/
    IAM-001.md
  observability/
    OBS-001.md
  cloud-cost/
    COST-001.md
```

Domain folder name must match one of the seven finalized review domains (requirements doc §4.3). File name is the item ID.

## Frontmatter schema

```yaml
---
id: CA-001                      # required, unique, stable across edits (used for FR-27 version tracking and FR-46 template cross-reference)
domain: cloud-architecture      # required, must match one of the 7 domains
title: "Landing zone segmentation required"   # required, short human-readable title
csp: [azure]                    # required, one or more of: azure, aws, gcp, all
standard_name: "Group Cloud Architecture Standard"  # required — the Group policy/standard this item enforces
clause_ref: "3.2.1"             # required — specific clause/section within the standard, for citation (FR-10)
advisory: false                 # required, true only for cloud-cost domain items (FR-9: cost is always advisory, never Pass/Gap)
status: active                  # required: active | retired
version: 1                      # required, integer — bump on every substantive edit (auto-incremented by the build pipeline if omitted)
last_updated: 2026-07-31        # required, ISO date — set by the build pipeline from commit metadata if omitted
---
```

## Body sections (Markdown, in this order)

```markdown
## Requirement

Plain-language statement of what must be true about a design for this item to Pass.
Written for the AI to evaluate against, and for a PortCo architect to understand
without reading the underlying standard.

## Check Guidance

What the checklist engine should look for in the structured design model (§3.2)
to decide Pass / Gap / Unclear. Be concrete about what evidence counts.

## Standard Reference

A short paraphrase of the relevant clause — enough for the citation (FR-10) to be
meaningful, not a full reproduction of the policy text. Point to where the full
clause lives (internal standards doc) rather than quoting it at length.

## Example Gap (optional)

One brief example of what a Gap finding looks like for this item, to calibrate
tone/specificity for the recommendation generator (§3.5).
```

## Editing rules

- **New item**: add a new file with a new, unused ID. The build pipeline validates ID uniqueness across the whole `checklist/` tree.
- **Modify item**: edit the file in place. Bump `version`, or let the pipeline auto-bump it. The previous version remains in git history — that history *is* the version record (no separate versioning system needed).
- **Retire item**: set `status: retired` rather than deleting the file. A retired item is excluded from active checklist evaluation but stays in history for audit purposes (so old findings that cited it remain interpretable).
- **PR review**: any PR touching `checklist/**` requires TDS Committee approval per `CODEOWNERS` (see `docs/CORPUS_BUILD_PIPELINE.md`) — this is FR-40's "authorized edit access," enforced by GitHub branch protection rather than custom application logic.
