# APEX — Architecture (MVP v1, Detailed Technical Spec)

Status: Draft (many `[CONFIRM]` / `[TBD]` items await build decisions)
Owner: [Kaia]
Last updated: 2026-09-01
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
                        │              API / BACKEND                   │
                        │  Auth middleware (single wrapper)[CONFIRM]   │
                        │  Endpoints: search · summarize · document ·  │
                        │  stances · events · session                  │
                        └───────┬───────────────────┬──────────────────┘
                                │                   │
              ┌─────────────────▼────┐   ┌──────────▼───────────────┐
              │  DATA / INGESTION    │   │  AI / RAG  (server-side) │
              │  Source connectors → │   │  Embeddings · Vector DB  │
              │  normalize (PDF/HTML │   │  Retrieval → grounded    │
              │  → text/structured)  │   │  generation → verification│
              │  dedup · versioning  │   │  eval harness · cost log  │
              └─────────┬────────────┘   └──────────┬───────────────┘
                        └────────────┬──────────────┘
                              ┌──────▼──────┐
                              │  Postgres   │  (relational + optional
                              │  (+vector)  │   pgvector [CONFIRM])
                              └─────────────┘
```

Guaranteed boundaries (non-optional):

- **No LLM keys in browser.** All model calls are proxied server-side.
- **Auth is a middleware wrapper**, not per-route code.
- **All writes follow the tenant-scoping convention** (Guardrail 1).

---

## 1. Data Engineering & Ingestion Pipelines

### 1.1 Source catalog

**Q1 RESOLVED (2026-09-01):** candidate source set confirmed by owner — see **[08-Sources.md](08-Sources.md)** (full registry with access details, licensing notes, tiers, and rationale per source).

MVP-core seed set (M1): Library of Congress A–Z · PMC/PubMed BioC · OpenAlex · Oxford ORA · Crossref (Sage) · owner's compiled "Excel with Articles" · Congress.gov · GovInfo · PolicyNote (incl. CQ) · Senate Lobbying Disclosure (lda.gov) · CourtListener · Caselaw Access Project · Regulations.gov · Federal Register.

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
   --(structured fields)→ documents + events rows -> Postgres
   --(dedup/version)--> version table integrity
```

Workflow:
1. **Fetch** (`[TBD]` framework: e.g. Python + httpx / Airflow-lite triggers).
2. **Normalize:** PDF/HTML → text with layout + section preservation; extract metadata (jurisdiction, doc type, date, entities, gov action types).
3. **Dedup & version:** content-hash; store previous versions; keep source canonical, expose `latest` view.
4. **Chunk:** sentence-aware chunking with `[TBD]`-token overlap; keep per-chunk source span anchors (needed for 3.2 citation grounding).
5. **Embed + index:** `[CONFIRM]` embedding model + vector DB; store chunk ↔ source span map.

### 1.3 Ingestion scheduling vs triggers

- Initial seed: one-shot bulk load.
- Ongoing: `[TBD]` — scheduled crawl (e.g. daily/nightly) or event-driven webhook where source provides it.
- MVP constraint: seed-first, refresh-later. Don't build a scheduler until M2 proves value.

### 1.4 Data warehouse/lake

None for MVP. Skip lake/lakehouse, go straight to Postgres + vector. Note if corpus > `[TBD]`GB, revisit.

---

## 2. AI Engineering & RAG Architecture

### 2.1 Retrieval layer

- **Embedding model:** `[CONFIRM] (e.g. bge-m3 / OpenAI text-embedding-3 / Cohere embed-v4)`.
- **Vector store:** `[CONFIRM]` — default recommendation: **Postgres + pgvector** (one tech to run; enough for MVP corpus). Managed vector DB (Pinecone) only if corpus/scale pull ahead.
- **Retrieval design:** hybrid (dense vector + BM25 fallback) `[TBD]`; top-k retrieval with rerank option `[TBD]` if quality demands.

### 2.2 Generation — grounded RAG

