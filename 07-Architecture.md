# APEX — Architecture (MVP v1, Detailed Technical Spec)

Status: **M0 locked (2026-09-10)** — stack + core decisions confirmed; build-phase `[TBD]`s remain
Owner: [Kaia]
Last updated: 2026-09-10

Current release baseline (2026-09-10): synthesis over a curated corpus first; Stance Tracker and Action Tracer are deferred. Shipping target November 3, 2026, subject to evidence quality; owner availability 10 hours/week; total operating allowance $100/month. See [decision log](docs/decisions/README.md).
Ref: PRD (01), WBS (03), Risks (05), Acceptance (06)

## Conventions used

- `[CONFIRM]` = decision required (from 05-Risks Q-list).
- `[TBD]` = fill in during build phase.
- A doc is "current" only after the `[CONFIRM]` items in its section are resolved; unresolved ones move to ADR-lite tickets.

---

## 0. System Overview (component diagram)

```
                        ┌──────────────────────────────────────────────┐
                        │                  FRONTEND (SPA)              │
                        │  Search UI · Summary UI · Pro/Con Dash ·    │
                        │  Timeline · Callout: citations/link chips    │
                        └──────────────────┬───────────────────────────┘
                                           │ HTTPS (JSON)
                        ┌──────────────────▼───────────────────────────┐
                        │        API — CLOUDFLARE WORKERS (Hono/TS)    │
                        │  Auth middleware (single wrapper, LOCKED)    │
                        │  Endpoints: search · summarize · document ·  │
                        │  stances · events · session                  │
                        └───────┬───────────────────┬──────────────────┘
                                │                   │
              ┌─────────────────▼────┐   ┌──────────▼───────────────┐
              │  AI / RAG            │   │     DATA (Cloudflare)    │
              │  AI Gateway (cost,   │   │  D1 — relational, tenant │
              │   caching, fallback) │   │  Vectorize — embeddings  │
              │  Workers AI: embed   │   │  R2 — raw docs / PDFs    │
              │   (bge-m3) + cheap   │   │  Cron/R2 → ingestion     │
              │   inference          │   └──────────────────────────┘
              │  Claude (Anthropic): │
              │   Sonnet synthesis · │
              │   Haiku extraction   │
              └──────────────────────┘
```

Guaranteed boundaries (non-optional):

- **No LLM keys in browser.** All model calls are proxied server-side (Worker → AI Gateway).
- **Auth is a middleware wrapper**, not per-route code (Hono middleware).
- **All writes follow the tenant-scoping convention** (Guardrail 1).

---

## 1. Data Engineering & Ingestion Pipelines

### 1.1 Source catalog

**Q1 RESOLVED (2026-09-01):** candidate source set confirmed by owner — see **[08-Sources.md](08-Sources.md)** (full registry with access details, licensing notes, tiers, and rationale per source).

Candidate pool for selecting the curated synthesis seed (not an all-source M1 requirement): Library of Congress A–Z · PMC/PubMed BioC · OpenAlex · Oxford ORA · Crossref (Sage) · owner's compiled "Excel with Articles" · Congress.gov · GovInfo · PolicyNote (incl. CQ) · Senate Lobbying Disclosure (lda.gov) · CourtListener · Caselaw Access Project · Regulations.gov · Federal Register.

| Source class | Candidate sources (tier) | Access | Reuse posture |
|---|---|---|---|
| Research / scholarly | PMC, OpenAlex, ORA, Crossref, CORE, arXiv, Wiley SRU, NASA Techport (MVP-core + extend) | API / OAI-PMH / bulk | License per source; default link + quote-level excerpt (details in 08-Sources) |
| Congressional / legislative | Congress.gov, GovInfo, PolicyNote (CQ), Congress Press, LDA (core); Open States (future) | API + bulk | Terms per source; keys in `.env` only |
| Courts / regulatory | CourtListener, Caselaw Access Project, Regulations.gov, Federal Register (core) | API / bulk | CourtListener citation-lookup for false-citation check; link + quote where not OA |
| Gov data | data.gov, Federal Register (cross-cutting) | API | Open |

