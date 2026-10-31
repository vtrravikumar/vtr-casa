# AI Provider Evaluation

**Status:** Comparative research; no provider selected  
**Research date:** 2026-10-31  
**Rule:** Prices and capabilities below are snapshots from linked official sources, not a quote or a guarantee of account-specific availability. Recheck before committing.

## 1. Evaluation summary

| Dimension | OpenAI Realtime API | Deepgram Voice Agent | ElevenLabs Conversational AI |
|---|---|---|---|
| Primary architecture | Speech-to-speech realtime session | Unified STT + LLM orchestration + TTS agent | Managed conversational agent over WebSocket |
| Browser connection | WebRTC; short-lived client secret or server-mediated SDP | Browser Agent SDK / WebSocket with temporary token | WebSocket with signed URL; SDK options |
| Interruption / turn-taking | Realtime VAD and conversation events; verify barge-in behavior | Built-in turn detection and barge-in described by provider | Configurable turn-taking and interruption behavior |
| Provider flexibility | OpenAI model stack; app tools and backend handoff | BYO LLM and/or TTS options documented | Configurable LLM and voice options; verify exact feature tier |
| Transcript events | Realtime transcription/events; persist according to app policy | Conversation Text and turn events | Conversation transcripts/history features |
| Latency visibility | Capture client event timestamps and provider events; validate details | Per-turn Latency Report with STT/LLM/TTS breakdown | Measure client-side; inspect current API event coverage |
| Pricing model | Audio/text token pricing per model (other model/tool usage may add cost) | Per connected minute for Voice Agent tiers | Credit/subscription and product-specific agent usage; obtain exact current agent quote/calculator |
| Initial recommendation | Strong baseline candidate for low-latency speech-to-speech | Strong comparison candidate for operational transparency and integrated pipeline | Strong candidate where voice quality/control and agent configuration matter |

No table cell is a substitute for a live technical test. Use identical scripts, browsers, network conditions and prompt versions.

## 2. OpenAI Realtime API

### Confirmed from current official material
- OpenAI documents browser voice-agent construction with Realtime API and Agents SDK.
- Browser clients can connect over WebRTC.
- One pattern has the backend mint a short-lived client secret, after which the browser connects directly to the Realtime API. The main API key stays server-side.
- A server-mediated WebRTC flow is also documented, where the backend exchanges the SDP offer with the Realtime API.
- Realtime is positioned for speech-to-speech experiences with natural turn-taking, low first-audio latency, interruption and tool use.
- Pricing is model-dependent and token-based for Realtime models; backend model/tool use may be charged separately.

### Pricing snapshot
The official pricing page on 2026-10-31 lists:
- `gpt-realtime-2.1`: audio input $32 / 1M tokens; audio output $64 / 1M tokens.
- `gpt-realtime-2.1-mini`: audio input $10 / 1M tokens; audio output $20 / 1M tokens.
- Text input/output are priced separately, and cached input rates are lower.
- The price page also lists GPT-Live 1 sessions at $0.05/minute, but that is a distinct product/model/session pricing line and must not be assumed to be the price of Realtime API sessions.

These are not directly comparable to a per-minute rate without estimating actual audio token usage, text context, response length, caching and tools. The full conversation context can affect later turn costs.

### Strengths
- Direct browser WebRTC path avoids routing every audio packet through the application backend.
- Native speech-to-speech architecture is a good fit for interruption-sensitive conversation.
- Short-lived credentials align with a thin Cloudflare control plane.
- Tools and handoff patterns support eventual workflow integration.

### Risks / questions
- Need to test audio token consumption and cost across realistic conversation lengths.
- Need to validate transcript event quality and how final transcripts are recovered.
- Must pin GA API semantics and avoid legacy beta endpoints.
- Test target language/accent quality and provider availability from target geographies.
- Recording strategy may be independent from provider conversation transcript features.

## 3. Deepgram Voice Agent

