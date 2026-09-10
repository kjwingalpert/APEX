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
