# Reference Design Template Library — Initial Set Plan

Which reference design templates to author first, and in what order. Companion to `TEMPLATE_FORMAT.md` (how to author each one) and the roadmap Stages 1a/4a in `TDS_AI_Architecture.md` §9 (when authoring happens).

**Framing:** three axes distinguish templates — `data_sensitivity`, `regulatory_scope`, and `pci_scope` — and they are not interchangeable. `data_sensitivity` (PII, confidential, etc.) is what most often drives real architecture differences (encryption, access controls, masking) and is *not* the same thing as `regulatory_scope`. `regulatory_scope: financial` means the workload carries financial-regulatory obligations (e.g. BOT/AMLO-style audit retention, transaction monitoring, data governance) *beyond* what its data sensitivity already implies — it describes the workload, not which PortCo happens to build it. `pci_scope` is its own axis again, specifically about card-data network segmentation.

**A template only gets split by `regulatory_scope` or `pci_scope` when that axis genuinely changes the diagram or the `satisfies` list** — not just as a label. Splitting every pattern into a full financial/non-financial matrix would roughly double this library for a team that's explicitly capacity-constrained (see `TDS_AI_Architecture.md` §1 principle 5), so the table below tags each template `financial` only where a real, distinct checklist obligation exists beyond generic PII handling; everything else is tagged `none` and relies on the matching engine's residual-gap flow (FR-36) to surface any extra items a specific financial submission needs.

---

## Tier 1 — author first (highest expected match volume)

These cover what most PortCo submissions are expected to look like. **The pilot (Stage 4a/5a) needs only this tier.**

| # | Template | data_sensitivity | regulatory_scope | pci_scope | Why first |
|---|---|---|---|---|---|
| 1 | **Public-facing web/mobile app backend** — App Service/AKS + WAF + API gateway + managed DB | public/internal | none | out-of-scope | The single most common pattern across any PortCo — the baseline everything else varies from. No sensitive data, no regulatory divergence — same template regardless of which PortCo builds it. |
| 2 | **Public-facing web app handling customer PII** — adds Private Link, CMK encryption, PDPA-aligned data handling, stricter logging | pii, confidential | financial | out-of-scope | Most SCBX PortCos touch customer PII. Tagged `financial` because a bank-context PII workload carries audit-retention and data-governance obligations (per Group standards) beyond generic PDPA handling — a genuine `satisfies` difference, not just a label. |
| 3 | **Card payment acceptance / CDE-adjacent app** — segmented CDE zone, tokenization boundary, HSM/Key Vault Managed HSM, PCI DSS network isolation | pii, restricted | financial | cde | Directly relevant to CardX and any PortCo taking card payments; the pattern where a wrong design is most expensive to fix later |
| 4 | **Internal API / microservice, no public exposure** — private endpoints only, internal APIM, service-to-service auth | internal | none | out-of-scope | Very common; also the "safe default" to point PortCos at when they overexpose services. Same diagram regardless of payload — if a financial payload needs extra retention, that's a residual gap, not a reason to fork this template. |
| 5 | **Batch data pipeline with on-prem integration** — ExpressRoute/VPN, Data Factory or equivalent, staged storage, no cross-border | internal/confidential | none | out-of-scope | Bank-to-cloud integration is a recurring TDS topic, but the pattern itself (network + data-residency mechanics) is identical whether the payload is financial or not — e.g. non-financial PortCos integrate with on-prem HR/logistics systems too |

## Tier 2 — author next (common, but fewer submissions expected)

| # | Template | data_sensitivity | regulatory_scope | pci_scope | Notes |
|---|---|---|---|---|---|
| 6 | **Analytics / data platform with PII** — lakehouse zoning (raw→curated→consumption), data classification gates, masked consumption layer | pii, confidential | financial | out-of-scope | Data & AI domain will see these. Tagged `financial` for the same reason as #2 — bank financial-data platforms have classification/retention obligations beyond generic PII masking. |
| 7 | **AI/ML workload using an LLM or model endpoint** — private AI endpoint, no-training-retention terms, prompt/output logging | varies (set per submission) | none | out-of-scope | Group AI Policy is a checklist domain — worth a template before ad-hoc AI designs proliferate. The pattern (private endpoint, no-retention terms) doesn't itself change based on regulatory scope; if the data flowing through is PII, `data_sensitivity` carries that, not this field. |
| 8 | **Event-driven / async processing** — Service Bus/Event Hubs, DLQ handling, idempotency, private messaging | internal | none | out-of-scope | Common in fintech transaction flows, but the messaging pattern itself is data-agnostic |
| 9 | **Static content / marketing site** — CDN + static hosting + WAF, no sensitive data | public | none | out-of-scope | Trivial pattern, but lets low-risk submissions self-serve instantly — good for adoption |
| 10 | **B2B partner integration via exposed API** — APIM external tier, mTLS/OAuth client credentials, rate limiting, partner network allowlisting | confidential | none | out-of-scope | Recurring pattern for INVX/CardX partner ecosystems, but the integration architecture itself doesn't change based on regulatory scope — a financial-data B2B flow gets the same diagram, with any extra obligation surfaced as a residual gap |

## Tier 3 — only when demand appears

| # | Template | data_sensitivity | regulatory_scope | pci_scope | Notes |
|---|---|---|---|---|---|
| 11 | **PCI-scoped reporting/back-office access to card data** — read paths into CDE, jump-host/PAW access, session recording | restricted | financial | cde-adjacent | Only needed once #3 exists and real PCI submissions arrive |
| 12 | **Multi-region DR/high-availability variant of #2** | pii, confidential | financial | out-of-scope | Authored as a variant of #2, not standalone — resilience requirements bolt onto an existing pattern; inherits #2's regulatory tagging |

---

## Authoring guidance

- **Start with Tier 1 only (5 templates), not all 12.** Per Stage 1a/4a, the pilot needs "reasonable coverage of common patterns, not full breadth." Authoring 12 templates before any pilot feedback risks building coverage nobody matches against. Pilot data (which requirements matched nothing — the FR-36 "no reasonable match" cases) should drive what gets authored next, more than this plan.
- **Before adding a new template, check whether it's really a split or a residual gap.** If a proposed "financial variant" of an existing template would have the same diagram and the same `satisfies` list, it isn't a new template — the matching engine should route that requirement to the existing template and flag the extra financial-specific items as things the PortCo still needs to address (FR-35/FR-36), not duplicate the whole pattern.
- **#3 (PCI) needs the deepest TDS Committee/Cyber involvement.** Non-PCI templates can be drafted and reviewed quickly; the PCI template encodes segmentation decisions Cyber will want to own. Start that conversation early even though it's one template.
- **All initial templates target Azure** (`target_csp: azure`), per the roadmap — AWS/GCP-native equivalents come in Stage 7 as those CSPs are Group-approved.
- **Each template must be validated against the checklist** (its `satisfies` list, per `TEMPLATE_FORMAT.md`) before its first commit — which means Tier 1 authoring can't fully complete until the checklist itself is at least stable-draft (Stage 1 dependency).
- This plan is a starting proposal for TDS Committee prioritization — decision item #7 in `TDS_AI_Architecture.md` §11 ("how many templates for a credible pilot, and who prioritizes") remains theirs to confirm, including whether they agree with which templates above are tagged `financial`.
