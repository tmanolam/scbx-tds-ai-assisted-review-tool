# AI-Assisted TDS Review Tool — Requirements

**Owner:** TDS Committee (CCoE Migration Enablement, EA, Cyber, Data)
**Status:** Exploratory design draft. An AI-assisted tool was raised as one idea during discussion (noted under Phase 4 of the TDS roadmap) — it is **not a decided or committed deliverable**. This document works out what such a tool would need to do, so the option is ready to evaluate if and when the TDS Committee chooses to pursue it.
**Related:** TDS Forum Expansion and Improvement (deck), TDS AI Architecture Design (companion doc)

---

## 1. Background

TDS (Technical Design Surgery) today sits with EA, with known **capacity gaps**. This effort is **not an ownership transfer** — CCoE Migration Enablement is joining EA, Cyber, and Data as an expanded TDS Committee, to add capacity and improve how TDS runs. The operating model:

- TDS is **voluntary** for PortCos, per Group Policy — this is not changing.
- Where a PortCo already runs its own governance forum (e.g. AWG and Infra Day at SCB Bank, ARC at INVX, DAC at CardX), the TDS Committee **joins that forum** rather than requiring a separate TDS session. For PortCos without their own forum, TDS is offered as a **forum-as-a-service**.
- A **checklist**, tied directly to Group policies and standards, replaces the expectation that PortCos read the underlying documents themselves.
- The Group is expanding beyond Azure-only to a **multi-cloud strategy** (AWS and GCP coming soon), so TDS coverage needs to extend beyond Azure.
- CADRB's current format — a 1-week pre-read window and a 1-hour formal session — is too short for detailed review. TDS is meant to absorb that detail work upstream, so CADRB only handles items that genuinely need its decision authority.

Because participation stays voluntary and the team is capacity-constrained (**currently 4 staff, plus 2 allocated to other teams**), the highest-leverage use of AI is making self-review fast enough that PortCos choose to use it, and light enough that a small TDS Committee can sustain it. This document defines requirements for an AI-assisted tool where a PortCo submits an architecture diagram or design document and receives automated, checklist-based recommendations, with TDS Committee review reserved for genuinely ambiguous or high-risk cases.

---

## 2. Goals

| # | Goal |
|---|------|
| G1 | Let a PortCo get structured, checklist-based feedback on a cloud design in minutes, without waiting for a session. |
| G2 | Reduce the reading burden on PortCos — they interact with a checklist and its output, not the underlying policy/standards documents. |
| G3 | Produce a consistent output artifact (findings, conditions, sign-off) for every review, feeding CADRB — so CADRB's limited session time is spent only on items needing its decision. |
| G4 | Preserve TDS Committee judgment for ambiguous, novel, or high-risk designs — AI drafts, a human signs off. |
| G5 | Keep the checklist and underlying Group standards as the single source of truth; avoid AI inventing or misquoting policy. |
| G6 | Support designs across all in-scope CSPs — Azure today, extending to AWS and GCP as the Group's multi-cloud strategy rolls out. |
| G7 | Stretch a capacity-constrained TDS Committee (4 staff, +2 shared) further, without requiring every PortCo interaction to be a live session. |

### Non-Goals

- This tool does not make TDS mandatory — participation remains optional per Group Policy.
- This tool does not replace CADRB decision-making — it feeds CADRB, and helps filter what actually needs CADRB's time.
- This tool does not replace or duplicate a PortCo's own forum (AWG, ARC, DAC, Infra Day, etc.) — where one exists, the TDS Committee joins it.
- This tool does not review application-layer solutions — cloud/infrastructure scope only.
- This tool does not automatically approve or reject a design; it only recommends.

---

## 3. Stakeholders / Personas

| Persona | Need |
|---|---|
| **PortCo Solution Owner / Architect** | Fast, self-serve feedback on a design without waiting for a session; plain-language guidance, not policy documents. |
| **TDS Committee** (CCoE Migration Enablement, EA, Cyber, Data) | A consistent pipeline that stretches limited capacity; a manageable queue focused on genuinely hard cases; a shared, joint accountability model (no single owner). |
| **CADRB** | A consistent output artifact — findings, conditions, sign-off — for every solution reaching them, on any CSP, so its short session time goes to decisions rather than discovery. |
| **PortCo-owned forum** (AWG, Infra Day at SCB Bank; ARC at INVX; DAC at CardX; etc.) | A committee that joins their existing process rather than asking them to run a parallel one. |
| **PortCo without an owned forum** | TDS offered as a forum-as-a-service, so they aren't left without any design-review venue. |

---

## 4. Functional Requirements

### 4.1 Intake

- **FR-1**: The system must accept an architecture diagram (image, PDF export, draw.io/Visio/Lucidchart export) as input.
- **FR-2**: The system must accept a design document (Word, PDF, markdown, or plain text) as input, optionally alongside a diagram.
- **FR-3**: The system must accept metadata with each submission: PortCo name, project name, target CSP(s), submitter contact, and — where applicable — which PortCo forum (if any) the project is also being reviewed through.
- **FR-4**: The system should provide a lightweight diagram template/guideline PortCos can optionally use to improve extraction accuracy (not mandatory).
- **FR-4a**: Because there is currently no reliable process to discover new projects before CADRB registration, the system should support a lightweight "register a project" entry point at kick-off, so a submission can happen early rather than only surfacing at CADRB.