Rule for every source: link + attribution rendered with citations; full-text redistribution only where license allows. Where full text cannot be stored, store metadata + quotes + link.

### 1.2 Ingestion pipeline

```
Sources --(fetch)→ raw store --(parse PDF/HTML→text)--> normalized text
   --(chunk, sentence-aware)--> chunks
   --(embed)--> vectors -> Vector DB
   --(structured fields)→ documents + versions rows -> D1
   --(dedup/version)--> version table integrity
```

Workflow:
1. **Fetch** (`[TBD]` framework: e.g. Python + httpx / Airflow-lite triggers).
2. **Normalize:** PDF/HTML → text with layout + section preservation; extract metadata (jurisdiction, doc type, date, entities, gov action types).
3. **Dedup & version:** content-hash; store previous versions; keep source canonical, expose `latest` view.
4. **Chunk:** sentence-aware chunking with `[TBD]`-token overlap; keep per-chunk source span anchors (needed for 3.2 citation grounding).
5. **Embed + index:** Workers AI **bge-m3** embeddings → **Vectorize** (LOCKED); store chunk ↔ source span map.

### 1.3 Ingestion scheduling vs triggers

- Initial seed: one-shot bulk load.
- Ongoing: `[TBD]` — scheduled crawl (e.g. daily/nightly) or event-driven webhook where source provides it.
- MVP constraint: seed-first, refresh-later. Don't build a scheduler until M2 proves value.

### 1.4 Data warehouse/lake

None for MVP. Skip lake/lakehouse, use D1 + Vectorize + R2. Note if corpus > `[TBD]`GB, revisit.

---

## 2. AI Engineering & RAG Architecture

### 2.1 Retrieval layer

- **Embedding model:** **Workers AI `@cf/baai/bge-m3`** (LOCKED; multilingual, solid retrieval for policy text).
- **Vector store:** **Cloudflare Vectorize** (LOCKED) — globally distributed; GA; ~$0 within free-tier for MVP corpus. If corpus scales far past MVP, pgvector/Pinecone re-evaluation noted in §5.5 (not a blocker).
- **Retrieval design:** hybrid (dense vector + keyword/BM25-style fallback) `[TBD]`; top-k retrieval with rerank option `[TBD]` (reranker available via Workers AI `bge-reranker-base` if quality demands).
- **AI Gateway presence:** model calls use AI Gateway. D1, R2, and Vectorize access use native Worker bindings; retrieval is not itself an LLM gateway call.

### 2.2 Generation — grounded RAG

- **LLM:** **Anthropic Claude (LOCKED)** — **Claude Sonnet-class for synthesis** (user-facing summaries; best structured-JSON reliability for the claims/citations contract), **Claude Haiku-class for extraction** (stances, events, metadata — high-volume, cheap). Bound context and measure actual token usage; pricing and caching eligibility depend on the concrete model and request. Embeddings stay on Workers AI (bge-m3), so Anthropic is only called for generation.
- Pattern: **retrieve → inject retrieved spans as context → generate with citation anchors** that the UI renders as `[n]` → link to span/URL. No free-form generation outside retrieved spans.
- Sentence-level citation: model emits every claim tagged to one or more spans; post-processor maps to exact source passage + URL.

### 2.3 Verification step (mandatory, non-optional)

- Check citation IDs, immutable document version, exact source spans, and source URLs against retrieved context.
- Separately assess whether evidence supports each claim, including numbers, dates, qualifications, and contradictory findings. Text overlap alone is insufficient.
- Withhold unsupported claims; return supported partial findings and explicit evidence gaps.
- Combine automated checks with domain review. Model checks are fallible; no structural citation check guarantees truth. Evaluation thresholds and reviewer remain open.

### 2.4 Eval harness

- Golden set: `[TBD]` hand-curated Q&A + highlight extractions with expected citations.
- Metrics: **hallucination rate (goal 0)**, citation recall/precision, summary factuality (LLM-as-judge + human spot-check `[TBD]`).
- Ran on every material prompt/retrieval change; gates Phase 3 done (see 06).

