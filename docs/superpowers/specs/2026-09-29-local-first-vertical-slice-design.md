# APEX Local-First Vertical Slice Design

Status: Approved design

Date: 2026-09-29

## Purpose

Build the smallest complete version of APEX that proves its core evidence workflow before expanding the corpus or depending on remote Cloudflare services. A researcher must be able to search a small indexed collection, inspect relevant documents and passages, and read only claims that resolve to supporting source passages.

The first collection uses 5–10 accessible documents about new and repurposed Alzheimer's drugs. This topic is a representative seed for the Aging & Cognition Research Program and does not make APEX dementia-specific. All application boundaries remain reusable for other policy-research collections.

## Success criteria

The vertical slice is complete when it can:

1. Ingest 5–10 licensed or link-only source documents with provenance and immutable versions.
2. Normalize and split each document into stable, traceable passages.
3. Store documents, versions, passages, and user-owned research records in local D1.
4. Store permitted raw source files in local R2.
5. Return ranked document and passage results for a keyword query.
6. Return document results before synthesis finishes.
7. Produce extractive local claims that cite exact passage identifiers.
8. Resolve each citation to the indexed document version, passage text, and source URL.
9. Withhold claims with missing, malformed, or unsupported citations.
10. Show corpus coverage, access restrictions, conflicts, and evidence gaps.
11. Prevent one development user from reading another user's private research.
12. Run locally without a Cloudflare account or remote Cloudflare resource.

## Scope

### Included

- pnpm monorepo and local development commands.
- Vite and React research interface.
- Hono API running through Wrangler locally.
- Shared TypeScript contracts for evidence and API responses.
- Drizzle schema and SQL migrations targeting local D1.
- Local R2 source-file storage.
- A small owner-managed seed corpus.
- Document ingestion, normalization, versioning, and passage creation.
- Keyword retrieval behind a provider interface.
- Extractive local synthesis behind a provider interface.
- Structural citation verification and a bounded claim-support verifier.
- Automated tests for contracts, tenant isolation, ingestion, retrieval, and citation failure behavior.

### Excluded

- Remote deployment and Cloudflare account provisioning.
- Vectorize, Workers AI, and AI Gateway integration.
- Live source discovery during a user query.
- User document uploads.
- Stance Tracker and Action Tracer.
- Production authentication, billing, organization administration, and public sharing.
- Integration with the U.S. Cost of Dementia Model.
- A comprehensive dementia corpus.

## Repository structure

```text
apps/
  api/                    Hono Worker, routes, middleware, composition
  web/                    Vite React application
packages/
  contracts/              shared request, response, evidence, and error types
  db/                     Drizzle schema, migrations, repositories
  evidence/               passage, citation, and verification rules
  retrieval/              retrieval interface and local keyword provider
  synthesis/              synthesis interface and local extractive provider
pipeline/
  src/                    source fetch, normalization, versioning, chunking
  fixtures/               small licensed test corpus and metadata manifests
scripts/                  local setup and seed commands
docs/superpowers/          design and implementation plans
wrangler.jsonc            local bindings and later remote configuration
```

Each package owns one responsibility. Application code consumes package interfaces and does not import another package's internal files.

## Runtime architecture

```text
React web
   |
   v
Hono API
   |-- research repositories ------> local D1
   |-- source-object repository ---> local R2
   |-- RetrievalProvider ----------> LocalKeywordRetrievalProvider
   |-- SynthesisProvider ----------> LocalExtractiveSynthesisProvider
   `-- ClaimVerifier -------------> indexed passages and citations
