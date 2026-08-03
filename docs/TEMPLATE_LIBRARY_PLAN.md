# Reference Design Template Library — Initial Set Plan

Which reference design templates to author first, and in what order. Companion to `TEMPLATE_FORMAT.md` (how to author each one) and the roadmap Stages 1a/4a in `TDS_AI_Architecture.md` §9 (when authoring happens).

**Framing:** the two classification axes — **financial vs. non-financial** and **PCI vs. non-PCI** — don't require four separate template families. Financial context mostly drives regulatory/data-residency strictness and audit depth; PCI scope drives network segmentation and encryption specifics (CDE isolation, key management). So most patterns exist as a base form plus a stricter variant, which keeps the library maintainable for a small team. Both axes are captured as filterable frontmatter metadata (`business_context`, `pci_scope` — see `TEMPLATE_FORMAT.md`) so the matching engine can filter on them before semantic matching.

---

## Tier 1 — author first (highest expected match volume)

These cover what most PortCo submissions are expected to look like. **The pilot (Stage 4a/5a) needs only this tier.**

| # | Template | business_context | pci_scope | Why first |
|---|---|---|---|---|
| 1 | **Public-facing web/mobile app backend** — App Service/AKS + WAF + API gateway + managed DB | non-financial | out-of-scope | The single most common pattern across any PortCo — the baseline everything else varies from |
| 2 | **Public-facing web app handling customer PII** — adds Private Link, CMK encryption, PDPA-aligned data handling, stricter logging | financial | out-of-scope | Most SCBX PortCos touch customer PII; likely the highest real-world match |
| 3 | **Card payment acceptance / CDE-adjacent app** — segmented CDE zone, tokenization boundary, HSM/Key Vault Managed HSM, PCI DSS network isolation | financial | cde | Directly relevant to CardX and any PortCo taking card payments; the pattern where a wrong design is most expensive to fix later |
| 4 | **Internal API / microservice, no public exposure** — private endpoints only, internal APIM, service-to-service auth | both | out-of-scope | Very common; also the "safe default" to point PortCos at when they overexpose services |
| 5 | **Batch data pipeline with on-prem integration** — ExpressRoute/VPN, Data Factory or equivalent, staged storage, no cross-border | financial | out-of-scope | Bank-to-cloud integration is a recurring TDS topic; encodes the network + data-residency answers once |

## Tier 2 — author next (common, but fewer submissions expected)

| # | Template | business_context | pci_scope | Notes |
|---|---|---|---|---|
| 6 | **Analytics / data platform with PII** — lakehouse zoning (raw→curated→consumption), data classification gates, masked consumption layer | financial | out-of-scope | Data & AI domain will see these; also where cross-border flags matter most |
| 7 | **AI/ML workload using an LLM or model endpoint** — private AI endpoint, no-training-retention terms, prompt/output logging | both | out-of-scope | Group AI Policy is a checklist domain — worth a template before ad-hoc AI designs proliferate |
| 8 | **Event-driven / async processing** — Service Bus/Event Hubs, DLQ handling, idempotency, private messaging | both | out-of-scope | Common in fintech transaction flows; often designed insecurely by default |
| 9 | **Static content / marketing site** — CDN + static hosting + WAF, no sensitive data | non-financial | out-of-scope | Trivial pattern, but lets low-risk submissions self-serve instantly — good for adoption |
| 10 | **B2B partner integration via exposed API** — APIM external tier, mTLS/OAuth client credentials, rate limiting, partner network allowlisting | financial | out-of-scope | Recurring pattern for INVX/CardX partner ecosystems |

## Tier 3 — only when demand appears

| # | Template | business_context | pci_scope | Notes |
|---|---|---|---|---|
| 11 | **PCI-scoped reporting/back-office access to card data** — read paths into CDE, jump-host/PAW access, session recording | financial | cde-adjacent | Only needed once #3 exists and real PCI submissions arrive |
| 12 | **Multi-region DR/high-availability variant of #2** | financial | out-of-scope | Authored as a variant of #2, not standalone — resilience requirements bolt onto an existing pattern |

---

## Authoring guidance

- **Start with Tier 1 only (5 templates), not all 12.** Per Stage 1a/4a, the pilot needs "reasonable coverage of common patterns, not full breadth." Authoring 12 templates before any pilot feedback risks building coverage nobody matches against. Pilot data (which requirements matched nothing — the FR-36 "no reasonable match" cases) should drive what gets authored next, more than this plan.
- **#3 (PCI) needs the deepest TDS Committee/Cyber involvement.** Non-PCI templates can be drafted and reviewed quickly; the PCI template encodes segmentation decisions Cyber will want to own. Start that conversation early even though it's one template.
- **All initial templates target Azure** (`target_csp: azure`), per the roadmap — AWS/GCP-native equivalents come in Stage 7 as those CSPs are Group-approved.
- **Each template must be validated against the checklist** (its `satisfies` list, per `TEMPLATE_FORMAT.md`) before its first commit — which means Tier 1 authoring can't fully complete until the checklist itself is at least stable-draft (Stage 1 dependency).
- This plan is a starting proposal for TDS Committee prioritization — decision item #7 in `TDS_AI_Architecture.md` §11 ("how many templates for a credible pilot, and who prioritizes") remains theirs to confirm.