### Confirmed from current official material
- Voice Agent combines speech-to-text, LLM orchestration and text-to-speech behind one WebSocket.
- Deepgram documents end-of-turn detection, barge-in and function calling.
- It supports bringing an external LLM and/or TTS provider, with documented provider choices and fallback capabilities.
- Server events include conversation text, speaker activity, agent thinking, function-call lifecycle and per-turn Latency Report.
- Browser Agent SDK documentation uses a token factory; the application backend should mint a short-lived token rather than expose an API key.
- Temporary API tokens are documented with a 30-second TTL for client-side use.

### Pricing snapshot
Deepgram's official pricing page on 2026-10-31 lists Voice Agent pay-as-you-go:
- Standard: $0.075/minute.
- Standard with BYO TTS: $0.065/minute.
- Custom with BYO LLM: $0.065/minute.
- Custom with BYO LLM and TTS: $0.050/minute.
- Advanced: $0.163/minute.
- Growth rates and capacity differ; the page lists an India concurrency figure for Growth, subject to account/region limits.

Pricing is based on WebSocket connection time. Confirm billing behavior, model choices and regional availability for the account before modelling cost. BYO components can introduce separate vendor charges.

### Strengths
- A unified WebSocket voice loop minimizes integration of separate STT, LLM and TTS streams.
- Latency reports offer useful diagnostic detail for the PoC.
- Bring-your-own LLM/TTS can reduce lock-in and make component replacement practical.
- Turn detection and barge-in are central product features, not bolt-ons.

### Risks / questions
- Verify the browser SDK maturity, reconnect behavior and required permissions.
- Confirm which transcript and latency events are persisted and retrievable after disconnect.
- BYO LLM/TTS can multiply vendors, secrets, failure modes and cost lines.
- Test perceived naturalness against speech-to-speech providers; integrated metrics alone do not establish quality.

## 4. ElevenLabs Conversational AI

### Confirmed from current official material
- Conversational AI supports real-time interactive audio over WebSocket.
- Private agents or authorized connections can use a signed URL minted by the backend; the API key must not be exposed client-side.
- Conversation flow settings cover silence, interruptions and turn-taking.
- Privacy settings include independent controls for audio saving, conversation history and retention.
- The current docs state a default two-year retention period for conversation data unless configured otherwise; set a shorter policy where appropriate and verify whether it applies to the exact plan and configuration.

### Pricing snapshot / limitation
The public pricing page uses credits shared across products and lists subscription bundles, but the surfaced official pricing material does not provide a directly comparable universal per-minute Conversational AI figure for this project. Do not infer an agent-minute price from text-to-speech character credits or creator plan prices. Before selection, obtain the exact agent usage calculator/rate for the intended plan, region, voice, LLM and concurrency.

### Strengths
- Strong voice and speech product focus.
- WebSocket client flow with signed URL pattern.
- Configurable conversation flow.
- Documented privacy and retention configuration.

### Risks / questions
- Need a comparable, written cost estimate per connected minute and for the expected concurrency.
- Confirm transcript event coverage, recording format, export and deletion behavior.
- Check exact model/voice availability, language quality and plan entitlements.
- Default retention is relatively long unless configured; test actual deletion behavior.

## 5. Cloudflare application platform evaluation

Cloudflare should initially act as the application backend and control plane, not necessarily the real-time audio transport.

| Service | Proposed responsibility | Fit / caveat |
|---|---|---|
| Workers | Authenticated API, provider credential minting, policy, webhooks, session metadata | Good fit for request/response control plane |
| Durable Objects | Optional session coordination, lifecycle and WebSocket server | Use only if coordination/state requires it; hibernation behavior is role-specific |
| D1 | Session, consent, transcript metadata, summary and usage records | Good relational store; observe row/query/database limits |
| R2 | Private recordings and exports | Good object store; access must be authorized; define retention/deletion |
| Queues (later) | Summaries, transcription, retention/deletion retry jobs | Add only when async workload justifies it |