### 4.2 Parsing & Extraction

- **FR-5**: The system must extract a structured representation from the submitted diagram/document: components, data flows, network zones/segmentation, CSP services used, IAM roles, encryption/data handling, and cost-relevant sizing (SKU choices).
- **FR-6**: The system must handle Azure service naming/patterns at launch, with AWS and GCP support phased in as the Group's multi-cloud rollout proceeds.
- **FR-7**: The system must flag when extraction confidence is low (e.g. poor-quality image, ambiguous diagram) rather than silently guessing.

### 4.3 Checklist Matching

- **FR-8**: The system must evaluate the extracted design against a machine-readable checklist covering the finalized review domains:
  - Cloud Architecture (Cloud Usage Guideline, Cloud Usage Policy, Group Cloud Architecture Standard)
  - Security Standard (Group Cloud Security Standard, Group Information and Cybersecurity Policy)
  - Network Standard (Group Network Security Standard)
  - Data and AI (Group Data Security and Encryption Standard, Group AI Policy)
  - IAM (Group IAM Standard)
  - Observability / Cyber Log (Group Logging and Auditing Standard)
  - Cloud Cost (no formal standard — advisory guidance toward the lowest-cost/highest-discount SKU that meets requirements)
- **FR-9**: Each checklist item result must be one of: **Pass**, **Gap**, or **Unclear / needs more information**. Cloud Cost findings are always advisory, never a Pass/Gap compliance call.
- **FR-10**: Every checklist evaluation (except Cloud Cost) must cite the specific checklist item and underlying Group standard/policy clause it is based on.
- **FR-11**: The checklist content itself must be maintained independently of the AI system (versioned, owned by the TDS Committee), so a policy update doesn't require a model change.

### 4.4 Recommendations

- **FR-12**: For every **Gap**, the system must generate a specific, actionable recommendation (not generic advice), referencing the relevant standard.
- **FR-13**: Each recommendation must carry a confidence score.
- **FR-14**: The system must present results as a scorecard: pass/gap/unclear counts per domain, plus cost-optimization suggestions, and the list of specific recommendations.

### 4.5 Confidence-Based Routing

- **FR-15**: High-confidence, low-risk gaps must be returned directly to the PortCo for self-serve fix-and-resubmit — no TDS Committee time required.
- **FR-16**: Low-confidence findings, or findings tagged high-risk (e.g. cross-border data, novel architecture pattern, security-critical gap), must be routed to a TDS Committee reviewer before being finalized.
- **FR-17**: The routing logic (what counts as "high-risk") must be configurable by the TDS Committee without a code change — this is TDS policy, not application logic.
- **FR-18**: A PortCo must be able to revise and resubmit a design after addressing AI-flagged gaps, and see a delta from the previous submission.
- **FR-18a**: Where a project is already going through a PortCo-owned forum, the routing/escalation path should support a "join their session" outcome rather than always scheduling a separate TDS session.

### 4.6 Output Artifact

- **FR-19**: The system must generate a TDS output artifact per completed review, matching the agreed minimum viable format: a document recording **Findings**, **Conditions**, and **sign-off**.
- **FR-20**: The sign-off mechanism should support email sign-off at minimum (per current decision); the specific sign-off authority (which role signs off) is still open — see §9.
- **FR-21**: A human must explicitly sign off before an artifact is marked final and sent to CADRB.

### 4.7 TDS Committee Review Workflow

- **FR-22**: Escalated cases must appear in a review queue visible to TDS Committee members (CCoE Migration Enablement, EA, Cyber, Data — whoever is designated for that domain).
- **FR-23**: A reviewer must be able to accept, edit, or override any AI-generated finding or recommendation before sign-off.
- **FR-24**: Escalated cases should support scheduling into a multi-session consult (per the proposed TDS pattern of multiple lighter touchpoints instead of one formal CADRB-style session), or into the PortCo's own forum where one exists.

### 4.8 Audit & Logging

- **FR-25**: Every submission, AI output, routing decision, and human decision must be logged and retrievable.
- **FR-26**: Logs must be sufficient to demonstrate, after the fact, that a given solution went through TDS and what the outcome was.
- **FR-27**: The system must record which checklist version was used for each review (for traceability as standards evolve), and which CSP(s) the design targets.

### 4.9 Integration

- **FR-28**: The system must support the agreed engagement channels: CWG announcement, a single mail group for intake, and a separate group chat per PortCo (with a named PIC per PortCo).
- **FR-29**: The system should support an integration point with CADRB's intake process (manual export acceptable for v1; API integration as a later phase).
- **FR-30**: For PortCos with their own forum, the system should support tagging a submission as "reviewed via [PortCo forum name]" so TDS doesn't duplicate a review that already happened elsewhere.

---

## 5. Non-Functional Requirements

