# APEX — Risks, Assumptions & Open Questions (MVP v1)

Status: Draft
Owner: [Kaia]
Last updated: 2026-09-10

## Top Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | **Hallucinated citations undermine the core promise** | Medium | Critical | Retrieval-grounded generation + mandatory post-generation verification step; eval harness in Phase 3; every claim renders a source link or is withheld |
| R2 | **Data source access/licensing constraints** (what can be summarized/redistributed) | Medium | High | Source terms review in Phase 1; prefer APIs with clear reuse terms ([CONFIRM] source list); summary-only + link approach to reduce legal surface |
| R3 | **LLM/embedding cost exceeds free-tester budget** | High | Medium | **AI Gateway spend limits (soft 80% / hard 100% block) + response caching + rate limits**; cheap Haiku-class for extraction; token/cost accounting built into the gateway; $100/month total approved; AI allocation/enforcement pending |
| R4 | **Tester recruitment too slow** (non-profits/advocacy are busy) | Medium | High | Start recruitment early (parallel with Phase 4); low-friction sign-in; offer usable artifact (brief/one-pager) as incentive |
| R5 | **Scoped-data violations slipping in (tenant isolation)** | Low | Medium | Guardrail acceptance checks: every endpoint scoped by tenant; code review + CI check |
| R6 | **Secret leak via committed `.env` or frontend key usage** | Low | Critical | Secrets only in env; `.env.example` only; CI secret-scan; never call LLM vendor directly from browser |
| R7 | **RAG quality too low to impress researchers** | Medium | Medium | Golden Q&A set; iterate on chunking/retrieval before testing real users; show evidence links over claims |
| R8 | **Stance/precedent data not trustworthy enough for real strategy work** | Medium | High | Scope test issues narrowly; show evidence + dates; label confidence/coverage limits in UI |
| R9 | **Scope creep (all 3 features want depth)** | Medium | Medium | 02-MVP-Scope deferral list governs; anything beyond v1 definitions goes to list |
| R10 | **Shared API keys discovered/leaked via chat, screenshots, commits** | Medium | High | One production key was pasted this session — rotate it; keys live only in `.env`; `wrangler secret put`/Workers secrets; CI secret-scan; never paste live keys into chats or docs (Guardrail 2) |

## Assumptions

- A1: Open-access sources are sufficient to seed a useful MVP corpus.
- A2: Non-profit researchers will adopt a free tool if it demonstrably saves prep time.
- A3: A small cohort (~`[CONFIRM]` testers) can validate the workflow meaningfully.
- A4: Managed services keep the MVP buildable solo without dedicated infra work.
- A5: Sentence-level citation rendering is technically feasible within cost ceiling.
- A6: The MVP-core sources allow metadata + quote-level excerpts + links (full-text where licensed) — confirmed during licensing review (WBS 1.2).

## Open Questions ([CONFIRM] list — resolve before Phase 1/3 go-live)

| # | Question | Default waiting on |
|---|---|---|
| Q1 | ~~Exact source list?~~ ✅ **Resolved** — 13 MVP-core classes confirmed; see [08-Sources.md](08-Sources.md). Licensing/redistribution review is WBS Phase 1 (1.2). | Done |
| Q2 | ~~LLM providers + embedding provider?~~ ✅ **Resolved (2026-09-10)** — Anthropic Claude: Sonnet-class for synthesis, Haiku-class for extraction; embeddings = Workers AI `bge-m3`. Concrete model IDs/key rotation pending (owner). | Done |
| Q3 | ~~Monthly AI cost ceiling?~~ ✅ **Mechanism resolved (2026-09-10)** — Cloudflare AI Gateway spend limits: soft alert 80%, hard block 100%. **Owner approved $100/month total pilot operating allowance (2026-09-10).** AI-only allocation and enforcement configuration remain TBD. | Partial |
| Q4 | ~~Vector store?~~ ✅ **Resolved** — Cloudflare Vectorize (GA). | Done |
| Q5 | ~~Deploy/hosting provider?~~ ✅ **Resolved** — Cloudflare (Workers + Pages, D1, Vectorize, R2, Workers AI, AI Gateway). | Done |
| Q6 | ~~Backend framework?~~ ✅ **Resolved** — Hono + TypeScript on Cloudflare Workers. | Done |
| Q7 | ~~Frontend + charting?~~ ✅ **Resolved** — Vite + React + Tailwind + shadcn/ui + React Query + Recharts. | Done |
| Q8 | First pilot organization/contact in Los Angeles, policy issue, cohort size, and sign-up channel? Owner needs to speak with a prospective organization first. | Open — owner outreach by September 24, 2026; grilling Q6 |
| Q9 | Shipping deadline: **November 3, 2026**, independent of an election-related workflow. Pilot start and testing window remain open. | Deadline resolved; testing window TBD |
| Q10 | Repeat use for real research tasks; weekly use is acceptable when research is not daily. Cohort, observation window, retention threshold, and time-saved target remain TBD. | Direction resolved; numeric criteria open |
| Q11 | First research task and output: question answer, literature summary, cited brief, or another deliverable? Resolve with the pilot organization before fixing corpus and workflow scope. | Open — owner discovery; grilling Q7 |