### 2.5 Cost control (free-tester budget)

- **Enforced via Cloudflare AI Gateway spend limits (LOCKED):** dollar-based budgets scoped by provider/model — soft alert at 80% of ceiling, hard block at 100%. Total operating allowance is **$100/month**. AI allocation and enforcement configuration remain open; reserve hosting/read-access costs before setting an AI cap.
- AI Gateway provides token/cost accounting per request and response caching (cache entries must respect private user ownership).
- Rate limits per tester session; prefer Haiku-class for extraction and reserve Sonnet for synthesis only.
- Monthly rollups reviewed at 05-Risks R3 tracking.

### 2.6 Stance & event extraction (capabilities 2 & 3 data layer)

- Same RAG pipeline reused: retrieve relevant statements/actions → stance classification `[TBD]` (for/against/neutral with confidence) → store in `stance_records` (see 3.2) with evidence citation.
- Event extraction: regulatory/court/policy events → normalized `events` rows with dates, actors, source docs.

---

## 3. Backend & Database Architecture

### 3.1 API framework

**LOCKED: Hono (TypeScript) on Cloudflare Workers.** Decision date 2026-09-10.

| Considered | Verdict |
|---|---|
| **Hono + TypeScript (Workers)** | Selected: edge-native, ~$0 free tier, native D1/Vectorize/R2/Workers AI/AI Gateway bindings, single language across frontend + API + pipeline; matches operator's proven stack (pnpm monorepo → wrangler) |
| FastAPI on Python Workers | Viable (Cloudflare supports FastAPI via ASGI on Python Workers) but open-beta WASM runtime; constrains the Python ecosystem advantage; two type systems. Revisit only if heavy Python data processing enters the request path |
| FastAPI on VPS/PaaS | Original doc default; rejected at M0: adds paid infra + candid ops that Cloudflare provides free |

**Python escape hatch:** ingestion/normalization may run as a separate `pipeline/` Python tool invoked from `scripts/` (local/CI), out of the edge request path — keeps strong PDF/ML libraries available without touching the API runtime.

Type-safety across API⇄frontend via Hono RPC types (server route types inferred on the client) or shared `packages/schema` types.

### 3.2 Draft relational schema

Single **Cloudflare D1** database (SQLite, LOCKED; read-replication scales for the tester cohort). Access layer: `drizzle-orm` or typed raw SQL (build-phase pick) — SQLModel is Python-only and not used. Embeddings live in **Vectorize**, not in D1. Tenant columns `user_id`, `organization_id` on all user-created rows (Guardrail 1). Public corpus records may be shared. Questions, summaries, claims, citations, and saved research must inherit authenticated user ownership; organization membership alone does not grant access to another tester’s research.

```
users(id, email, display_name, role DEFAULT 'tester', created_at)
organizations(id, name, created_at)                       -- one org in MVP
memberships(id, organization_id, user_id, role, created_at)

documents(id, source_id, source_type, title, jurisdiction, doc_kind,
          published_at, version, content_full OR content_ref, metadata_json,
          created_by_user_id, organization_id, created_at)
chunks(id, document_id, seq, text, span_start, span_end, vector_id, corpus_scope)
versions(id, document_id, content_hash, changed_at)

queries(id, user_id, organization_id, q_text, results_json, created_at)   -- private research
usage_events(id, user_id, event_type, request_id, estimated_cost, created_at) -- no research text
summaries(id, user_id, organization_id, query_id, status, created_at)
summary claims / citations:
  claims(id, summary_id, text, sentence_idx)
  citations(id, claim_id, chunk_id, document_id, source_url, span_text, veracity)
stances:
  stance_records(id, entity_id, issue_tag, stance, confidence, evidence_citation_id,
                 acted_on_at, created_by_user_id, organization_id)
  stance_revisions(id, stance_record_id, stance, confidence, changed_by, changed_at)
events(id, doc_kind, event_type, occurred_at, actor, title, description_url,
       created_by_user_id, organization_id, related_document_ids, timeline_metadata)

feedback_tasks (tester loop): (id, tester_user_id, task_type, started_at, completed_at,
       claimed_time_saved, org_id)
```