### Pricing/limits snapshot
At the research date, Cloudflare's official pages list:
- Workers Paid plan minimum $5/month with included usage; standard requests and CPU have included allowances, then overage. Free plan has daily request and CPU constraints.
- D1 has included row reads/writes/storage and usage-based overage; plan limits differ.
- R2 Standard storage is $0.015/GB-month, Class A operations $4.50/million, Class B $0.36/million and internet egress is free (per the official R2 price page).
- Durable Objects have their own request/duration pricing and limits; check current Workers pricing and DO docs.

This excludes provider AI usage, domain, observability, identity provider, support and telephony charges. Do not treat the minimum Workers subscription as the total product cost.

## 6. Recommendation and selection process

Do not select a vendor solely from published marketing latency or prices. Run a small, time-boxed browser experiment with two candidates first:

1. OpenAI Realtime as the baseline speech-to-speech experience.
2. Deepgram Voice Agent as the integrated pipeline and latency-observability comparison.
3. Add ElevenLabs when its exact agent rate and connection/retention requirements are confirmed, or if voice quality testing justifies it.

Use the same five scripts and score:
- first-audio latency and p50/p95 turn latency;
- interruption yield latency and false interruptions;
- transcript accuracy on names, dates and numeric details;
- naturalness/helpfulness, rated by blinded human listening;
- disconnect/reconnect behavior;
- usage cost per connected minute and completed task;
- consented recording and deletion workflow;
- language and accent performance.

Provider selection should be captured in a new ADR only after the test. The app must remain capable of supporting a second provider even if one provider is selected for Phase 1.

## 7. Official sources

All links below were checked on 2026-10-31.

### OpenAI
- [Realtime API getting started](https://developers.openai.com/api/docs/guides/realtime)
- [Realtime WebRTC guide](https://developers.openai.com/api/docs/guides/voice-webrtc)
- [Create Realtime client secret](https://developers.openai.com/api/reference/resources/realtime/subresources/client_secrets/methods/create)
- [Pricing](https://developers.openai.com/api/docs/pricing)
- [Realtime latency and cost optimization](https://developers.openai.com/api/docs/guides/voice-latency-cost)

### Deepgram
- [Voice Agent product and capabilities](https://deepgram.com/product/voice-agents)
- [Voice Agent pricing](https://deepgram.com/pricing)
- [Voice Agent server events](https://developers.deepgram.com/docs/voice-agent-outputs/)
- [Browser Agent SDK overview](https://developers.deepgram.com/docs/browser-agent-overview)
- [Temporary token authentication](https://developers.deepgram.com/guides/fundamentals/token-based-authentication)

### ElevenLabs
- [Conversational AI WebSocket](https://elevenlabs.io/docs/eleven-agents/libraries/web-sockets)
- [Signed URL authentication](https://elevenlabs.io/docs/eleven-agents/customization/authentication)
- [Conversation flow](https://elevenlabs.io/docs/eleven-agents/customization/conversation-flow)
- [Privacy settings](https://elevenlabs.io/docs/eleven-agents/customization/privacy)
- [Retention settings](https://elevenlabs.io/docs/eleven-agents/customization/privacy/retention)
- [Pricing](https://elevenlabs.io/pricing)

### Cloudflare and Apple
- [Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/)
- [Workers limits](https://developers.cloudflare.com/workers/platform/limits/)
- [Durable Objects WebSockets](https://developers.cloudflare.com/durable-objects/best-practices/websockets/)
- [D1 pricing](https://developers.cloudflare.com/d1/platform/pricing/)
- [D1 limits](https://developers.cloudflare.com/d1/platform/limits/)
- [R2 pricing](https://developers.cloudflare.com/r2/pricing/)
- [Apple CallKit](https://developer.apple.com/documentation/CallKit)
- [Apple CXCallObserver](https://developer.apple.com/documentation/callkit/cxcallobserver)
- [Apple App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
