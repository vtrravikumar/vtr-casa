# ADR-001: Voice Agent Architecture

- **Status:** Proposed — experiment required before final selection
- **Date:** 2026-10-31
- **Decision owners:** Project owner

## Context

VTR-CASA needs a browser-based real-time voice agent before cellular/telephony integration. The product values natural conversation, low latency, turn-taking, barge-in, transcripts, consented recording, usage/cost visibility and future provider replacement.

Candidate providers expose different integration shapes:
- OpenAI Realtime supports browser WebRTC with short-lived client secrets and a server-mediated SDP option.
- Deepgram Voice Agent provides a unified WebSocket loop with STT/LLM/TTS orchestration and latency reports.
- ElevenLabs Conversational AI supports real-time WebSocket sessions and signed-URL authentication.

A single vendor should not dictate the whole application domain model.

## Proposed decision

1. Use a **provider-neutral voice adapter** at the application boundary.
2. Keep account authentication, authorization, policy, session metadata and secret issuance on the application backend.
3. Prefer a **browser-direct real-time connection** using short-lived provider credentials when the provider supports a secure flow. Do not relay all audio through Cloudflare by default.
4. Evaluate OpenAI Realtime and Deepgram Voice Agent as the first two implementation candidates. Keep ElevenLabs in the comparison set.
5. Do not finalize the provider until a repeatable test compares conversation quality, latency, interruption, transcript quality, failure handling and actual usage cost.
6. Treat recording as an independent consent-gated capability, not an automatic side effect of a live voice connection.

## Consequences

**Positive**
- The backend remains a control plane instead of a media bottleneck.
- Provider changes should be limited to adapters and provider-specific configuration.
- Short-lived client credentials avoid exposing master keys.
- Provider comparison can use a consistent test harness.

**Negative / trade-offs**
- Browser-direct sessions limit how much raw audio the application backend can inspect in real time.
- Provider adapters must normalize different lifecycle, transcript and usage event schemas.
- A browser-supplied event stream must not be trusted for authorization or billing.
- Credential TTL, quota and revocation behavior vary by provider.

## Experiment required

Build the same small conversation scenario with at least two candidates. Measure:
- time to first audible response;
- p50/p95 turn latency;
- interruption stop latency and false interruption rate;
- transcript accuracy for names, numbers and dates;
- disconnect/reconnect behavior;
- task completion and blinded naturalness rating;
- provider cost per connected minute and completed scenario;
- recording consent and deletion behavior.

Record browser, device, network, locale, model, prompt version and pricing date.

## Revisit when

- A provider cannot satisfy the required browser connection or security model.
- Target-language testing shows materially different outcomes.
- Provider costs or data terms change.
- A telephony integration introduces different transport or compliance constraints.

## References

- [OpenAI Realtime browser guide](https://developers.openai.com/api/docs/guides/voice-webrtc)
- [Deepgram Voice Agent](https://deepgram.com/product/voice-agents)
- [Deepgram Voice Agent events](https://developers.deepgram.com/docs/voice-agent-outputs/)
- [ElevenLabs WebSocket](https://elevenlabs.io/docs/eleven-agents/libraries/web-sockets)
- [ElevenLabs authentication](https://elevenlabs.io/docs/eleven-agents/customization/authentication)