Index notes: `documents(jurisdiction, published_at)`, `claims(summary_id)`, `stance_records(entity_id, issue_tag)`, `events(occurred_at)`, `queries(user_id, created_at)`. All queries scoped `organization_id = ?` / `user_id = ?` per tenant rule.

### 3.3 API endpoints (draft list)

| Method+Path | Purpose | Notes |
|---|---|---|
| `POST /auth/register`, `POST /auth/login`, `GET /auth/me` | Tester sign-in (wrapper) | Modular auth; `/logout` etc. as needed |
| `GET /search?q=&filters=` | Semantic search over corpus | Hybrid retrieval; tenant-scoped |
| `GET /documents/{id}` | Document detail + source link | Version-aware |
| `POST /summarize` | Grounded synthesis w/ citations | Runs verification pass; returns claims + citations |
| `POST /search/summarize` | Instant summary of results | Cached where possible |
| `GET /stances?entity=&issue=` | Pro/Con dashboard data | Evidence drilldown payload |
| `GET /events?issue=&sent` | Timeline data | Entity/jurisdiction filter |
| `POST /feedback/tasks` | Tester task start/complete + time-saved | Metrics |
| `GET /admin/metrics` | Time-saved + usage rollups | Phase 7 |

### 3.4 Secrets & config

- `.env` (git-ignored) + `.env.example`; keys: vendor LLM/embedding keys, DB URL, platform secret.
- CI secret-scan; nothing deployment-side committed; server-to-server only.
- No vendor key ever shipped to browser bundle. (Guardrail 2.)

### 3.5 Modular auth middleware

- One Hono middleware wrapper in the API Worker (LOCKED).
- MVP: basic email/password (or magic link) sessions `[CONFIRM]` build-phase detail.
- Contract: routes declare `requires_auth`; switching to SSO/enterprise = one middleware change, no per-route edits. (Guardrail 3.)

---

## 4. Frontend Engineering & Data Visualization

### 4.1 Stack

- Framework: **Vite + React SPA** (confirmed 2026-09-10). Hono owns the API; no Next.js runtime or server rendering is required.
- Styling: **Tailwind CSS** (LOCKED).
- Components: **shadcn/ui** (LOCKED).
- Data fetching/state: **React Query (TanStack Query)** (LOCKED).
- Charting: **Recharts-style declarative** (LOCKED); D3 only if timeline demands bespoke.

### 4.2 Pages/components

| Page | Components |
|---|---|
| Search | Search box, filter panel (jurisdiction/type/date `[tbd]`), result list, doc detail drawer |
| Document view | Title/meta, full text ref, **citation chips** → anchor to source span + URL open |
| Summaries | AI takeaway cards; each claim renders `[n]` citation links; "show evidence" expanders; verification badge |
| Pro/Con Dashboard | Entity × issue matrix; for/against/neutral counts; evidence drilldown modal; date/action support |
| Action Tracker | Timeline (event nodes: date, type, actor, doc link); filters: entity/jurisdiction/issue |
| Onboarding/tester | Light sign-in; sample task prompts; feedback mini-form |

### 4.3 Data flow (UI ⇄ API)

- Search/summary/stances/events all call API endpoints; server does ALL grounding; UI is presentation-only.
- Citation anchor payload shape (from `POST /summarize`):

```json
{
  "claims": [
    {
      "text": "The bill would expand ..., ",
      "sentence_index": 3,
      "citations": [
        {
          "n": 1,
          "document_id": "doc-318",
          "source_url": "https://...",
          "span_text": "...exact passage...",
          "veracity": "verified"
        }
      ]
    }
  ]
}
```

- If `veracity != verified` → claim withheld from UI (Guardrail 4).

---

## 5. Deployment & Ops (MVP)