- **LLM:** `[CONFIRM]` (candidate Claude for summaries/Haiku-class for speed; OpenAI/Cohere embeddings `[TBD]`).
- Pattern: **retrieve → inject retrieved spans as context → generate with citation anchors** that the UI renders as `[n]` → link to span/URL. No free-form generation outside retrieved spans.
- Sentence-level citation: model emits every claim tagged to one or more spans; post-processor maps to exact source passage + URL.

### 2.3 Verification step (mandatory, non-optional)

- After generation, a **check pass** re-verifies every emitted citation resolves to an actual fetched span (exact/overlapping text match where feasible `[TBD]` tolerance).
- Claims failing verification: dropped from output (not shown to user). This enforces "never hallucinate."
- Classifier/matcher: simple overlap + `[TBD]` threshold; escalate to eval set when uncertain.

### 2.4 Eval harness

- Golden set: `[TBD]` hand-curated Q&A + highlight extractions with expected citations.
- Metrics: **hallucination rate (goal 0)**, citation recall/precision, summary factuality (LLM-as-judge + human spot-check `[TBD]`).
- Ran on every material prompt/retrieval change; gates Phase 3 done (see 06).

### 2.5 Cost control (free-tester budget)

- Token accounting per request; daily + monthly rollups vs `[CONFIRM]` ceiling.
- Caching of identical/computed summaries; rate limits per tester session.
- Prefer cheap models (Haiku-class) for extraction; reserve flagship for synthesis only if necessary.
- Soft alert at 80% of ceiling; hard pause at 100% `[CONFIRM]`.

### 2.6 Stance & event extraction (capabilities 2 & 3 data layer)

- Same RAG pipeline reused: retrieve relevant statements/actions → stance classification `[TBD]` (for/against/neutral with confidence) → store in `stance_records` (see 3.2) with evidence citation.
- Event extraction: regulatory/court/policy events → normalized `events` rows with dates, actors, source docs.

---

## 3. Backend & Database Architecture

### 3.1 API framework

`[CONFIRM]` candidates:

| Option | Fit for MVP |
|---|---|
| **FastAPI (Python)** | Aligns with ingestion/LLM ecosystem; typed; fast to extend; single language across data layer |
| Next.js API routes (TypeScript) | One language with frontend; but co-locates heavy AI logic |

Default recommendation: **FastAPI for API + AI + ingestion; React/Next SPA for frontend** (`[CONFIRM]`).

### 3.2 Draft relational schema

Single Postgres DB `[CONFIRM]` (add `pgvector`). ORM: **SQLModel** (official FastAPI full-stack template stack — typed, keeps schema + Pydantic validation in sync). Tenant columns `user_id`, `organization_id` on all user-created rows (Guardrail 1). MVP single soft-tenant (a default org), but columns present.