```

Remote Cloudflare access changes provider composition rather than API contracts:

```text
LocalKeywordRetrievalProvider   -> VectorizeHybridRetrievalProvider
LocalExtractiveSynthesisProvider -> AnthropicSynthesisProvider via AI Gateway
local D1/R2 bindings             -> remote D1/R2 bindings
```

## Core data model

### Public corpus records

- `sources`: source organization, canonical URL, access method, reuse posture, and last checked date.
- `documents`: stable logical identity, title, authors, publication date, evidence category, access status, and current version.
- `document_versions`: immutable content hash, source retrieval date, raw-object key, normalized-text hash, and superseded version link.
- `passages`: immutable passage identifier, document-version identifier, ordinal, section label, character offsets, text, and source locator.

### User-owned records

- `users`: development identities used to prove ownership boundaries.
- `research_sessions`: user identifier, query, status, coverage note, and timestamps.
- `search_results`: user identifier, session identifier, document and passage identifiers, rank, score, and provider name.
- `syntheses`: session identifier, user identifier, state, limitation text, and timestamps.
- `claims`: user identifier, synthesis identifier, claim text, verification status, and withholding reason.
- `citations`: user identifier, claim identifier, passage identifier, quoted text, source URL, and verification status.
- `saved_documents`: user identifier and document identifier.

Every user-owned table carries `user_id` directly. Repository methods require a user identifier; no unscoped private-record method exists.

## Evidence contracts

### Retrieval result

A retrieval result contains:

- Document and immutable document-version identifiers.
- Passage identifier and exact passage text.
- Source URL and source locator.
- Rank and provider-specific score.
- Access status and evidence category.

### Synthesis claim

A synthesis claim contains:

- Plain-language claim text.
- One or more cited passage identifiers.
- Claim type: finding, conflict, limitation, or evidence gap.
- Verification status: pending, verified, or withheld.
- Withholding reason when not verified.

Unchecked claims never appear in the public API response used by the interface. The API may expose counts and progress states without exposing pending claim text.

## Ingestion flow

1. Read a source manifest containing canonical URL, access status, reuse posture, and local fixture path.
2. Compute a raw-content hash and create or reuse the stable document identity.
3. Store the raw file in local R2 only when the manifest permits storage.
4. Normalize permitted HTML, text, or PDF input into UTF-8 text while preserving section labels and source locators.
5. Compute a normalized-content hash and create an immutable document version.
6. Split normalized text at sentence boundaries into passages with a bounded target size.
7. Record passage ordinals, offsets, section labels, and locators.
8. Commit the version and passages in one D1 transaction. Mark the version current only after that transaction succeeds. A failed ingestion leaves no partially current database version; cleanup of any earlier R2 object is idempotent and may be retried.
9. Produce an ingestion report listing accepted, rejected, unchanged, and failed documents.

Documents without accessible full text may appear as metadata-only search records but do not produce passages and cannot contribute to synthesis.

## Retrieval flow

The first provider performs deterministic keyword retrieval over locally stored passages. It normalizes the query, matches terms, scores passages by term frequency and query-term coverage, and returns a stable ranking with document metadata.

The `RetrievalProvider` interface accepts a query, collection scope, filters, and result limit. It returns the shared retrieval-result contract. Vectorize integration must implement this interface and may add semantic ranking without changing consumers.

Retrieval success is independent from synthesis success. The API returns document results as soon as retrieval completes and starts synthesis as a separate state transition.

## Local synthesis flow

The local provider is extractive and deterministic. It selects complete sentences from the highest-ranked passages, preserves their meaning, and emits each sentence as a claim citing the originating passage. It may also emit an evidence-gap record when the retrieved corpus is too small or lacks query-term coverage.

This provider exists to test orchestration, UI states, citation resolution, and withholding. It is not presented as equivalent to model-generated synthesis quality.

## Verification flow

Verification has two gates:

1. **Structural integrity:** confirm that every cited passage exists, belongs to the stated immutable document version, contains the quoted text, and resolves to the stored source URL.
2. **Claim support:** for the extractive provider, require the claim to be an exact normalized sentence from a cited passage. Later model-generated claims require a separate semantic-support provider and domain-review evaluation.

A claim that fails either gate receives `withheld` status and a machine-readable reason. Only verified claims reach the user-facing synthesis response.

## API surface

- `GET /health`: local runtime and binding status without secrets.
- `POST /research`: create a user-scoped research session and execute retrieval.
- `GET /research/:id`: return session status, results, coverage, and verified synthesis when available.
- `GET /documents/:id`: return document metadata and accessible passages.
- `GET /documents/:id/passages/:passageId`: resolve a citation target.
- `POST /documents/:id/save`: save a document for the current development user.

All error responses use a shared envelope containing a stable code, safe message, and request identifier. Private-resource misses return the same not-found response whether the record does not exist or belongs to another user.

## Interface behavior

The first interface has four views or states:

1. Search input with collection scope and evidence-category filter.
2. Ranked document results with source, date, access, and coverage labels.
3. Synthesis status that remains separate from the usable results list.
4. Verified synthesis claims with expandable supporting passages, conflicts, limitations, and gaps.

If synthesis fails, search results remain available. If the corpus is insufficient, the interface explains the bounded coverage rather than claiming that no research exists.

## Authentication during the vertical slice

Production authentication is deferred. A development-only authentication middleware reads a test-user header accepted only in the local environment and resolves it to a seeded user. Tests use two seeded users to prove isolation. The middleware conforms to the same `AuthContext` consumed by routes so a production authentication provider can replace it without changing route logic.

The application must refuse to start with development authentication enabled outside the local environment.

## Testing strategy

- Contract tests ensure the API and interface share identical types.
- Migration tests create a fresh local D1 database and apply every migration.
- Repository tests prove that one user cannot read another user's sessions, syntheses, or saved documents.
- Ingestion tests cover unchanged content, new versions, metadata-only records, malformed documents, and atomic failure.
- Retrieval tests use fixed fixtures and assert deterministic ordering.
- Citation tests cover missing passages, wrong versions, altered quoted text, and broken source mappings.
- Synthesis tests prove pending and withheld claim text is absent from user-facing responses.
- API tests prove results survive synthesis failure and private-record misses do not reveal existence.
- A browser-level smoke test covers search, results, verified citation expansion, and an explicit evidence-gap case.

## Local and remote configuration

The repository pins Wrangler as a development dependency and uses `wrangler dev --local` for account-independent work. Local D1 and R2 state lives under `.wrangler/state` and is ignored by Git. Configuration contains placeholder resource names and no production identifiers or secrets.

After USC IT approval:

1. Create development D1, R2, and Vectorize resources.
2. Replace placeholder identifiers through environment-specific Wrangler configuration.
3. Add remote retrieval and synthesis providers without removing local providers.
4. Run provider contract tests against development resources.
5. Verify AI Gateway logging, limits, and secret handling.
6. Deploy only after local and remote contract tests agree.

## Delivery order

1. Workspace, commands, and continuous integration.
2. Shared contracts and development authentication boundary.
3. D1 schema, migrations, and tenant-scoped repositories.
4. Fixture manifest, local R2 storage, and ingestion pipeline.
5. Retrieval interface and deterministic local provider.
6. Citation resolver and verification gates.
7. Local synthesis provider and asynchronous research-session states.
8. API endpoints and React interface.
9. End-to-end acceptance checks.
10. Remote Cloudflare provider integration after access approval.

Each delivery step must leave a testable result. Corpus expansion begins only after the vertical slice passes the evidence and isolation checks.
