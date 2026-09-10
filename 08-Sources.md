# APEX — Source Registry (Data Catalog)

Status: Draft — candidate set confirmed by owner; licensing review pending (Phase 1, WBS 1.2)
Owner: [Kaia]
Last updated: 2026-09-01
Ref: Architecture §1 (07), Risks Q1 (05)

> **Security note:** one API key (PolicyNote) was pasted during a planning session. It is **not** stored in any project file. Rotate that key, and store all keys only in `.env` (Guardrail 2, see 07 §3.4).

## How to read this registry

- **Feeds:** which core capability the source powers — F1 Research Repository & Synthesis, F2 Stance Tracker (Pro/Con), F3 Decision & Action Tracer.
- **Tier:** MVP-core (in the seed corpus) vs MVP-extend (add post-M0 if time) vs Future (do not build against yet).
- **Reasoning:** the "why" — every choice is justified so the platform is built deliberately, not by accident.

## A. Research & scholarly sources (pipeline F1)

| # | Source | Access / Endpoint | Key/docs | Tier | Feeds | Why we chose it |
|---|---|---|---|---|---|---|
| A1 | **Library of Congress A–Z database** | LoC APIs | [loc.gov/apis] | MVP-core | F1 | LoC catalogs are the canonical federal bibliographic index across congressional, judicial, and agency records; one surface ties together most other federal doc types. |
| A2 | **PubMed Central / PMC BioC** | `https://pmc.ncbi.nlm.nih.gov/tools/developers/` & BioC XML | Open, no key | MVP-core | F1 | Authoritative open-access bio/medical research with full text + machine-readable BioC format; great initial verifiable-synthesis demo set. |
| A3 | **OpenAlex API** | `https://api.openalex.org` (docs.openalex.org) | Free, no key | MVP-core | F1 | 250M+ scholarly works across ALL fields with links to OA full text — discovery layer that fills non-health coverage (econ, education, climate, governance) that PMC lacks. |
| A4 | **CORE API v3** | `https://api.core.ac.uk/v3` | Free academic key | MVP-extend | F1 | ~300M aggregated OA full-text records across thousands of repositories — the non-health equivalent of PMC for actual full text. |
| A5 | **arXiv API** | `https://export.arxiv.org/api/query` | Open, no key | MVP-extend | F1 | Preprints in CS/econ/physics relevant to tech & defense policy; OAI-PMH same as Oxford ORA so nearly free to add alongside it. |
| A6 | **Oxford University Research Archives (ORA)** | `https://ora.ox.ac.uk/oai2` (OAI-PMH) | Open | MVP-core | F1 | Harvestable full-text repository via OAI-PMH; proves the metadata-harvest pattern we reuse for other repos. |
| A7 | **Wiley Search API / SRU** | `https://onlinelibrary.wiley.com/action/sru` | SRU protocol | MVP-extend | F1 | Broad commercial-publisher search without full-text licensing; signals + citation links without redistributing paywalled text. |
| A8 | **Sage Journals — Crossref REST API** | Crossref via DOI | Open | MVP-core | F1 | Crossref is the DOI registration authority — reliable title/publisher/preprint metadata and citation links for Sage + all DOI-registered content. |
| A9 | **NASA Techport API** | `https://techport.nasa.gov/help/api` | Open | MVP-extend | F1 | Public-project metadata — a lightweight, safe first API integration and a niche (gov-funded tech) differentiator. |
| A10 | **"Excel with Articles" (compiled list)** | Manual dataset load | Owner-owned | MVP-core | F1/F2/F3 | Curated starting corpus from your own research — guarantees the first 10–50 articles exist offline, no API dependency at M1. |

## B. Congressional & legislative sources (pipeline F2)

