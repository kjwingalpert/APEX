# APEX — Work Breakdown Structure (MVP v1)

Status: Draft
Owner: [Kaia]
Last updated: 2026-09-01

Legend: `[P-Phase]` tasks are scoped interdependently within that phase. Estimates are day-effort units for a solo/small build (`[CONFIRM]` sizing). Owners default to [Kaia].

## Phase 0 — Foundation & Setup (Estimates: 3–5d)

| # | Task | Owner | Est. | Dependencies |
|---|---|---|---|---|
| 0.1 | Confirm MVP tech stack per 07-Architecture (resolve all `[CONFIRM]` items) | Kaia | 1d | — |
| 0.2 | Init git repo + GitHub; branch strategy (main + feature) | Kaia | 0.5d | 0.1 |
| 0.3 | Repo scaffolding: monorepo layout (`app/`, `api/`, `pipeline/`), lint/format, CI basics | Kaia | 1d | 0.2 |
| 0.4 | Secrets setup (.env pattern, `.env.example`, guard rails; verify nothing secrets committed) | Kaia | 0.5d | 0.3 |
| 0.5 | Tenant-scoping convention documented + schema template (all user tables carry `user_id`/`org_id`) | Kaia | 0.5d | 0.1 |

## Phase 1 — Data & Access Layer (Estimates: 4–6d)

| # | Task | Owner | Est. | Dependencies |
|---|---|---|---|---|
| 1.1 | Finalize source list: MVP-core set from 08-Sources.md (13 classes); fallback to MVP-extend only if needed | Kaia | 1d | 0.1 |
| 1.2 | Source licensing/terms review per 08-Sources licensing questions (what can be stored vs link-only) | Kaia | 1d | 1.1 |
| 1.3 | API/integration spike per source (Congress.gov/GovInfo, journal APIs, court filings) | Kaia | 2d | 1.1 |
| 1.4 | Auth middleware wrapper selected + stubbed (NextAuth / FastAPI middleware) | Kaia | 1d | 0.2 |

## Phase 2 — Ingestion & Indexing (Estimates: 6–8d)

| # | Task | Owner | Est. | Dependencies |
|---|---|---|---|---|
| 2.1 | Ingestion pipeline: fetch → normalize (PDF/HTML → text/structured) → store | Kaia | 3d | 1.3 |
| 2.2 | Dedup + document versioning; source metadata (jurisdiction, type, date, entity) | Kaia | 1d | 2.1 |
| 2.3 | Chunking strategy for RAG (sentence-aware, overlap tuned) | Kaia | 1d | 2.1 |
| 2.4 | Embeddings generation + vector store population | Kaia | 2d | 2.3 |
| 2.5 | Schema written with tenant columns; seed corpus loader | Kaia | 1d | 2.1, 0.5 |

## Phase 3 — Citation-Verified Synthesis (AI Core) (Estimates: 8–12d)

| # | Task | Owner | Est. | Dependencies |
|---|---|---|---|---|
| 3.1 | Retrieval pipeline: query → retrieve → ground LLM generation in retrieved spans | Kaia | 3d | 2.4 |
| 3.2 | Citation grounding design: claim ↔ source span ↔ URL mapping rendered in output | Kaia | 2d | 3.1 |
| 3.3 | Verification step: post-generation check that every citation resolves to a real retrieved span (no hallucinated refs) | Kaia | 2d | 3.2 |
| 3.4 | Eval harness: golden set of Q&A/highlights; hallucination + citation-recall metrics | Kaia | 2d | 3.3 |
| 3.5 | Cost monitoring: token/request logging vs `[CONFIRM]` ceiling | Kaia | 1d | 3.1 |

## Phase 4 — Semantic Search + AI Summaries (Frontend Core) (Estimates: 6–8d)

| # | Task | Owner | Est. | Dependencies |
|---|---|---|---|---|
| 4.1 | API endpoints: search, summarize, get-document, get-citations | Kaia | 2d | 3.3, 1.4 |
| 4.2 | Search UI: semantic search box + result list + document view | Kaia | 2d | 4.1 |
| 4.3 | Summary UI: AI takeaways with sentence-level citation chips/anchors + links | Kaia | 2d | 4.1 |
| 4.4 | Guardrail test: no unscoped query; scoped-by-tenant on every endpoint | Kaia | 0.5d | 4.1, 0.5 |

## Phase 5 — Pro/Con Stance Tracker (Estimates: 6–9d)

| # | Task | Owner | Est. | Dependencies |
|---|---|---|---|---|
| 5.1 | Stance extraction: analyze statements, legislative actions, historical arguments → stance labels with evidence | Kaia | 3d | 3.3 |
| 5.2 | Stance data model + storage (entity, issue, stance, evidence citation, date) | Kaia | 1d | 5.1 |
| 5.3 | Pro/Con visualization (dashboard): entity-by-issue stance counts + evidence drilldown | Kaia | 2.5d | 5.2, 4.2 |
| 5.4 | Support targeting: export/share view for lobbying strategy meeting | Kaia | 1.5d | 5.3 |

## Phase 6 — Action Tracer (Estimates: 5–7d)

| # | Task | Owner | Est. | Dependencies |
|---|---|---|---|---|
| 6.1 | Event model: regulatory/policy/legal events with dates, actors, documents, outcomes | Kaia | 1.5d | 2.2 |
| 6.2 | Precedent timeline generation from corpus (court decisions, regulatory actions) | Kaia | 2.5d | 6.1, 3.3 |
| 6.3 | Timeline visualization + filter by entity/jurisdiction/issue | Kaia | 2d | 6.2, 4.2 |

## Phase 7 — Tester Workflow & Feedback (Estimates: 5–7d, spans testing window)

| # | Task | Owner | Est. | Dependencies |
|---|---|---|---|---|
| 7.1 | Recruit tester cohort (non-profit researchers, advocacy groups) | Kaia | parallel | 4.3 |
| 7.2 | Light-touch sign-in for testers (free access; modular auth wrapper) | Kaia | 1.5d | 4.1 |
| 7.3 | Session guidance: real prep tasks (lobbying, testimony, coalitions) + time-saved survey | Kaia | 1d | 7.1, 7.2 |
| 7.4 | Usage analytics for time-saved baseline (task timers, task completion) | Kaia | 2d | 4.1 |
| 7.5 | Interview/survey synthesis → prioritized build feedback | Kaia | 2d | 7.4 |

## Phase 8 — Hardening & Release (Estimates: 3–5d)

| # | Task | Owner | Est. | Dependencies |
|---|---|---|---|---|
| 8.1 | QA against acceptance criteria (06) incl. citation-integrity pass | Kaia | 2d | all |
| 8.2 | Data cleanup, seed corpus refresh, secrets review | Kaia | 0.5d | 8.1 |
| 8.3 | Deploy + free-tier hosting check (`[CONFIRM]` provider) | Kaia | 1d | 8.1 |
| 8.4 | Tester launch + metrics dashboard live | Kaai | 0.5d | 8.3 |

**Totals:** 8 phases, ~46–68d effort (solo build path). Sizing `[CONFIRM]` — revisit after stack decisions.

## Open WBS decisions
- 0.1 stack confirmation gates Phases 1–3 estimates.
- 7.x assumes single-owner recruitment; adjust if a second person joins.
- 8.3 deploy provider drives cost/ops notes in 05–06.