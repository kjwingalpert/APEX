# APEX — Applied Policy & Evidence Exchange

APEX is an AI-powered B2B/B2G SaaS web application that aggregates open-access research, court decisions, and government filings into a single interactive platform, enabling non-profits and policy teams to track legislative stances, analyze historical precedent, and drive evidence-based advocacy.

## MVP Goal

Build a functional web prototype that proves AI can automatically aggregate, synthesize, and contextualize open-access policy research and government data into verifiable, citation-backed insights for non-profits and advocacy teams.

**First build priorities:**
1. **Demonstrate Verifiable Synthesis** — search pre-loaded policy data (open-access journals, congressional records) and generate high-impact takeaways backed by exact, sentence-level citations that never hallucinate.
2. **Ship synthesis first** — Research Repository with semantic search and cited summaries. Stance Tracker and Action Tracer follow pilot validation.
3. **Validate the target audience workflow** — test with real non-profit researchers and small advocacy groups to confirm measurable time saved preparing for direct lobbying, expert testimony, and coalition building.

> Future direction: B2B/B2G scaling, monetization, multi-tenant feels. The MVP is deliberately a **free, usable product** to prove value with testers first.

## Document Index

| Doc | Purpose |
|---|---|
| [01-PRD.md](01-PRD.md) | Product requirements: problem, users, features (P0/P1), success metrics, non-functional requirements |
| [02-MVP-Scope.md](02-MVP-Scope.md) | What's in v1 + explicit deferral list |
| [03-WBS.md](03-WBS.md) | Work breakdown structure: phases → tasks → estimates → owners |
| [04-Milestones.md](04-Milestones.md) | Build milestones incl. "user-testing-ready" |
| [05-Risks.md](05-Risks.md) | Assumptions, open questions, top risks + mitigations |
| [06-Acceptance-Criteria.md](06-Acceptance-Criteria.md) | Per-phase "done" definitions + guardrails |
| [07-Architecture.md](07-Architecture.md) | Detailed technical spec (data, AI/RAG, backend/DB, frontend/viz) |
| [08-Sources.md](08-Sources.md) | Data source registry: access, licensing, tier, and the "why" for each |
| [Project APEX.md](Project%20APEX.md) | Original vision; current scope is defined in 02-MVP-Scope.md |
| [Decision log](docs/decisions/README.md) | Approved decisions, rationale, tradeoffs, and unresolved choices |

## Tech Stack (locked 2026-09-10, Milestone M0)

- **API:** Hono + TypeScript on Cloudflare Workers
- **Frontend:** Vite + React + Tailwind + shadcn/ui + React Query + Recharts
- **Data:** Cloudflare D1 (SQLite, relational) + Vectorize (embeddings) + R2 (raw docs)
- **AI:** Workers AI `bge-m3` embeddings; Anthropic Claude (Sonnet synthesis / Haiku extraction); Cloudflare AI Gateway for cost spend-limits, caching, fallbacks
- **Hosting:** Cloudflare (Workers, Pages, D1, Vectorize, R2, Workers AI, AI Gateway)
- Current technical specification: [07-Architecture.md](07-Architecture.md)

## Status

- [x] Planning docs drafted
- [x] Architecture decisions confirmed (M0 locked, 2026-09-10 — see 07-Architecture.md §6.2)
  - [ ] Remaining `[TBD]`: AI allocation/enforcement within $100/month, concrete model IDs/key rotation, build-phase picks (WBS 0.x + Phase 1)
- [x] Git repo initialized + GitHub live ([kjwingalpert/APEX](https://github.com/kjwingalpert/APEX))
- [ ] Repo scaffolded (WBS 0.3)
- [ ] MVP built and user-testing-ready