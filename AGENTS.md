# APEX — Agent Context

## What this project is

APEX (**Applied Policy & Evidence Exchange**) is an AI-powered B2B/B2G SaaS that aggregates open-access research, court decisions, and government filings into one platform for non-profits and policy teams. It enables semantic search, AI summaries with verifiable sentence-level citations, a Pro/Con Stance Tracker, and a precedent Action Tracer.

**Core promise:** every AI-generated claim must render a verifiable sentence-level source citation. No hallucinated citations. Ever.

## Default agent workflow — Superpowers

Apply these defaults across this entire repository: application code, ingestion, configuration, architecture, planning, and documentation. Keep code and project context in this one repository. More specific directory instructions may extend these defaults without weakening APEX guardrails.

Use the installed `superpowers` skills through the current environment's skill discovery mechanism. Read the applicable skill before applying it; do not hard-code a developer's plugin-cache path or copy plugin internals into this repository. Start sessions with `superpowers:using-superpowers`. If required skills are unavailable, report the missing dependency and follow the documented workflow as far as possible without claiming the skill ran.

| Work | Default workflow |
|---|---|
| New features, behavior, or architecture | `superpowers:brainstorming` to clarify intent and review the design; `superpowers:writing-plans` for approved multi-step implementation. Reuse decisions already approved in the session. |
| Executing an approved plan | `superpowers:executing-plans`, or `superpowers:subagent-driven-development` when independent tasks and available agent tools justify delegation. |
| Feature or bugfix implementation | `superpowers:test-driven-development` for meaningful behavior tests, then implementation and relevant checks. |
| Bugs, failures, unexpected behavior | `superpowers:systematic-debugging` before proposing a fix. |
| Review | `superpowers:requesting-code-review` for substantial implementation; `superpowers:receiving-code-review` when responding to feedback. |
| Isolated feature work | `superpowers:using-git-worktrees` when isolation is needed; recognize an existing isolated checkout. |
| Completing code work | `superpowers:verification-before-completion`, then `superpowers:finishing-a-development-branch` when integration is needed. Report actual checks and remaining limitations. |
| Routine planning/documentation edits | Direct edits, decision-log updates, consistency/link checks, and `git diff --check`. No code tests or elaborate design cycle solely for prose changes. Substantive new architecture decisions still use brainstorming. |

Use parallel agents only for concrete independent tasks when useful local work can continue. Follow the current environment's tool and permission rules. Do not require delegation for routine edits.

Scale the process to the task. Explicit user instructions and existing authorization take precedence over skill defaults; do not repeatedly seek approval for an already approved design or routine reversible work. Repository instructions do not grant blanket permission to publish, deploy, spend money, or send messages.

Every build decision must be recorded in [the decision log](docs/decisions/README.md) with its rationale and status, and reflected in affected specifications. Keep proposals and unresolved questions visibly separate from accepted decisions.

## Tech stack (locked, M0 — 2026-09-10)

| Layer | Technology |
|---|---|
| **API** | Hono + TypeScript on Cloudflare Workers |
| **Frontend** | Vite + React + Tailwind + shadcn/ui + React Query + Recharts |
| **Database** | Cloudflare D1 (SQLite, relational, tenant-scoped) |
| **Vectors** | Cloudflare Vectorize (embeddings stored via Workers AI `bge-m3`) |
| **Storage** | Cloudflare R2 (raw PDFs, full text) |
| **LLM — Synthesis** | Anthropic Claude Sonnet-class (via AI Gateway) |
| **LLM — Extraction** | Anthropic Claude Haiku-class (via AI Gateway) |
| **Embeddings** | Workers AI `@cf/baai/bge-m3` |
| **Cost control** | Cloudflare AI Gateway spend limits (soft 80%, hard 100%) |
| **Auth** | Single Hono middleware wrapper (email/password or magic link MVP) |
| **Hosting** | Cloudflare (Workers, Pages, D1, Vectorize, R2, Workers AI, AI Gateway) |

Decision history and rationale: `docs/decisions/README.md`. Current technical specification: `07-Architecture.md`.

## Monorepo layout

```
APEX/
├── apps/
│   ├── web/          # Frontend (Vite + React)
│   └── api/          # API Worker (Hono/TS)
├── packages/         # Shared types, citation schema, utils
├── pipeline/         # Python ingestion/normalization scripts (optional, CI/local)
├── scripts/          # Build, seed, dev scripts
├── docs/             # Decision log (01–08 planning docs remain at repo root)
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
3. **Citation verification mandatory:** post-generation checks verify citation integrity and that the source supports the claim; unverified claims are withheld.
4. **Auth middleware wrapper:** single module; SSO/enterprise swap = one-file change.

## AI/RAG pipeline

```
Query → retrieve (Vectorize hybrid) → inject spans as context
      → generate (Claude via AI Gateway) with citation anchors
      → verify: citation resolves to a fetched span AND supports its claim
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
- AI-only allocation and enforcement within the approved $100/month total operating allowance (Q3)
- Auth method: email/password vs magic link
- ORM: Drizzle vs typed raw SQL (build-phase pick)

## Conventions to follow

- **Decision logging:** whenever a build decision is made, add a dated entry to `docs/decisions/README.md` with status, decision, rationale, alternatives/tradeoffs, and affected docs. Distinguish approved decisions from proposals and open questions; reconcile affected planning docs in the same change.
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