- Hosting **LOCKED: Cloudflare** — Workers (API) + Pages/Static Assets (frontend) / Worker Assets; D1, Vectorize, R2, Workers AI, AI Gateway all on the same account. No VPS/PaaS.
- Observability: AI Gateway logs/metrics for AI; Worker logs + D1 analytics; structured logs + token cost rollups via AI Gateway. Uptime not critical in testing window.
- CI: `wrangler deploy` via GitHub Actions (or Workers Builds); `.env.example` + `wrangler secret put` for keys.
- Cost budget: $100/month total approved. Stop new AI requests at the allocated limit, preserve existing research access, and require owner approval for more spending. Verify gateway enforcement capabilities during setup; do not assume an AI limit caps all Cloudflare charges.

## 5.5 Decision framework — how these defaults were derived

Every recommendation in this doc is derived from engineering principles and the product's own constraints, weighed in priority order:

1. **Product guarantee** — citation integrity is the differentiator; any choice that weakens "never hallucinate" is rejected outright.
2. **Correctness of the citation contract** — the pipeline must be able to render claim → exact source span → URL with structural guarantees, not just prompt-level instruction.
3. **Operational simplicity** — the MVP is a small team on a free budget; fewer systems = fewer failure modes, less to secure, back up, and monitor.
4. **Maintainability** — prefer mature, actively maintained, widely deployed components: abundant documentation, fewer exotic bugs, and working knowledge that outlasts the MVP.
5. **Cost discipline** — free product with no revenue; AI spend dominates and is capped.

### Technology maturity evidence

The selected design uses Hono/TypeScript on Workers, Vite + React, D1, Vectorize, and R2. Native platform bindings reduce integration work; retrieval-grounded generation and verification remain separate application responsibilities.

### Sharpening decisions

- **Frontend conventions:** Tailwind + shadcn/ui + React Query — broadly used conventions with strong documentation; fast to build and debug (§4.1).
- **ORM:** Drizzle or typed raw SQL remains open. Both must support D1 migrations and explicit user scoping (§3.2).
- **Core engineering surface:** citation-verified synthesis + eval harness + cost-cap loop is the product's hardest engineering requirement — build it as a first-class subsystem (WBS Phase 3), not an afterthought.
- **Async seam:** long-running AI calls should eventually move to a background-job queue; documented as a seam, kept OUT of the MVP build, added if tester latency demands it.

---

## 6. Decision register (ADR-lite)

Canonical decision history and rationale: [decision log](docs/decisions/README.md). Historical alternatives below are retained for context; current choices follow that log.

Every decision below states its **rationale** — so the "why" is captured, not just the "what". Status: `Done` (locked) or `Recommended` (my default; awaiting your confirmation).

### 6.1 Resolved

| Date | Decision | Rationale | Status |
|---|---|---|---|
| 2026-09-01 | Documentation-first approach (PRD/WBS/Arch/Sources) | Plan before code — cheap to change now, expensive later | Done |
| 2026-09-01 | **Q1:** source set = 13 MVP-core classes in 08-Sources.md | Coverage for all 3 capabilities with free/open APIs first; CourtListener's built-in citation checker directly supports the no-hallucination promise | Done (candidate set; licensing review pending in Phase 1) |
| 2026-09-01 | Citations: claim → source span → URL, verification pass mandatory | The #1 differentiator vs generic AI (no fabricated refs); enforcement is in the pipeline, not the prompt | Done |

### 6.2 Confirmed at M0 (2026-09-10) — each with rationale; framework in §5.5

