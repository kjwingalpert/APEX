# APEX — Milestones (MVP v1)

Status: Draft
Owner: [Kaia]
Last updated: 2026-09-01

Dates are placeholder gates until you set a start date. Each milestone maps to WBS phases in [03-WBS.md](03-WBS.md) and done-criteria in [06-Acceptance-Criteria.md](06-Acceptance-Criteria.md).

## Milestone M0 — Foundations locked
- WBS: Phase 0
- Stack + all `[CONFIRM]` architecture decisions resolved and recorded in 07-Architecture.md.
- Repo scaffolded, git + GitHub live, secrets pattern enforced.
- **Definition of done:** a new dev can clone and run with `.env.example`, all rows carry tenant scoping.

## Milestone M1 — Corpus online
- WBS: Phases 1–2
- Seeded corpus ingested, deduped, versioned, embedded, and searchable.
- **Definition of done:** semantic search returns ranked documents with source metadata over test topics.

## Milestone M2 — Verifiable Synthesis engine
- WBS: Phase 3
- Retrieval-grounded summaries with sentence-level citations pass the eval harness (0 hallucinated citations).
- **Definition of done:** golden-set hallucination rate = 0; citation-recall ≥ `[CONFIRM]`%.

## Milestone M3 — Research Repository usable
- WBS: Phase 4
- Web app supports semantic search + AI summaries with clickable citations.
- **Definition of done:** a first-time user can issue a search and read a citation-backed summary end-to-end.

## Milestone M4 — Stance Tracker live
- WBS: Phase 5
- Pro/Con dashboard renders entity stances with evidence drilldown for at least one test issue.
- **Definition of done:** non-technical reviewer can answer "where does X stand on Y?" with visible support.

## Milestone M5 — Action Tracer live
- WBS: Phase 6
- Precedent/regulatory timeline renders for a test issue with event + document links.
- **Definition of done:** timeline shows evolution of a policy/court/regulatory thread with sources.

## Milestone M6 — User-testing-ready (release gate)
- WBS: Phases 7–8, all upstream
- Free access for tester cohort; all three capabilities usable; time-saved analytics recording.
- **Definition of done:** full acceptance run passes (06), deploy live for cohort.

## Milestone M7 — Cohort validation complete
- WBS: Phase 7
- Tester sessions complete; time-saved data collected; feedback synthesized into a build roadmap.
- **Definition of done:** go/no-go review for scope-beyond-MVP (monetization, B2B/B2G features).

## Note
- No business/monetization milestones in this MVP (free product, per scope). Revisit after M7.