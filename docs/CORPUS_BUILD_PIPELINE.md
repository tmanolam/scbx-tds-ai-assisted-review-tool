# Repo Structure, Coding Assistant Implementation Plan & Corpus Build Pipeline

How this tool gets built in a single repo using an AI coding assistant, and how plain-text checklist/template authoring turns into the indexed artifacts the running system queries. Companion to `TDS_AI_Requirements.md` and `TDS_AI_Architecture.md`.

---

## 1. Repo structure

```
/docs/                          # existing — requirements, architecture, this doc, format specs
  TDS_AI_Requirements.md
  TDS_AI_Architecture.md
  CHECKLIST_ITEM_FORMAT.md
  TEMPLATE_FORMAT.md
  CORPUS_BUILD_PIPELINE.md

/checklist/                     # TDS Committee-authored source content (plain text, see CHECKLIST_ITEM_FORMAT.md)
  cloud-architecture/
  security/
  network/
  data-ai/
  iam/
  observability/
  cloud-cost/

/templates/                     # TDS Committee-authored source content (plain text, see TEMPLATE_FORMAT.md)
  <template-id>/
    template.md
    diagram.mmd

/build/                         # generated, not hand-edited — output of the corpus build pipeline
  checklist-index.json          # normalized, chunked checklist ready for retrieval store ingestion
  template-index.json           # normalized, chunked templates ready for retrieval store ingestion
  revalidation-flags.json       # templates flagged per FR-46

/src/                           # the actual application: ingestion, parsing, checklist engine, routing,
  ingestion/                    #   output artifact generator, audit log, template matching — per
  parsing/                      #   architecture doc §3.1–§3.11
  checklist-engine/
  template-matching/
  routing/
  output-artifact/
  audit-log/

/scripts/
  build-corpus.*                # the parse/validate/chunk/diff script the coding assistant implements (§3 below)

/.github/
  workflows/
    build-corpus.yml            # CI: runs build-corpus on push to checklist/** or templates/**
  CODEOWNERS                    # enforces FR-40: only TDS Committee members can approve changes
                                 #   to /checklist/** and /templates/**

CODEOWNERS.example:
  /checklist/  @tds-committee-team
  /templates/  @tds-committee-team
```

One repo holds the specs, the source content, and the implementation — the coding assistant operates across all of it with full context of the requirements and architecture docs already in `/docs`.

---

## 2. What the coding assistant should build, in order

This maps directly to the architecture doc's roadmap (§9), scoped to what a single-repo, coding-assistant-driven build changes about *how* each stage gets implemented.

1. **`scripts/build-corpus`** — the corpus build pipeline (detailed below). Build this first, even before the full application, because it's what turns your plain-text checklist/template authoring into something usable, and because it validates FR-40/41/42/46 cheaply and early.
2. **`.github/workflows/build-corpus.yml` + `CODEOWNERS`** — wires the pipeline into CI and enforces authorization. This is Stage 5b (corpus maintenance readiness) from the architecture roadmap, but there's no reason not to do it in week one — it's cheap and it validates the format specs against real content immediately.
3. **`/src/ingestion`, `/src/parsing`** — Stage 2 (core pipeline build), Azure-first per the architecture doc.
4. **`/src/checklist-engine`** — Stage 2/3, consumes `build/checklist-index.json`.
5. **`/src/template-matching`** — Stage 4a/5a (design-assist), consumes `build/template-index.json`.
6. **`/src/routing`, `/src/output-artifact`, `/src/audit-log`** — Stage 3.

---

## 3. The corpus build pipeline (what the coding assistant generates from your plain text)

Triggered by CI on any push touching `checklist/**` or `templates/**` (or run locally via the script directly).

**Steps:**

1. **Parse** every `.md` file under `checklist/` and `templates/` — frontmatter (YAML) + body (Markdown sections) per the two format specs.
2. **Validate** each file against its schema: required fields present, `domain`/`csp`/`status` values from the allowed set, `id` uniqueness across the whole tree, `satisfies` references in templates point to checklist IDs that actually exist. Fail the build with a clear error pointing at the offending file — this is the safety net that keeps hand-authored plain text from silently producing a broken index.
3. **Normalize & chunk** each item/template body into the retrieval-ready form (§7 of the architecture doc — chunked deliberately, not dumped as raw text) and write `build/checklist-index.json` / `build/template-index.json`.
4. **Diff against the previous build** (previous commit's `build/*.json`, or the prior state in the retrieval store once Stage 2 infra exists) to determine which checklist item IDs actually changed.
5. **Cross-reference for FR-46**: for every checklist ID that changed, find templates whose `satisfies` list references it. Write `build/revalidation-flags.json` and open a tracking issue (or comment on the triggering PR) listing affected templates — surfaced to the TDS Committee review queue (§3.7), not auto-resolved.
6. **Push to the retrieval store** (once Stage 2's retrieval infrastructure exists) — this is the literal implementation of FR-42's "automatic re-indexing." Until Stage 2 exists, the build artifacts in `/build` are the interim output — useful on their own for reviewing what a change produced.

**Why this satisfies the requirements cheaply:**

| Requirement | How the single-repo + coding-assistant approach satisfies it |
|---|---|
| FR-40 (authorized edits only) | `CODEOWNERS` + branch protection on `checklist/**`, `templates/**` |
| FR-41 (audit trail) | Git commit history — author, diff, timestamp, message — is the audit trail |
| FR-42 (auto re-index) | CI workflow runs the build pipeline on every relevant push |
| FR-43 (in-flight version pinning) | A review references the specific `build/*.json` (or indexed version) live at the time it started — application-level, not something the corpus pipeline itself needs to solve |
| FR-45 (immutable findings) | Output artifacts store a version reference, never re-derived from a later build |
| FR-46 (template re-validation flagging) | Steps 4–5 above |

---

## 4. Suggested immediate next steps

1. **Confirm the format specs** (`CHECKLIST_ITEM_FORMAT.md`, `TEMPLATE_FORMAT.md`) — adjust any fields before content gets authored against them, since changing the schema later means migrating existing files.
2. **Author one pilot checklist item and one pilot template** by hand, in the new format, to sanity-check the format is actually usable before generating many.
3. **Have the coding assistant build `scripts/build-corpus`** against those two pilot files — smallest possible slice that proves the parse → validate → chunk → diff → flag loop works end to end.
4. **Wire up CI + CODEOWNERS** once the script works locally.
5. **TDS Committee starts authoring real checklist content** (Stage 1 of the architecture roadmap) against the now-validated format, in parallel with the coding assistant building `/src/ingestion` and `/src/parsing` (Stage 2).
6. Everything else follows the roadmap in `TDS_AI_Architecture.md` §9, now with an implementation substrate in place.
