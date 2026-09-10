# APEX — Agent Context

## What this project is

APEX (**Applied Policy & Evidence Exchange**) is an AI-powered B2B/B2G SaaS that aggregates open-access research, court decisions, and government filings into one platform for non-profits and policy teams. It enables semantic search, AI summaries with verifiable sentence-level citations, a Pro/Con Stance Tracker, and a precedent Action Tracer.

**Core promise:** every AI-generated claim must render a verifiable sentence-level source citation. No hallucinated citations. Ever.

## Tech stack (locked, M0 — 2026-09-10)

| Layer | Technology |
|---|---|
| **API** | Hono + TypeScript on Cloudflare Workers |
| **Frontend** | Next.js/React SPA (Vite-compatible) + Tailwind + shadcn/ui + React Query + Recharts |
| **Database** | Cloudflare D1 (SQLite, relational, tenant-scoped) |
| **Vectors** | Cloudflare Vectorize (embeddings stored via Workers AI `bge-m3`) |
| **Storage** | Cloudflare R2 (raw PDFs, full text) |
| **LLM — Synthesis** | Anthropic Claude Sonnet-class (via AI Gateway) |
| **LLM — Extraction** | Anthropic Claude Haiku-class (via AI Gateway) |
| **Embeddings** | Workers AI `@cf/baai/bge-m3` |
| **Cost control** | Cloudflare AI Gateway spend limits (soft 80%, hard 100%) |
| **Auth** | Single Hono middleware wrapper (email/password or magic link MVP) |
| **Hosting** | Cloudflare (Workers, Pages, D1, Vectorize, R2, Workers AI, AI Gateway) |

Full rationale: `07-Architecture.md` §6 decision register.

## Monorepo layout

```
APEX/
├── apps/
│   ├── web/          # Frontend (Next.js/React)
│   └── api/          # API Worker (Hono/TS)
├── packages/         # Shared types, citation schema, utils
├── pipeline/         # Python ingestion/normalization scripts (optional, CI/local)
├── scripts/          # Build, seed, dev scripts
├── docs/             # Planning docs (the 01–08 files below)
└── wrangler.jsonc    # Cloudflare Worker config
```

## Key planning docs

| Doc | Purpose |
|---|---|
| `01-PRD.md` | Product requirements: problem, users, features (P0/P1), success metrics |
| `02-MVP-Scope.md` | What's in v1 + deferral list |
| `03-WBS.md` | Work breakdown structure: phases → tasks → estimates |
| `04-Milestones.md` | Build milestones (M0–M7) |
| `05-Risks.md` | Risks, assumptions, open questions |
| `06-Acceptance-Criteria.md` | Per-phase done definitions + guardrails |
| `07-Architecture.md` | Technical spec + decision register |
| `08-Sources.md` | Data source registry + licensing |

## Architecture guardrails (non-optional)

1. **Tenant scoping:** every user-created row carries `user_id` / `organization_id`; all queries scoped by tenant.
2. **No LLM keys in browser:** all model calls proxied server-side via Workers → AI Gateway.
3. **Citation verification mandatory:** post-generation pass checks every cited span resolves to a real source; unverified claims are withheld.
4. **Auth middleware wrapper:** single module; SSO/enterprise swap = one-file change.

## AI/RAG pipeline

```
Query → retrieve (Vectorize hybrid) → inject spans as context
      → generate (Claude via AI Gateway) with citation anchors
      → verify: every emitted citation resolves to a fetched span
      → unverified claims DROPPED
      → render to UI: [n] → span text → source URL
```

## Current status

- [x] Planning docs drafted
- [x] M0 locked (architecture decisions confirmed, 2026-09-10)
- [x] Git repo initialized + synced to GitHub
- [ ] WBS 0.3: repo scaffolding (pnpm workspace, wrangler config, lint/format, CI)
- [ ] WBS 0.4: secrets setup + R10 key rotation
- [ ] WBS 0.5: tenant-scoping convention + schema template

**Next immediate step:** WBS 0.3 — scaffolding the monorepo layout.

## Open decisions (still TBD)

- Concrete model IDs + which Anthropic key (Q2 partial)
- Monthly AI cost ceiling number for AI Gateway spend limits (Q3)
- Auth method: email/password vs magic link
- ORM: Drizzle vs typed raw SQL (build-phase pick)

## Conventions to follow

- **pnpm workspaces** (match `cloudflare-os` conventions)
- **TypeScript** everywhere in `apps/` and `packages/`; Python only in `pipeline/`
- **No secrets in code** — `.env.example` only; production keys via `wrangler secret put`
- **No comments unless asked**
- **Each claim must have a verifiable citation path** — this is the core product differentiator

## Common commands

```bash
pnpm install              # install deps
pnpm run dev              # local dev (wrangler)
pnpm run build            # build all packages
pnpm run lint             # lint + typecheck
wrangler dev              # local Worker dev server
wrangler deploy           # deploy to Cloudflare
wrangler secret put KEY   # set a secret
```
