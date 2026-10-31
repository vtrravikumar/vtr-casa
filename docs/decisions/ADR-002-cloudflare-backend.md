# ADR-002: Cloudflare Backend and Control Plane

- **Status:** Proposed — validate during Phase 1
- **Date:** 2026-10-31
- **Decision owners:** Project owner

## Context

The proposed stack is React + TypeScript with Cloudflare Workers, optional Durable Objects, D1 and R2. The browser should establish the real-time connection directly to the selected voice provider where secure short-lived credentials make that possible.

Cloudflare's platform has separate products and limits for request/response compute, Durable Object coordination, relational metadata and object storage. It should not be assumed that all are required for the first vertical slice.

## Proposed decision

Adopt Cloudflare Workers as the **proposed application control plane** for:
- account/session authorization;
- rate and duration limits;
- provider master-secret handling and short-lived credential issuance;
- session metadata, consent and usage APIs;
- webhook validation and post-session workflow initiation;
- authenticated access and deletion of stored artefacts.

Use:
- **D1** for structured session, consent, transcript, summary and usage metadata.
- **R2** for private, opt-in recordings and exports.
- **Durable Objects only if needed** for active-session coordination, connection lifecycle or a VTR-CASA-hosted WebSocket service.
- **Queues later** if post-call processing or deletion retries need asynchronous decoupling.

Do not proxy the complete audio stream through Workers unless a later experiment demonstrates a compelling need and confirms runtime, latency, concurrency and cost suitability.

## Consequences

**Positive**
- Clear separation between control plane and provider media plane.
- Server-side secret custody.
- Cloudflare services map to API, metadata, coordination and object-storage needs.
- Avoids introducing a custom real-time media service prematurely.

**Negative / trade-offs**
- More than one persistence product means deletion and retention must coordinate across D1, R2 and provider systems.
- Durable Objects add lifecycle and billing complexity if used without need.
- Direct provider media paths require careful client credential scope and normalized telemetry.
- Cloudflare plan limits and pricing must be checked before launch.

## Validation required

- Prove the Worker can issue provider credentials safely and enforce account/session quotas.
- Verify CORS, authentication and CSRF protection for the chosen auth model.
- Confirm D1 data model, row volume and query patterns.
- Verify private R2 upload/download authorization and deletion behavior.
- Determine whether a Durable Object is actually necessary for the first PoC.
- Test provider connection latency from target geographies and browsers.
- Estimate total monthly cost including Workers, D1, R2, provider usage and observability.

## Revisit when

- Provider protocol requires a server-side media proxy or relay.
- Session concurrency or state requirements exceed a simple Worker API.
- Data residency, retention or regulatory requirements favour another service.
- Workload profiling shows a meaningful cost or performance advantage elsewhere.

## References

- [Cloudflare Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/)
- [Cloudflare Workers limits](https://developers.cloudflare.com/workers/platform/limits/)
- [Durable Objects WebSocket guidance](https://developers.cloudflare.com/durable-objects/best-practices/websockets/)
- [D1 pricing](https://developers.cloudflare.com/d1/platform/pricing/)
- [D1 limits](https://developers.cloudflare.com/d1/platform/limits/)
- [R2 pricing](https://developers.cloudflare.com/r2/pricing/)