```
users(id, email, display_name, role DEFAULT 'tester', created_at)
organizations(id, name, created_at)                       -- one org in MVP
memberships(id, organization_id, user_id, role, created_at)

documents(id, source_id, source_type, title, jurisdiction, doc_kind,
          published_at, version, content_full OR content_ref, metadata_json,
          created_by_user_id, organization_id, created_at)
chunks(id, document_id, seq, text, span_start, span_end, embedding vector, org_scope)
versions(id, document_id, content_hash, changed_at)

queries(id, user_id, organization_id, q_text, results_json, created_at)   -- audit/telemetry
summary_jobs / citations:
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

Index notes: `documents(jurisdiction, published_at)`, `claims(document_id)`, `stance_records(entity_id, issue_tag)`, `events(occurred_at)`, `queries(user_id, created_at)`. All queries scoped `organization_id = ?` / `user_id = ?` per tenant rule.

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

- One wrapper in the API framework (FastAPI `Depends`/middleware or NextAuth) `[CONFIRM]`.
- MVP: basic email/password (or magic link) sessions `[CONFIRM]`.
- Contract: routes declare `requires_auth`; switching to SSO/enterprise = one module change, no per-route edits. (Guardrail 3.)

---

## 4. Frontend Engineering & Data Visualization

### 4.1 Stack

- Framework: `[CONFIRM]` (default recommendation Next.js/React SPA).
- Styling: **Tailwind CSS** (recommended; fast iteration, consistent design).
- Components: **shadcn/ui** (pre-built, accessible, customizable) — recommended.
- Data fetching/state: **React Query (TanStack Query)** (server-state standard) — recommended.
- Charting: `[CONFIRM]` — candidates: **Recharts** (fast, React-native) vs **D3** (heavy) vs lightweight (visx). Default: Recharts-style declarative; D3 only if timeline needs bespoke.

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

- Hosting `[CONFIRM]`: default recommendation — a small VPS/free-tier PaaS (Fly.io/Render/Railway `[TBD]`) for API + DB; static/frontend hosting `[TBD]`.
- Observability: structured logs + token.log → metric rollups; uptime not critical in testing window.
- Cost budget: infra small; AI usage dominates → see 2.5 and `[CONFIRM]` ceiling.

## 5.5 Decision framework — how these defaults were derived

Every recommendation in this doc is derived from engineering principles and the product's own constraints, weighed in priority order:

1. **Product guarantee** — citation integrity is the differentiator; any choice that weakens "never hallucinate" is rejected outright.
2. **Correctness of the citation contract** — the pipeline must be able to render claim → exact source span → URL with structural guarantees, not just prompt-level instruction.
3. **Operational simplicity** — the MVP is a small team on a free budget; fewer systems = fewer failure modes, less to secure, back up, and monitor.
4. **Maintainability** — prefer mature, actively maintained, widely deployed components: abundant documentation, fewer exotic bugs, and working knowledge that outlasts the MVP.
5. **Cost discipline** — free product with no revenue; AI spend dominates and is capped.

### Technology maturity evidence

The defaults are widely deployed and actively maintained components that interoperate cleanly: FastAPI for the API, Next.js/React for the UI, Postgres + pgvector for relational + vector storage, and retrieval-grounded generation (chunking → embeddings → hybrid search → re-ranking) as the mainstream production pattern for verifiable generative features. Choosing these minimizes deployment risk and troubleshooting time; nothing in the default set is fringe or experimental.

### Sharpening decisions

- **Frontend conventions:** Tailwind + shadcn/ui + React Query — broadly used conventions with strong documentation; fast to build and debug (§4.1).
- **ORM:** SQLModel (the official FastAPI full-stack template stack) — typed, keeps schema and validation in sync (§3.2).
- **Core engineering surface:** citation-verified synthesis + eval harness + cost-cap loop is the product's hardest engineering requirement — build it as a first-class subsystem (WBS Phase 3), not an afterthought.
- **Async seam:** long-running AI calls should eventually move to a background-job queue; documented as a seam, kept OUT of the MVP build, added if tester latency demands it.

---

## 6. Decision register (ADR-lite)

Every decision below states its **rationale** — so the "why" is captured, not just the "what". Status: `Done` (locked) or `Recommended` (my default; awaiting your confirmation).

### 6.1 Resolved

| Date | Decision | Rationale | Status |
|---|---|---|---|
| 2026-09-01 | Documentation-first approach (PRD/WBS/Arch/Sources) | Plan before code — cheap to change now, expensive later | Done |
| 2026-09-01 | **Q1:** source set = 13 MVP-core classes in 08-Sources.md | Coverage for all 3 capabilities with free/open APIs first; CourtListener's built-in citation checker directly supports the no-hallucination promise | Done (candidate set; licensing review pending in Phase 1) |
| 2026-09-01 | Citations: claim → source span → URL, verification pass mandatory | The #1 differentiator vs generic AI (no fabricated refs); enforcement is in the pipeline, not the prompt | Done |

### 6.2 Recommended defaults (awaiting [CONFIRM] — each with reasoning; framework in §5.5)

| # | Decision | Why this choice (rationale) | Alternative considered | Status |
|---|---|---|---|---|
| Q4 | Vector store: **Postgres + pgvector** | Run ONE database for relational + vectors; MVP corpus is small enough; zero extra service/ops; easy auth/tenant scoping in same queries; mature, well-documented extension. | Pinecone/Weaviate (managed vector): better at 10M+ vectors, but extra cost + data-egress complexity now | Recommended |
| Q2 | **Haiku-class LLM for extraction, flagship for synthesis** | Extraction (stances, events, metadata) is high-volume → cheap model; synthesis shown to users → flagship; balances the free-tester cost ceiling (Q3). | Single flagship everywhere: simplest but cost-prohibitive at scale | Recommended |
| Q3 | **Cost ceiling `[TBD]`** — soft alert 80%, hard pause 100% | Free product needs a guard so testers can't bankrupt the build; alerts precede surprise. | No limit: risky for a free prototype | Recommended (value TBD) |
| Q6 | API: **FastAPI (Python)** + React/Next SPA | Same language as ingestion/LLM ecosystem; typed; async; pairs with pgvector; separates heavy AI from UI. | Next.js API routes: one language for UI too, but couples AI logic to the web server | Recommended |
| Q5 | Hosting: small PaaS/VPS `[TBD]` (Fly.io/Render/Railway) | Cheap, fast deploys, enough for testing window; no infra hobby. | Self-managed VPS: more control, more upkeep | Recommended (value TBD) |
| Q7 | Charting: **Recharts-style declarative**; D3 only if timeline demands | Pro/Con dashboard + timeline are standard charts; declarative lib is fast to ship and React-native; D3 is heavy and slower to build. | D3: full control, but overkill for MVP views | Recommended |
| 2.1 | Retrieval: **hybrid dense + BM25** (rerank optional) | Dense embeddings catch semantics; BM25 catches exact citations/numbers; combined recall protects citation-groundedness. | Dense-only: simplest, misses exact-match edge cases | Recommended |
| 2.2 | Generation: **retrieve → grounded generation with span anchors**; never free-form beyond retrieved spans | Structural guarantee of truthfulness; UI renders `[n]`→span→URL. | Agentic open search: flexible but unpredictable citations | Done (pattern locked) |
| 3.2 | ORM: **SQLModel** (official FastAPI full-stack template) | Standard, typed; keeps schema + Pydantic validation in sync. | Raw SQL: more control, less safety/verbosity | Recommended (build-phase decision) |
| 4.1 | Frontend conventions: **Tailwind + shadcn/ui + React Query** | Mature, documented, widely used UI stack; declarative and fast to build/debug. | Custom CSS / bespoke state: slower, less discoverable | Recommended |

### 6.3 Open (deferred)

| Date | Open item | Why deferred |
|---|---|---|
| — | Q8 tester cohort (who/how many/channel) | Needs product decisions, not architecture |
| — | Q9 launch date / testing window | Needs schedule decision |
| — | Q10 time-saved target metric (e.g. ≥X%) | Needs baseline data from a small dry-run |

## 7. Confirmed decisions checklist (mirrors 05-Risks Q list)

- [x] Q1 source list ✅ (13 MVP-core classes; see 08-Sources.md — licensing review pending in Phase 1)
- [ ] Q2 LLM + embedding providers (recommended: Haiku-class + flagship split — pick concrete providers)
- [ ] Q3 AI cost ceiling (recommended: alerts at 80% / pause at 100%; set a number)
- [ ] Q4 vector store (recommended: Postgres + pgvector — next: pin pgvector version/setup)
- [ ] Q5 hosting provider (recommended: Fly.io/Render/Railway — pick one)
- [ ] Q6 API framework (recommended: FastAPI + React/Next SPA — locked unless you object)
- [ ] Q7 frontend + charting (recommended: Next.js + Tailwind + shadcn/ui + React Query + Recharts)
- [ ] Q8 tester cohort
- [ ] Q9 launch date/window
- [ ] Q10 time-saved target metric