| # | Decision | Why this choice (rationale) | Alternative considered | Status |
|---|---|---|---|---|
| Q4 | Vector store: **Cloudflare Vectorize** (+ D1 for relational) | GA, globally distributed, ~$0 within free tier for MVP corpus; single platform for embeddings + queries, no egress; native Worker binding. Scale path: pgvector/Pinecone if corpus outgrows. | Postgres+pgvector: fine but spins up managed PG; D1 covers relational data + tenant scoping | **Done** |
| Q2 | **Claude (Anthropic) — Sonnet-class for synthesis, Haiku-class for extraction; embeddings bge-m3 via Workers AI** | Best structured-JSON reliability for the claims/citations contract; Model-dependent pricing and caching; measure cost before selecting IDs. Cheap embeddings stay in-platform. Concrete model IDs + key `[TBD]` (owner). | Bare Workers AI open models for synthesis: cheap but weaker instruction-following for citation JSON | **Done** |
| Q3 | **Cost ceiling via AI Gateway spend limits** — soft alert 80%, hard block 100% | AI Gateway provides dollar-based budgets scoped by provider/model; one config block instead of custom code. $100/month total approved; AI allocation and enforcement TBD. | No limit: risky for a free prototype | **Total approved; allocation TBD** |
| Q6 | API: **Hono (TypeScript) on Cloudflare Workers** | Edge-native, ~$0, native bindings, one language across frontend+API+pipeline; matches operator's proven stack. | FastAPI (VPS or Python Workers): constrains to paid infra or beta WASM runtime | **Done** |
| Q5 | Hosting: **Cloudflare** (Workers/Pages, D1, Vectorize, R2, Workers AI, AI Gateway) | Free tiers cover MVP; scaling = hand off to the edge; zero infra hobby. | Fly.io/Render/Railway VPS: paid, manually scaled, outside operator's skill stack | **Done** |
| Q7 | Charting: **Recharts-style declarative** | Pro/Con dashboard + timeline are standard charts; declarative lib is fast to ship and React-native. | D3: full control, overkill for MVP views | **Done** |
| 2.1 | Retrieval: **hybrid dense + keyword fallback** (rerank optional via bge-reranker-base) | Dense embeddings catch semantics; keyword catches exact citations/numbers; combined recall protects citation-groundedness. | Dense-only: simplest, misses exact-match edge cases | **Recommended** |
| 2.2 | Generation: **retrieve → grounded generation with span anchors**; never free-form beyond retrieved spans | Structural citation traceability; semantic support requires separate checks; UI renders `[n]`→span→URL. | Agentic open search: flexible but unpredictable citations | **Done (pattern locked)** |
| 3.2 | DB access: **Drizzle-ORM or typed raw SQL** (build-phase pick) | No Python ORM in a TS stack; typed schema + migrations for D1. | SQLModel: Python-only, not applicable on Workers | **Recommended (build-phase)** |
| 4.1 | Frontend conventions: **Tailwind + shadcn/ui + React Query** | Mature, documented, widely used UI stack; declarative and fast to build/debug. | Custom CSS / bespoke state: slower, less discoverable | **Done** |

### 6.3 Open (deferred)

| Date | Open item | Why deferred |
|---|---|---|
| — | Q8 tester cohort (who/how many/channel) | Needs product decisions, not architecture |
| — | Q9 testing window (shipping deadline November 3, 2026) | Needs schedule decision |
| — | Q10 time-saved target metric (e.g. ≥X%) | Needs baseline data from a small dry-run |

## 7. Decisions checklist (mirrors 05-Risks Q list)

- [x] Q1 source list ✅ (13 MVP-core classes; see 08-Sources.md — licensing review pending in Phase 1)
- [x] Q2 LLM + embedding providers ✅ (Anthropic Claude Sonnet/Haiku + Workers AI bge-m3; **concrete model IDs + key rotation pending**)
- [ ] Q3 AI cost ceiling ✅ mechanism (AI Gateway spend limits: soft 80% / hard 100%) — **$100/month total approved; AI allocation/enforcement TBD**
- [x] Q4 vector store ✅ (Cloudflare Vectorize)
- [x] Q5 hosting provider ✅ (Cloudflare — Workers/Pages/D1/Vectorize/R2/Workers AI/AI Gateway)
- [x] Q6 API framework ✅ (Hono + TypeScript on Cloudflare Workers)
- [x] Q7 frontend + charting ✅ (Vite + React + Tailwind + shadcn/ui + React Query + Recharts)
- [ ] Q8 tester cohort
- [x] Q9 shipping deadline November 3, 2026; testing window remains open
- [ ] Q10 time-saved target metric