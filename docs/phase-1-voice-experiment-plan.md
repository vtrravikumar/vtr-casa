# Phase 1 — Browser Voice PoC Experiment Plan

**Status:** Proposed  
**Scope:** Technical validation and provider comparison; no production implementation  
**Decision owner:** Project owner  
**Last updated:** 2026-10-31

## 1. Purpose

Validate that VTR-CASA can support a responsive, natural browser-based voice conversation before committing to a provider or building the application. The experiment should answer:

1. Can a browser capture microphone audio and play assistant speech reliably?
2. Does conversation feel natural, including turn completion and interruption?
3. What is the measured end-to-end latency?
4. What are the practical cost and integration constraints?
5. Can recording, transcript, and summary features remain independently consented and off by default?

This is an experiment, not a production launch. Do not build telephony, iOS cellular-call integration, personalized voice, account/billing, or a full screening workflow in this phase.

## 2. Candidate providers and experiment design

Use a small, controlled comparison of **OpenAI Realtime** and **Deepgram Voice Agent** as the first candidates. Keep **ElevenLabs Conversational AI** as a reserve candidate if the first two do not meet voice-quality or integration needs.

The exact model, region, voice, SDK/API version, and pricing must be recorded at test time. Confirm current provider documentation, browser transport support, short-lived credential flow, data handling, and pricing before provisioning credentials. Provider choice remains open until the evidence review.

Prefer a browser-to-provider media connection using short-lived, scoped credentials where supported. The backend should issue/authorize sessions and keep long-lived secrets server-side. Do not relay live audio through the Worker unless measurements or provider constraints establish a clear need.

## 3. Minimal prototype scope

The prototype should be limited to one browser page and a small session-control backend:

- Start/end a voice session with explicit microphone permission.
- Show session state: idle, connecting, listening, assistant speaking, and error.
- Use a fixed assistant persona and a small set of scripted test prompts.
- Support natural turn-taking and barge-in (user speech interrupts assistant output).
- Display a live transcript only when the tester has enabled transcript display.
- Recording is off by default and requires a separate, explicit opt-in.
- Show basic session metrics: connection time, response latency, interruptions, and approximate provider usage/cost where available.
- Provide a clear end-session control and visibly indicate microphone/recording state.

Avoid account creation and persistent user profiles. Use synthetic test content; do not test with sensitive personal, financial, medical, or third-party call data.

## 4. Test protocol

Run the same scenario set with each provider under comparable conditions. Use the same browser, device, network, prompt, and approximate session duration. Record configuration and any deviations.

### Scenario set

1. **Simple response:** “Hello. What can you help me with?” — baseline connection and first response.
2. **Natural pause:** Ask a multi-part question and pause briefly between clauses — evaluate premature turn endings.
3. **Long user turn:** Speak for approximately 20–30 seconds — evaluate capture and end-of-turn behavior.
4. **Barge-in:** Interrupt assistant speech with a new question — measure how quickly output stops and the new turn begins.
5. **Noisy environment:** Repeat a short prompt with moderate ambient noise — observe transcription and response robustness.
6. **Recovery:** Deny microphone permission, then retry; also test a transient network interruption — confirm understandable error/recovery behavior.
7. **Consent path:** Start with recording disabled; confirm no recording artifact is created. Enable recording explicitly and verify the visible state and resulting artifact. Turn it off and end the session.
8. **Transcript/summary path:** Test transcript and summary as distinct controls; verify each remains off unless explicitly enabled and that summaries do not invent unsupported details.

For each scenario, perform at least five repetitions per provider for an initial directional result. This is not a statistically powered benchmark.

## 5. Metrics and definitions

| Metric | Definition |
|---|---|
| Session connection time | Start action to confirmed ready/listening state |
| First-response latency | End of user turn (or provider's turn-end event) to first audible assistant output |
| Barge-in stop latency | User interruption onset to assistant audio stopping |
| Turn-end errors | Premature cutoffs or assistant responses before the user finishes |
| Task completion | Whether the assistant answers the scripted request accurately and within scope |
| Speech quality | Tester rating for intelligibility, naturalness, and voice fatigue |
| Reliability | Successful sessions / attempted sessions, with failure reason |
| Approximate cost | Provider-reported usage and current public rates; report assumptions and exclusions |

Report median and p90 for latency where sample size allows. Keep raw observations and provider configuration alongside the summary. Do not compare unlike pricing units without converting and documenting assumptions.

## 6. Provisional decision criteria

These are experiment targets, not promises or production SLAs. Revisit them after the first baseline run.

- Median first-response latency at or below **1.0 second**; p90 at or below **2.0 seconds**, where provider events make measurement possible.
- Median barge-in stop latency at or below **500 ms**.
- At least **90%** successful session starts across repetitions, with no unexplained failures.
- No critical consent-state failure: recording must not start unless the tester explicitly opts in.
- No long-lived provider secret is exposed in browser code or browser-visible configuration.
- Assistant responses remain within the scripted scope and do not claim to have screened or blocked a real telephone call.
- Cost can be estimated transparently from observed usage; no fixed cost commitment is made at this stage.

A provider that misses a target may still be selected if the cause is understood and the trade-off is acceptable. The decision should consider latency, barge-in, voice quality, browser support, privacy controls, developer complexity, and cost together.

## 7. Privacy and security checklist

- Request microphone permission only after the tester initiates a session.
- Recording defaults to off and has a separate opt-in from transcript display.
- Clearly indicate recording status; stop recording immediately when disabled or when the session ends.
- Use synthetic test content and restrict access to experiment artifacts.
- Keep provider API keys and other long-lived secrets on the server; issue only short-lived, scoped session credentials to the browser.
- Do not store audio, transcript, or summary by default. If persistence is required for evaluation, document the purpose, access, retention, and deletion procedure first.
- Verify provider retention/training terms and regional processing before real-user testing.
- Do not treat this prototype as an emergency service or as a system that can access ordinary cellular-call audio.

## 8. Deliverables

1. A minimal browser prototype capable of the scoped scenarios.
2. A completed test matrix with provider/model/configuration details and observed metrics.
3. A brief evaluation note summarizing trade-offs, unresolved issues, and recommendation.
4. An architecture decision record updating the selected provider and transport only after reviewing results.

## 9. Exit gate

Phase 1 is complete when the prototype scenarios have been exercised, the test matrix is documented, consent behavior is verified, and a provider/transport recommendation is made with evidence. The project owner reviews the recommendation before Phase 2 implementation scope is finalized.

## 10. Open questions for the owner

- Which browser and primary test device should be the baseline?
- Should the first evaluation prioritize English only, or include Tamil from the first pass?
- Is the experiment strictly local/manual for the first iteration, or should a private preview deployment be included?
- What maximum approximate cost per 10-minute test session is acceptable?

These questions do not block preparing the experiment plan, but should be answered before provider credentials or a running prototype are provisioned.
