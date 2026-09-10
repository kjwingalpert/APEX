# APEX — Risks, Assumptions & Open Questions (MVP v1)

Status: Draft
Owner: [Kaia]
Last updated: 2026-09-01

## Top Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | **Hallucinated citations undermine the core promise** | Medium | Critical | Retrieval-grounded generation + mandatory post-generation verification step; eval harness in Phase 3; every claim renders a source link or is withheld |
| R2 | **Data source access/licensing constraints** (what can be summarized/redistributed) | Medium | High | Source terms review in Phase 1; prefer APIs with clear reuse terms ([CONFIRM] source list); summary-only + link approach to reduce legal surface |
| R3 | **LLM/embedding cost exceeds free-tester budget** | High | Medium | Token/request logging + ceiling alert (Phase 3.5); cache summaries; rate limits; `[CONFIRM]` ceiling |
| R4 | **Tester recruitment too slow** (non-profits/advocacy are busy) | Medium | High | Start recruitment early (parallel with Phase 4); low-friction sign-in; offer usable artifact (brief/one-pager) as incentive |
| R5 | **Scoped-data violations slipping in (tenant isolation)** | Low | Medium | Guardrail acceptance checks: every endpoint scoped by tenant; code review + CI check |
| R6 | **Secret leak via committed `.env` or frontend key usage** | Low | Critical | Secrets only in env; `.env.example` only; CI secret-scan; never call LLM vendor directly from browser |
| R7 | **RAG quality too low to impress researchers** | Medium | Medium | Golden Q&A set; iterate on chunking/retrieval before testing real users; show evidence links over claims |
| R8 | **Stance/precedent data not trustworthy enough for real strategy work** | Medium | High | Scope test issues narrowly; show evidence + dates; label confidence/coverage limits in UI |
| R9 | **Scope creep (all 3 features want depth)** | Medium | Medium | 02-MVP-Scope deferral list governs; anything beyond v1 definitions goes to list |
| R10 | **Shared API keys discovered/leaked via chat, screenshots, commits** | Medium | High | One production key was pasted this session — rotate it; keys live only in `.env`; CI secret-scan; never paste live keys into chats or docs (Guardrail 2) |
| R10 | **Shared API keys discovered/leaked via chat, screenshots, commits** | Medium | High | One production key was pasted this session — rotate it; keys live only in `.env`; CI secret-scan; never paste live keys into chats or docs (Guardrail 2) |

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
| Q2 | LLM providers + embedding provider? (recommended: Haiku-class for extraction, flagship for synthesis — just pin providers) | Build decision |
| Q3 | Monthly AI cost ceiling for free MVP? (recommended: alert at 80% / pause at 100% — need a number) | Budget decision |
| Q4 | Vector store choice? (recommended: Postgres + pgvector) | Build decision |
| Q5 | Deploy/hosting provider for free testers? (Fly.io/Render/Railway) | Build decision |
| Q6 | Backend framework: FastAPI + React/Next SPA? (recommended) | Build decision |
| Q7 | Frontend + charting (Next.js + Tailwind + shadcn/ui + React Query + Recharts recommended)? | Build decision |
| Q8 | Tester cohort: who, how many, sign-up channel? | Product decision |
| Q9 | Target launch date + testing window length? | Schedule decision |
| Q10 | Success metric target: e.g. ≥ `[CONFIRM]`% claimed time saved? | Product decision |

## Risk reviews

Next formal review: after Phase 1 (M1). Update R1/R2/R3 with real source + cost data.