# APEX decision log

This is the canonical history of product and build decisions. Current implementation details live in `07-Architecture.md`; unresolved discovery items live in `05-Risks.md`. All entries below were recorded on September 10, 2026. Earlier stack decisions are carried forward from the existing architecture register, not newly selected in this session.

For every future build decision, append a dated entry with status, decision, rationale, alternatives/tradeoffs, and affected documents. Update affected specifications in the same change. Never label a recommendation approved without owner agreement. Supersede entries explicitly rather than silently deleting history. Logging does not itself authorize deployment or spending.

## Approved decisions

| ID | Decision | Rationale and tradeoff | Affected documents |
|---|---|---|---|
| D001 | Vite + React SPA; Hono owns the API. | No server-rendering requirement has been identified. Vite gives a clear frontend/API boundary without a second backend runtime. Next.js was considered and not selected. | AGENTS, README, Architecture, WBS |
| D002 | Retain Hono + TypeScript on Cloudflare Workers. | Existing M0 choice: native Cloudflare bindings, one application language, and limited infrastructure work. FastAPI is a historical alternative, not an active default. | Architecture, AGENTS |
| D003 | Retain D1 for relational data, Vectorize for embeddings, R2 for source files. | Existing M0 choice: managed services on one platform. Avoid a separate Postgres deployment; application-level ownership and consistency still require implementation. | Architecture |
| D004 | Retain Workers AI bge-m3 embeddings and Claude Sonnet-class synthesis / Haiku-class extraction through AI Gateway. | Existing M0 choice: separate retrieval, synthesis, and cheaper extraction responsibilities; keep provider keys server-side. Concrete model IDs remain open and require measured quality/cost checks. | Architecture, Risks |
| D005 | Retain Tailwind, shadcn/ui, React Query, pnpm workspaces, and TypeScript in apps/packages. Recharts remains a later charting choice. | Existing M0 conventions reduce bespoke styling, state, and tooling work. Synthesis-first scope does not require building charts now. Python remains available only for offline ingestion. | AGENTS, Architecture |
| D006 | Ship research synthesis first; defer Stance Tracker and Action Tracer. | Test the core evidence-backed research value before spending limited time on additional capabilities. Earlier all-three-feature release gates are superseded. | PRD, Scope, WBS, Milestones, Acceptance |
| D007 | Target a nonprofit or small organization in Los Angeles. Contact a prospective pilot within two weeks, by September 24. | Learn from real research work before fixing the first topic, collection, and deliverable. No organization or research output has been selected yet. | Risks |
| D008 | November 3, 2026 is the shipping target, not an election-specific workflow requirement. Owner availability is 10 hours/week. | Gives planning a concrete deadline and capacity constraint, approximately 77 owner hours from September 10. This is available time, not a validated build estimate. | Milestones, WBS, Risks |
| D009 | Evidence quality takes precedence over the date; reduce coverage or delay the pilot if material unsupported claims remain. | Credibility is the core product promise. Shipping a misleading synthesis would defeat the intended value. | Acceptance, Risks |
| D010 | Verify both citation integrity and actual support for each claim; combine automated checks with domain review. | A real citation can accompany a false interpretation. Numbers, dates, qualifications, and context matter. Automated checking is fallible; reviewer identity and evaluation thresholds remain open. | Architecture, Acceptance, AGENTS |
| D011 | Return supported partial findings with explicit evidence gaps when sources are insufficient. | Useful supported information is preferable to invented completeness. Do not fill missing evidence with unsupported claims. | Architecture, Acceptance |
| D012 | Present credible conflicting evidence, even when it weakens the user's preferred argument. | Evidence quality takes priority over advocacy usefulness. Synthesis must explain disagreement and limitations. | Architecture, Acceptance |
| D013 | Start with a curated corpus with visible coverage and dates; defer live-web discovery. | A bounded collection makes source provenance, retrieval, and evaluation manageable within the pilot constraints. Breadth is intentionally limited. This does not decide whether testers can upload documents. | Scope, Sources, Architecture |
| D014 | Questions and saved research are private by default; public corpus documents may be shared. | Testers should not expose their research to peers merely by joining an organization. Organization scoping alone is insufficient for private records. | Architecture, Scope, Acceptance |
| D015 | Approve $100/month total pilot operating allowance. Stop new AI work at the allocated limit, alert before exhaustion, preserve existing research access, and seek an owner decision before extra spending. | Constrains costs for a free pilot while retaining access to completed work. Reserve non-AI operating costs before allocating the AI limit. Configuration is not complete; an AI Gateway cap alone cannot cap all infrastructure costs. | Risks, Architecture, Acceptance |
| D016 | Validate repeat use for real research tasks; weekly use is acceptable when the underlying work is not daily. | Adoption should match actual research cadence. No numeric retention, time-saved, or useful-answer-rate threshold has been approved. | PRD, Risks |
| D017 | Maintain this decision log whenever a build decision is made. | Preserve reasoning across sessions and keep the specifications aligned. Separate accepted decisions from proposals and unresolved questions. | AGENTS, README |