| Category | Requirement |
|---|---|
| **Data sensitivity** | Architecture diagrams and design docs may contain sensitive network/security detail. The system must not send this data to a public/shared AI endpoint without an explicit data-handling agreement; prefer private/enterprise-tier AI endpoints with no training-data retention. |
| **Data residency** | Submitted data and logs should be stored within an approved region/tenant consistent with SCBX group data policy. |
| **Access control** | Only authorized PortCo submitters and TDS Committee reviewers may access submissions; role-based access required. |
| **Auditability** | Every finding must be traceable to a checklist item and standard clause; every human decision must be attributed and timestamped. |
| **Explainability** | Recommendations must be understandable by a PortCo architect without TDS Committee assistance — plain language, not raw policy text. |
| **Availability** | Self-serve review path should be usable during business hours at minimum; not required to be 24/7 at launch. |
| **Performance** | Initial automated feedback (parse + checklist match + draft recommendations) should return within a few minutes for a typical submission — this is what makes the tool a real alternative to waiting on a capacity-constrained team. |
| **Extensibility** | Checklist items, standards references, and routing rules must be updatable by the TDS Committee without redeploying the AI pipeline. Multi-cloud coverage (AWS, GCP) must be addable without a redesign. |
| **Model/vendor independence** | The architecture should avoid hard-coupling to a single AI vendor or CSP where reasonably possible, to allow future re-evaluation (see companion architecture document for options). |

---

## 6. Example User Stories

1. *As a PortCo architect*, I want to upload my architecture diagram and get a checklist scorecard back quickly, so I don't have to read the full set of Group standards myself.
2. *As a PortCo architect at a company with its own forum (e.g. AWG, ARC, DAC)*, I want TDS to join my existing forum, not ask me to run a second review process.
3. *As a PortCo architect without an owned forum*, I want TDS available as a forum-as-a-service so I still have somewhere to bring my design.
4. *As a TDS Committee reviewer*, I want only genuinely ambiguous or high-risk cases in my queue, given our limited staff, so I'm not re-reviewing things AI already handled correctly.
5. *As a TDS Committee reviewer*, I want to see exactly which standard clause a finding is based on, so I can validate or override it quickly.
6. *As a CADRB member*, I want every solution to arrive with a findings/conditions/sign-off artifact, regardless of CSP, so our 1-hour session is spent deciding, not discovering.
7. *As the TDS Committee*, I want a full audit trail per review, so we can demonstrate TDS was applied if asked later.
8. *As the TDS Committee*, I want to update the checklist when a standard changes, or when AWS/GCP coverage is added, without needing to retrain or redeploy the AI system.

---

## 7. Success Metrics (proposed)

- % of PortCo submissions resolved without TDS Committee escalation (capacity-relief signal).
- Median time from submission to first feedback.
- % of CADRB submissions arriving with a completed findings/conditions/sign-off artifact.
- Reduction in CADRB session time spent on discovery vs. decision-making.
- PortCo satisfaction / re-use rate (voluntary tool — repeat usage signals it's actually helpful).
- Number of PortCos joined via their own forum vs. onboarded via forum-as-a-service.
- Number of findings later overturned by a reviewer (signal for AI accuracy / confidence calibration).

---

## 8. Assumptions & Constraints

- The TDS checklist (Slide 7 domains: Cloud Architecture, Security, Network, Data and AI, IAM, Observability/Cyber Log, Cloud Cost) exists or will be authored as a structured, versioned artifact.
- TDS remains a **joint committee** (CCoE Migration Enablement, EA, Cyber, Data) — no single team owns it; this tool must not assume a single-owner approval chain.
- PortCos are not required to use this tool; it is one channel among others, including PortCo-owned forums.
- Initial CSP coverage is Azure; AWS and GCP checklist coverage is phased in alongside the Group's multi-cloud rollout.
- Initial scope is cloud/infrastructure designs only, not application-layer review.
- CADRB's intake format/process is assumed to accept a structured document in v1; deeper API integration is a later phase.
- Current TDS Committee capacity is 4 staff (plus 2 shared with other teams) — this materially shapes how much can go to escalation vs. self-serve.

---

## 9. Open Questions

- **Capacity**: current team is 4 staff (+2 shared). To be re-evaluated once the checklist is finalized and a pilot project has run — how much load can this tool realistically take off the team, and does that change the answer?
- **Sign-off authority**: the output artifact will use email sign-off, but *who* holds sign-off authority (which role, per domain) is still undecided.
- **Multi-cloud timing**: what is the actual rollout sequence for AWS/GCP approval, and does the checklist/AI tool need AWS/GCP coverage ready before or after Group approval lands?
- **Forum-as-a-service**: for PortCos without an owned forum, what does "TDS as their forum" look like operationally — recurring session, on-demand, or purely tool-driven?
- **Cost domain**: since there's no formal standard for Cloud Cost, how prescriptive should AI recommendations be (e.g. can it suggest specific SKUs, or only flag "cheaper option available")?
- **Reference architectures**: Phase 4 also calls for "other reference architecture[s]" beyond the standard checklist — is this in scope for this tool, or a separate, parallel workstream?
