# Source and infrastructure review — September 11, 2026

Status: recommendations, except the two evidence categories approved in D024.

## Workbook review

Read both sheets of `/Users/kaia/Downloads/APEX_API_Integration_Specs.xlsx` without modifying the workbook. Main sheet: 15 entries in rows 2–16. Workbook endpoint, quota, and access statements remain historical planning notes, not freshly verified integration contracts.

- Research candidates: Crossref (row 8), PMC (9), Wiley (10), Oxford ORA (11). Crossref is metadata discovery; accessible licensed full text must be resolved separately. Repository contents require document-level eligibility screening.
- Later tracer sources: Congress.gov (2), Federal Register (3), GovInfo (4), CourtListener (5), CAP (6), PolicyNote (7), congress-press (15). Official actions/statements must not be mislabeled as peer-reviewed research.
- Specialized context/discovery: NASA Techport (12), data.gov (13), Library of Congress (14).
- Policy Synth (16) is application tooling, not a source.
- OpenAlex, Unpaywall, CORE, and Pew are absent from this workbook; consider evaluating them separately. No Pew API has been verified here.
- The build-order sheet B10:B17 prioritizes all three features and PolicyNote setup. That order predates synthesis-first scope and should not drive the MVP. API-specific quotas, institutional memberships, subscription permissions, and methods require current verification before integration.

## Infrastructure recommendations

| Tool | MVP recommendation | Revisit trigger |
|---|---|---|
| Kafka | Defer; consider Cloudflare Queues for background ingestion if needed. Queues is not a Kafka feature-equivalent replacement. | Multiple independent consumers requiring durable event replay/stream processing |
| Docker | Optional for a reproducible parser/OCR environment; unnecessary for normal Workers deployment. | Native PDF/OCR dependencies that do not fit Workers |
| Kubernetes | Defer; no cluster to manage in the current Workers design. | A separately justified fleet of custom container services |
| GraphQL | Defer; use Hono HTTP/JSON endpoints. | Multiple clients require substantially different nested data views |
| gRPC | Defer; no independent service fleet requiring protobuf RPC. | Measured service-to-service needs or an integration requiring it |
| WebSockets | Defer; job polling or one-way progress events suffice initially. Never stream unverified claims. | Real-time shared research/collaboration |
| Redis | Defer; D1 for durable records, scoped caches for reuse. KV/cache are not substitutes for atomic budget accounting. | Measured latency or Redis-specific data-structure needs |

Cloudflare Queues and Workflows may help durable ingestion or synthesis jobs; select based on retry, duration, and ordering requirements, not by default. Use idempotent document processing and stable versions. Keep claims withheld until verification, even if progress is streamed.

References checked: [Queues](https://developers.cloudflare.com/queues/), [Containers](https://developers.cloudflare.com/containers/), [Durable Object WebSockets](https://developers.cloudflare.com/durable-objects/best-practices/websockets/), [storage choices](https://developers.cloudflare.com/workers/platform/storage-options/), [Cache behavior](https://developers.cloudflare.com/workers/reference/how-the-cache-works/). Recommendations are architectural judgments, not measured cost savings.