## September 10, 2026 — Repository workflow

### D018 — One repository for code and project context (approved)

Keep application code, ingestion, planning documents, and decision history in APEX. A single change can carry implementation, reasoning, and validation together; separate repositories would add coordination work without a current access or release-boundary requirement. The planned apps/packages layout remains unchanged.

### D019 — Project-wide Superpowers defaults, scoped by task (approved)

Use installed Superpowers skills by default throughout the repository. Root `AGENTS.md` maps features, architecture, debugging, testing, reviews, and completion to the appropriate skills. Routine documentation changes use lightweight consistency checks instead of code-testing ceremony. This preserves disciplined engineering while avoiding unnecessary process for prose edits. Existing user approvals remain valid.

Superpowers is an environment dependency, already available in the current session; this repository contains workflow instructions rather than a vendored plugin or machine-specific cache paths. New collaborators need the skills available in their agent environment. This change configures agent workflow only; application scaffolding (WBS 0.3) remains pending.

Affected files: `AGENTS.md`, `README.md`, and this log.

## Proposed details and open decisions

- Pilot organization/contact, policy issue, jurisdictional coverage, first research task, and exact output: open pending outreach.
- User uploads versus owner-managed corpus ingestion: open. The owner asked about usage tracking instead of selecting an upload policy.
- Export format, including whether copyable cited text suffices: explicitly open.
- Usage analytics: owner requested discussion of tracking. Proposed events are sign-ins, research requests, return visits, citation clicks, feedback, and estimated AI cost, without private research text in analytics. Event schema, consent/notice, access, and retention are not yet approved.
- Evaluation sample, domain reviewer, and numeric quality/usefulness thresholds: open. The proposed 16-of-20 useful-answer threshold was not adopted.
- Auth method: email/password versus magic link remains open; centralized Hono middleware is already required.
- Data access: Drizzle versus typed raw SQL remains open.
- Concrete model IDs, AI budget allocation, gateway enforcement verification, and key rotation: pending.
- Detailed hour-based build plan: pending. The old 46–68 day full-roadmap estimate does not represent the approved synthesis-first scope.
- Proposed build order: foundation and privacy/cost boundaries → small licensed corpus → retrieval → citation integrity and evidence-support verification → research UI → usage tracking and pilot checks. This is a planning recommendation, not a completed implementation plan.

## Budget reasoning

The $100/month allowance was accepted after discussing a small-pilot estimate: roughly five testers and 500 synthesis requests/month, $5–15 for hosting/storage/retrieval, $30–60 for synthesis/checks, and $15–25 for ingestion/evaluation/contingency. These are assumptions, not cohort commitments or vendor quotes. Paid data licenses, domain registration, development subscriptions, and human reviewer time are excluded. See `05-Risks.md` for token assumptions and pricing references.


## September 10, 2026 — Research interaction decisions

### D020 — Keywords and two synthesis levels (approved)

Keyword searches return related papers and an overall synthesis of included evidence; opening a paper provides a document-specific synthesis. Both expose exact supporting paragraphs and evidence checks. Rationale: match how the owner expects researchers to search while supporting both discovery and close reading. Question-first interaction is superseded as the primary flow. Display scope and included documents to avoid implying comprehensive coverage of all research.

### D021 — Full-text evidence for initial synthesis (approved)

Abstract-only records may appear in search results but are excluded from synthesis initially. Label access and explain insufficient full-text coverage. Rationale: abstracts omit methods and limitations needed for faithful interpretation. Tradeoff: fewer papers may contribute; coverage must be visible.

### D022 — Private saved papers and permitted downloads (requested MVP behavior)

Researchers can save papers privately and download where permissions allow, otherwise follow the source access link. Rationale: finding evidence should lead to retaining and reading useful papers. Free API access alone does not establish redistribution rights. Synthesis exports and uploads are separate open decisions.

### D023 — Preserve the future stance and decision features (confirmed roadmap)

After synthesis, keyword searches should support evidence-backed positions of people and organizations and a decision/action history. Rationale: these address support and policy evolution in APEX's mission. Reuse stable document versions and passage citations, but defer stance/event extraction and UI. Scientific disagreement in synthesis is not itself an actor's policy stance.