## Remaining discovery questions

- Tester document uploads versus owner-managed ingestion remain open (grilling Q19).
- Export format and whether copyable cited text is sufficient remain open (grilling Q20).
- Numeric useful-answer-rate thresholds are deferred; the proposed 16/20 threshold was not adopted (grilling Q22).
- Usage tracking is requested for planning: sign-ins, research requests, repeat use, citation clicks, feedback, and estimated AI cost. Proposed analytics omit private question/answer text; retention and implementation remain to be decided.

## Owner decisions — grilling session, 2026-09-10

- Evidence quality takes precedence over the shipping date: reduce coverage or delay the pilot if material unsupported claims remain.
- First release prioritizes research synthesis. Stance Tracker and Action Tracer are later priorities; scope and acceptance docs now exclude the deferred capabilities from the first release gates.
- Evidence verification must check that the source supports the claim, including qualifications and context, not merely that the citation exists. Withhold unsupported claims.
- Quality assurance combines domain review of real question/answer samples with layered automated checks. Reviewer identity, evaluation thresholds, and exact checks remain open; more checks alone do not establish accuracy.
- Owner availability: **10 hours per week**, approximately **77 hours** from September 10 to November 3. Allocate time for outreach, evidence review, and feedback as well as development.
- November 3 is a shipping deadline, not a requirement to support an election-specific task.
- When evidence is insufficient, return supported partial findings and explicitly identify evidence gaps; do not fill gaps with unsupported claims.
- Owner will speak with a potential pilot organization within the next two weeks, by September 24, 2026.
- First release answers from a curated collection with visible coverage and dates. Live-web discovery is deferred.
- Synthesis must present credible conflicting evidence and relevant limitations, including findings that weaken the user's preferred argument.
- Tester questions and saved research are private by default; public source documents may be shared. A single shared organization must not expose individual research to other testers.
- When the approved operating allowance is exhausted, stop new AI requests while keeping existing research accessible. Alert before exhaustion; additional spending requires an owner decision. Enforcement must account for non-AI operating costs and reserve enough budget for continued read access; the exact allocation and mechanism remain implementation decisions.

## Approved pilot operating allowance — September 10, 2026

- Owner-approved operating allowance: **$100/month**, with an estimated **$50–100/month** for a small, curated pilot. This is an estimate, not a configured spending limit.
- Assumptions: approximately five testers, 500 synthesis requests/month, bounded retrieved context, one synthesis and one evidence-check pass per request, and a modest seed corpus.
- Allow $5–15/month for Cloudflare hosting/storage/retrieval, $30–60 for generation and verification, and $15–25 for ingestion, evaluation runs, and contingency.
- Conservative token assumption: 20,000 aggregate input tokens and 2,000 aggregate output tokens across both passes per request. At a planning rate of $3/million input and $15/million output, 500 requests cost approximately $45. Actual costs depend on selected models, context size, retries, and evaluation volume.
- Excludes paid source licenses, domain registration, development-tool subscriptions, and human reviewer time. Confirm commercial source access before depending on it.
- Pricing references checked September 10: [Cloudflare Workers](https://developers.cloudflare.com/workers/platform/pricing/), [Anthropic pricing](https://claude.com/pricing). Concrete model IDs, AI-only allocation, and enforcement configuration remain unresolved (Q2/Q3).

## Risk reviews

Next formal review: after Phase 1 (M1). Update R1/R2/R3 with real source + cost data.