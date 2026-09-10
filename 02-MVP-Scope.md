# APEX — MVP Scope (v1)

Status: Draft
Owner: [Kaia]
Last updated: 2026-09-10

Current release baseline (2026-09-10): synthesis over a curated corpus first; Stance Tracker and Action Tracer are deferred. Shipping target November 3, 2026, subject to evidence quality; owner availability 10 hours/week; total operating allowance $100/month. See [decision log](docs/decisions/README.md).

## What MVP v1 IS

A **free, usable web prototype** that proves AI can aggregate, synthesize, and contextualize open-access policy research into verifiable, citation-backed insights — validated by real non-profit researchers and small advocacy groups.

**Capability roadmap (only Research Repository is a v1 release requirement):**

| Capability | v1 definition |
|---|---|
| Research Repository | Semantic search over a seeded corpus of open-access policy documents; instant AI summaries; sentence-level, verifiable citations on every claim |
| Stance Tracker — deferred | Visual Pro/Con breakdown of where key government entities/policymakers stand on an issue, built from analyzed statements/actions |
| Action Tracer — deferred | Basic timeline of historical precedent and regulatory evolution for an issue |

**Also in v1:**
- A small tester cohort with free access (light-touch sign-in) to validate the workflow.
- Baseline/measurement of time saved on lobbying, testimony, and coalition prep.
- Guardian requirements honored from day one: tenant-scoped data, secrets in env only, modular auth middleware.

## What MVP v1 is NOT (explicit deferral list)

These are intentionally deferred. They reopen only after tester validation shows product-market signal.

- **Auth/roles & enterprise admin:** no org management, RBAC, SSO, admin consoles. (Middleware design still makes these cheap later.)
- **B2B/B2G sales features:** no quotas, billing, invoicing, procurement, SOC2/security review packaging.
- **Monetization:** product stays free during testing; no pricing, trials, or paywalls.
- **Enterprise administration:** deferred; private user research and enforced user isolation are mandatory in MVP.
- **Scale engineering:** defer additional scaling infrastructure. Bounded, user-scoped response caching for cost control remains permitted.
- **Corpus breadth:** MVP seeds a curated subset of sources; full open-access ecosystem is post-MVP.
- **Enterprise-grade citation/legal vetting:** sentence-level citation rendering ships, but formal legal review workflows do not.

## Scope guardrails for the build

- Any task that can be deferred without blocking tester validation is deferred.
- If a feature grows beyond its v1 definition, it goes to the deferral list — not into the build.
- Approved $100/month total operating allowance is respected; alerts when approaching it.

## Definition of "scope done"

- Testers can use search and cited synthesis for real research; exact first task and output remain open.
- Citation-backed summaries render verifiable source links.
- Time-saved data collected from the cohort.
- Ref: [06-Acceptance-Criteria.md](06-Acceptance-Criteria.md)