Affected specifications for D020–D023: PRD, MVP Scope, Architecture, Acceptance Criteria. Detailed implementation and revised estimates remain pending.


## September 11, 2026 — Evidence eligibility

### D024 — Two labeled evidence categories (approved)

Admit peer-reviewed research and separately labeled institutional research. Provide a peer-reviewed-only filter. Institutional reports require documented methods, data provenance, limitations, and an identifiable review process; institutional review must not be mislabeled as external journal peer review. Peer-reviewed studies still require evidence assessment. Rationale: include useful policy research such as Pew-style reports without weakening or obscuring the review standard. Full-text-only synthesis remains in force. Record eligibility at document level; API inclusion alone does not qualify a document. Detailed review workflow remains to be designed.

The September 11 infrastructure assessment is a recommendation, not approval to add Kafka, Docker, Kubernetes, GraphQL, gRPC, WebSockets, Redis, or additional Cloudflare services. See `docs/research/2026-09-11-source-and-infrastructure-review.md`.


### D025 — Indexed MVP, live discovery later (approved September 11, 2026)

Populate a versioned, searchable collection from selected APIs before user searches. Prioritize popular topics and adjust coverage based on user interest; defer live API discovery during searches. Rationale: reuse document preparation, make evidence review repeatable, and reduce request-time dependencies within the pilot budget. Tradeoff: coverage is bounded and freshness depends on refreshes; display both explicitly. Popularity determines which topics to cover, not which conclusions qualify for inclusion. Apply the same evidence eligibility rules to supporting and conflicting findings. Initial topics, demand measurement, refresh cadence, and expansion thresholds remain open. This resolves the indexed-versus-live discovery question and refines D013.

Affected documents: MVP Scope, Architecture, Sources, Risks, Acceptance Criteria.


### D026 — Show document results before synthesis (approved September 11, 2026)

Display document results as soon as retrieval completes, without waiting for synthesis. Researchers can browse and open papers while synthesis is prepared and checked. Show synthesis progress separately and render substantive claims only after verification. Rationale: keep research usable during generation without exposing unchecked output. Search completion does not guarantee instantaneous response. A synthesis failure must not remove successful search results. Initial topics remain open pending partner discussion.

Affected documents: MVP Scope, Architecture, Acceptance Criteria.


## September 29, 2026 — First validation user

### D027 — Aging & Cognition Research Group is the first validation user (approved)

Use the Aging & Cognition Research Group at the USC Leonard D. Schaeffer Institute for Public Policy & Government Service as APEX's first user for workflow discovery and prototype testing. The contact is one of its lead researchers; the name remains to be recorded. This supersedes the first-user portion of D007, which assumed the initial pilot would be a nonprofit or small organization in Los Angeles. The research group is a strong setting for testing source discovery, synthesis, citation verification, and researcher trust, but it does not by itself validate adoption or purchasing needs in APEX's primary nonprofit customer segment. Record its named contact, real research task, topic, participant count, and session date before fixing the pilot corpus. Detailed owner-provided discovery notes are in `docs/research/2026-09-29-aging-cognition-validation-user-profile.md`.

Affected documents: PRD, WBS, Risks, decision log.


### D028 — Build a local-first vertical slice before Cloudflare access (approved)

Build the first complete APEX evidence path locally with Wrangler/workerd, local D1, local R2, deterministic keyword retrieval, and extractive synthesis. Keep retrieval and synthesis behind provider interfaces so Vectorize and Anthropic through AI Gateway can replace the local providers without changing application contracts. Use 5–10 accessible documents about new and repurposed Alzheimer's drugs as the representative seed collection. This lets the team implement and test provenance, tenant isolation, retrieval, citation resolution, withholding, and interface states while USC IT approval for Cloudflare Workers remains pending. It does not authorize production deployment or treat local retrieval and synthesis quality as equivalent to the remote services.

Design: `docs/superpowers/specs/2026-09-29-local-first-vertical-slice-design.md`.

Affected documents: Architecture design, Risks, WBS, decision log.


### D029 — Start with GitHub CodeQL default setup (approved)

After the first TypeScript and Python application code is committed, enable CodeQL default setup for both `javascript-typescript` and `python` using the `security-extended` query suite. Review coverage after the first successful scan and adopt a repository-owned advanced workflow only if default setup lacks needed path, query, build, schedule, or runner control. Configure GitHub secret scanning and push protection separately because CodeQL does not detect committed credentials, and use Dependabot separately for dependency vulnerabilities and updates. This keeps initial security scanning current and low-maintenance while preserving a path to explicit workflow configuration when the monorepo provides evidence that it is needed.

Affected documents: WBS, local-first vertical-slice design, decision log.
