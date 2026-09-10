# APEX — Product Requirements Document (MVP v1)

Status: Draft
Owner: [Kaia]
Last updated: 2026-09-10

Current release baseline (2026-09-10): synthesis over a curated corpus first; Stance Tracker and Action Tracer are deferred. Shipping target November 3, 2026, subject to evidence quality; owner availability 10 hours/week; total operating allowance $100/month. See [decision log](docs/decisions/README.md).

## 1. Problem

An information gap exists between high-level research and its real-world application, disproportionately affecting small organizations and non-profits with limited staff and resources.

- **Resource and bandwidth constraints:** Non-profits and small teams must educate lawmakers, deliver expert testimony, and submit regulatory comments, but lack the bandwidth to manually sift through thousands of pages of academic papers, agency filings, and historical precedent.
- **Flawed workarounds & misinformation:** Many small teams rely on generic AI models (ChatGPT/Claude) for context, but these are not pre-loaded with comprehensive policy databases and frequently hallucinate or generate unverified citations, putting the organization's credibility at risk.

Every policy researcher must answer three questions for a policy to succeed:
1. **Is it correct?** — grounded in the best available evidence, ethical, and equitable.
2. **Is it administratively feasible?** — implementable with budget, staff, resources, skills.
3. **Will it get support?** — backed by decision-makers and affected stakeholders.

APEX exists to help answer all three faster and with verifiable evidence.

## 2. Target User

**Primary:** Non-profit researchers and policy teams at small advocacy organizations.

**Who they answer to:** legislators and staffers they lobby, expert testimony panels, regulatory comment dockets, and coalition partners.

**Jobs to be done (MVP validation targets):**
- Prepare for direct lobbying of policymakers.
- Prepare expert testimony.
- Drive evidence-based coalition building.

**Success = measurable time saved on these workflows** versus manual research and generic AI.

## 3. Core Capabilities (MVP)

### 3.1 Research Repository & Synthesis  [P0]
- Semantic search across a pre-loaded corpus of open-access academic, legislative, judicial, and agency documents.
- Instant AI summaries of search results and documents.
- Every takeaway includes verifiable, sentence-level citations linking claims to source spans.
- **Quality bar:** citations never hallucinate; every rendered claim maps to an exact source passage and link.

### 3.2 Stance Tracker (Pro/Con)  [Deferred]
- Analyzes public statements, legislative actions, and historical arguments.
- Visual Pro/Con dashboard quantifying where government entities and policymakers stand on an issue.
- Supports targeting direct lobbying, testimony, community data, and coalition building.

### 3.3 Decision & Action Tracer  [Deferred]
- Maps historical precedent of an issue: evolution of policies, court decisions, and regulatory actions over time.
- Basic timeline view of precedent and regulatory evolution.

## 4. Priority Tagging

| Requirement | Priority | Notes |
|---|---|---|
| Verifiable, sentence-level citations | P0 | Core differentiator vs. generic AI; do not ship without it |
| Semantic search across policy corpus | P0 | Foundation for all three capabilities |
| Instant AI summaries | P0 | Core value of repository capability |
| Pro/Con stance dashboard | Deferred | Validate with testers that stance data is usable for strategy |
| Precedent/regulatory timeline | Deferred | Basic version; depth can iterate |
| Tester cohort access (free sign-up) | P1 | Light-touch authenticated access; private research |
| Usage analytics for time-saved validation | P1 | Measure the success metric |
| Corpus expansion beyond seeded sources | P2 | Post-tester feedback |

## 5. Success Metrics

- **Primary:** Testers return for real research tasks; weekly use is acceptable when research is not daily. Collect time-saved feedback as a supporting measure; numeric targets remain open.
- **Quality:** 0 hallucinated citations in tested sessions; every claim renders a source link.
- **Engagement:** Tester cohort uses the product for real prep tasks, not demos.
- **Qualitative:** Non-profit researchers find stance and precedent views credible and actionable.

## 6. Non-Functional Requirements (MVP guardrails)

- **User/Org Data Isolation — always:** every database row created by a user carries a `user_id` and/or `organization_id`; all queries are scoped by tenant by default. (B2B isolation later = already correct SQL.)
- **Secrets hygiene:** API keys (Claude/OpenAI, any provider) live only in environment variables / managed secrets. Never in code, `.env` files committed, or frontend repos.
- **Modular auth:** authentication sits behind one centralized middleware/wrapper (Hono middleware). Swapping basic login for Enterprise SSO later must touch one module.
- **Cost-conscious:** MVP runs free for testers; LLM/embedding spend must stay under the approved $100/month total allowance; AI allocation and enforcement remain open.
- **Fast iteration:** decisions in `07-Architecture.md` default to managed services to keep the build small.

## 7. Out of Scope for v1

See [02-MVP-Scope.md](02-MVP-Scope.md).

Open items are surfaced in [05-Risks.md](05-Risks.md).