| # | Source | Access / Endpoint | Key/docs | Tier | Feeds | Why we chose it |
|---|---|---|---|---|---|---|
| B1 | **Congress.gov API** | congress.gov API | Free key | MVP-core | F2, F3 | Official bill text, committee actions, member info, votes; the authoritative legislative record — core of the stance tracker. |
| B2 | **GovInfo API** | govinfo.gov | Free; API+bulk | MVP-core | F2, F3 | USGPO's official full-text (Federal Register, CFR, statutes, Congressional docs) — the "records" backbone and regulatory timeline source. |
| B3 | **PolicyNote API (FiscalNote)** | `https://data.policynote.com/` | **Key in .env only** | MVP-core | F2 | Commercial legislative tracking incl. **CQ** (Congressional Quarterly) — "publication of record" lens + committee/vote tracking that complements Congress.gov's raw data. |
| B4 | **Congress Press (dwillis/congress-press)** | GitHub repo (not an API) | open source | MVP-extend | F2 | Downloaded press-release corpus; members' press statements are strong stance signals for the Pro/Con tracker. |
| B5 | **Senate Lobbying Disclosure API (lda.gov)** | `https://lda.gov/api/v1` | Free key, ~120 req/min | MVP-core | F2 | Lobbying registrations + quarterly activity: who lobbies for whom on which issues — directly feeds stance/advocacy strategy. (Old lda.senate.gov retired 6/30/2026 — use lda.gov.) |
| B6 | **Open States API v3** | `https://v3.openstates.org` | Free key | **Future** | F2, F3 | State-level legislation, legislators, votes (all 50 states). Everything else here is federal — revisit when state coverage matters. Explicit deferral. |

## C. Court & legal sources (pipeline F3)

| # | Source | Access / Endpoint | Key/docs | Tier | Feeds | Why we chose it |
|---|---|---|---|---|---|---|
| C1 | **CourtListener (Free Law Project)** | REST API v4 `https://www.courtlistener.com/help/api/` | Free; EDU membership opens PACER | MVP-core | F3 | 9M+ decisions from 2,000+ federal/state courts; RECAP/PACER dockets; citation graph; **built-in citation-lookup (false-citation checker)** — directly enforces our no-hallucination rule for case law. |
| C2 | **Caselaw Access Project (Harvard LIL)** | `https://case.law/` | Free bulk | MVP-core | F3 | Official state/federal decisions through 2020 bulk-loaded as the historical precedent layer; CourtListener covers current/ongoing. |
| C3 | **Regulations.gov API v4** | `https://open.gsa.gov/api/regulationsgov/` | Free key, 1,000 req/hr | MVP-core | F2, F3 | All rulemaking dockets, proposed rules, public comments — regulatory action history (F3) + who-comments-on-what as stance signals (F2). |

## D. Government data & reference (cross-cutting)

| # | Source | Access / Endpoint | Key/docs | Tier | Feeds | Why we chose it |
|---|---|---|---|---|---|---|
| D1 | **data.gov API** | data.gov | Open | MVP-extend | F1, F2 | Aggregates thousands of federal datasets; useful for stats/context in summaries even when not a doc-repo source. |
| D2 | **Federal Register API** | APIs via GovInfo (see B2) | Free | MVP-core | F2, F3 | Daily rulemaking/notices = forward-looking regulatory events for the timeline. |
| D3 | **Policy Synth (npm)** | `npm install @policysynth/api` | open source (AI toolkit) | MVP-extend | F1, F2, F3 | Open-source research toolkit — deployable building blocks (retrieval/synthesis) that can accelerate Phases 2–3 without licensing cost. |

## Prioritization summary

- **M1 seed corpus (MVP-core):** A1–A3, A6, A8, A10, B1–B3, B5, C1–C3, D2 → ~11 source classes + your compiled Excel to prove all 3 capabilities.
- **After M1 (MVP-extend), only if time:** A4, A5, A7, A9, B4, D1, D3.
- **Explicitly deferred (not in MVP):** B6 Open States (state coverage), full OA ecosystem breadth, enterprise legal-vetting workflows.

## Open licensing questions (Phase 1, WBS 1.2)

- Confirm per-source redistribution terms (what can be stored vs link-only) for: PMC, OpenAlex aggregates, Wiley SRU, CORE full text, CourtListener (EDU terms), Regulations.gov comments.
- Default posture: **metadata + quote-level excerpts + links** everywhere; full-text storage only where license explicitly allows.
- Rule (from 07 §1.1): link + attribution rendered with every citation; no full-text redistribution where disallowed.