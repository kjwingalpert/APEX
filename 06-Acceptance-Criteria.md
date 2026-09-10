# APEX — Acceptance Criteria (MVP v1)

Status: Draft
Owner: [Kaia]
Last updated: 2026-09-01

Per-phase "done" definitions, including the non-negotiable guardrails. A phase is not done until its checks below pass. Ref: [03-WBS.md](03-WBS.md), [07-Architecture.md](07-Architecture.md).

## Guardrails (apply to every phase)

1. **Tenant isolation:** every user-created database row has `user_id` and/or `organization_id`; no unscoped query ships; all API data paths filter by tenant by default. (MVP-lite tenant is fine, but the columns and scoping exist pattern-first.)
2. **Secrets hygiene:** no secrets in code, committed files, or frontend bundles; `.env`/.env equivalents git-ignored; `.env.example` documents shape; CI secret-scan passes; the browser never holds a vendor LLM key.
3. **Modular auth:** all authentication flows through one centralized wrapper (NextAuth/middleware); no ad-hoc auth per page/endpoint. Replacing login with SSO later = one module.
4. **Citation integrity:** any AI-generated claim shown to a user is either backed by a rendered sentence-level source citation and link, or explicitly withheld. No fabricated citations, ever.

## Phase definitions of done

### Phase 0 — Foundation
- [ ] Stack decisions recorded in 07-Architecture.md; no `[CONFIRM]` left blocking.
- [ ] Repo scaffolding with lint/format/CI; git history clean.
- [ ] Guardrails 1–3 demonstrated on the schema template and fetching a non-existent resource (404 envelope, no keyboard-smash of secrets).

### Phase 1 — Data & Access
- [ ] Source list + licensing terms reviewed and recorded.
- [ ] At least one end-to-end fetch from a live approved source succeeds.
- [ ] Auth wrapper stubbed; authenticating a request blocks unauthenticated data access.

### Phase 2 — Ingestion & Indexing
- [ ] Corpus of `≥ [CONFIRM]` documents ingested, deduped, versioned, embedded.
- [ ] Chunking produces sentence-aware units that survive to citation time.
- [ ] Tables carry tenant columns per Guardrail 1.

### Phase 3 — Citation-Verified Synthesis
- [ ] Golden eval set: hallucination rate = **0** on tested Q&A; citation-recall ≥ `[CONFIRM]`%.
- [ ] Every synthesized takeaway outputs claim + exact source span + URL.
- [ ] Cost logging live; no request bereft of token accounting; within ceiling.

### Phase 4 — Research Repository UI
- [ ] Search returns ranked, metadata-rich results.
- [ ] Summary UI renders citation chips/anchors linking to source passages; Guardrail 4 holds on screen.
- [ ] Endpoint guardrail check: every API path scoped by tenant (Guardrail 1).

### Phase 5 — Stance Tracker
- [ ] Pro/Con dashboard renders entities, issues, stance labels, and evidence drilldown with dates + citations.
- [ ] A blind reviewer can answer "where does X stand on Y?" solely from the dashboard.

### Phase 6 — Action Tracer
- [ ] Timeline renders precedent/regulatory evolution with event + document + date + source link.
- [ ] Filters (entity/jurisdiction/issue) work without re-architecting data.

### Phase 7 — Tester Workflow
- [ ] Cohort (≥ `[CONFIRM]` testers) completes real prep tasks.
- [ ] Time-saved data collected and interpretable; feedback synthesized.

### Phase 8 — Hardening & Release
- [ ] Full acceptance sweep passes (all above).
- [ ] Deployment live for cohort; analytics dashboard correct; secrets re-verified.
- [ ] No P0/P1 bug open.

## Release gate (M6 definition of done — main thread)

- [ ] All three capabilities reachable and usable by a first-time tester without assistance.
- [ ] Every AI claim has a visible, working citation link.
- [ ] Tenant scoping, secrets, and auth-middleware guardrail checks all pass in review.
- [ ] Free access path for cohort works end-to-end.