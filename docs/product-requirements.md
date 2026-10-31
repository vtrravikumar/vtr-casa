# Product Requirements

**Status:** Draft for Phase 0 review  
**Product:** VTR-CASA — Call Assistant & Screening Agent  
**Last reviewed:** 2026-10-31

## 1. Product vision

VTR-CASA is a personal AI call assistant that can answer selected calls, converse naturally with callers, understand why they are calling, handle routine requests within user-defined boundaries, and escalate important calls to the account holder.

The product should reduce unwanted interruptions without making callers feel trapped in a robotic phone tree. The assistant must be transparent that it is an AI assistant when speaking to people on the user's behalf.

## 2. Problem and target user

People receive calls that vary widely in importance: urgent personal matters, known contacts, service calls, sales calls, delivery coordination, and unknown numbers. Answering every call interrupts focus; ignoring calls risks missing something important.

The initial target is an individual who wants to test whether a voice agent can hold a useful, natural conversation. The browser PoC is a controlled experiment, not yet a telephone product.

## 3. Goals

1. Validate useful, natural voice conversation before investing in telephony or native mobile applications.
2. Measure perceived conversation quality, turn-taking, barge-in, latency and reliability.
3. Give the user clear control over microphone access, recording, retention and deletion.
4. Produce usable transcripts and concise summaries when those features are enabled.
5. Measure provider usage and estimate per-session AI cost.
6. Preserve the option to replace the voice provider.
7. Design the future call-screening workflow around user-defined boundaries and escalation.

## 4. Non-goals for the initial PoC

- Receiving, forwarding, routing or intercepting actual cellular calls.
- Accessing audio from the iOS Phone app.
- Public telephone numbers, PSTN/SIP integration or carrier partnerships.
- Background microphone capture or always-on listening.
- Automatic recording or covert caller profiling.
- Personalized voice cloning, voice conversion or imitating a third party.
- Autonomous commitments such as purchases, payments, sharing private information, or changing accounts.
- Complex multi-user tenancy, contact sync or enterprise administration.
- A production reliability, emergency-call or life-safety guarantee.

## 5. Users and actors

- **Account holder:** configures the assistant, starts sessions, chooses recording settings and reviews outputs.
- **Conversation participant:** speaks to the browser-based agent in the PoC; in future, may be an incoming telephone caller.
- **VTR-CASA application:** authenticates the account holder, issues short-lived provider credentials, stores opted-in session artefacts and exposes user controls.
- **Voice provider:** streams audio and returns audio, events and/or transcripts under its own service terms and retention controls.

The PoC must not imply that a browser participant is a telephone caller.

## 6. Functional requirements

### FR-01 — Session lifecycle
The user can explicitly start and end a voice session. The UI shows connecting, listening, thinking, speaking, interrupted, error and ended states.

### FR-02 — Microphone permission
Microphone access is requested only after a clear user action and is stopped when the session ends. Permission denial and device failures are handled without trapping the user.

### FR-03 — Real-time conversation
The agent receives live microphone audio and returns synthesized speech with interactive turn-taking. A text fallback or transcript display should make the interaction understandable when audio output fails.

### FR-04 — Interruption / barge-in
The user can speak while the agent is speaking. The agent should stop or yield promptly, listen to the new turn, and avoid treating ordinary background noise as a deliberate interruption. The exact behavior is provider-dependent and must be tested.

### FR-05 — Agent identity and disclosure
The agent uses a configured assistant identity and does not claim to be the account holder. For future calls, it should identify itself as an AI assistant early in the interaction, subject to jurisdiction-specific legal review.

### FR-06 — Conversation boundaries
The agent follows a constrained instruction set. It may collect a caller's stated purpose and urgency, answer approved routine questions, and offer to pass on a message. It must not disclose account-holder secrets or make commitments outside explicit permissions.

### FR-07 — Recording consent
Recording is off by default. If enabled, the UI obtains the account holder's explicit choice and provides clear notice to all participants before recording begins. The PoC should support a no-recording mode. Applicable local law and provider-specific requirements must be assessed before external use.

### FR-08 — Transcript
Where supported and enabled, the application captures a speaker-labelled transcript with timestamps or turn ordering. It must distinguish provider-generated transcription from a transcript derived from a stored recording.

### FR-09 — Summary
After a session, the user can request a summary containing purpose, key details, urgency cues, promised follow-ups and unresolved questions. The summary must be labelled AI-generated and remain editable; uncertain details must not be presented as facts.

### FR-10 — Session history
The user can view session metadata and, when enabled, transcripts, summaries and recording availability. Access is restricted to the owning account.

### FR-11 — Delete
The user can delete a session and its stored artefacts. Deletion should include associated object storage and derived records, with failures surfaced and retried safely.

### FR-12 — Latency and cost
Each session records timestamps for connection, first user audio, end of user turn, first agent audio, turn completion and session end where available. Provider usage events are captured and converted to an estimated cost using a dated pricing configuration. Estimated and billed cost must be distinguished.

### FR-13 — Provider adapter
Application workflows consume normalized session, transcript, usage and error events. Provider-specific SDKs, wire formats and credentials stay inside an adapter.

### FR-14 — Data export (later PoC enhancement)
The account holder can export a transcript and summary in a portable text format. Audio export is a separate, explicit action.

## 7. Non-functional requirements

- **Security:** provider master keys remain server-side; short-lived credentials are scoped and expire; all stored artefacts are private by default.
- **Privacy:** collection is minimized; recording is opt-in; retention is documented; deletion is user-accessible.
- **Reliability:** connection failures, disconnections and provider throttling produce actionable UI states and preserve completed artefacts where possible.
- **Latency:** establish a baseline before setting a hard target. Report median and p95 separately; measure time-to-first-audio and interruption response.
- **Accessibility:** keyboard-operable controls, visible focus, captions/transcript and clear permission/error messaging.
- **Portability:** provider-specific data is preserved only where needed; normalized domain events support future vendor changes.
- **Cost control:** configurable session duration and usage limits; show estimates before/after sessions where practicable.
- **Maintainability:** small modular frontend and backend; decisions and provider assumptions are documented.

## 8. Initial PoC acceptance criteria

A PoC candidate is acceptable when all of these can be demonstrated in a browser on HTTPS:

1. User starts a session and grants microphone permission.
2. Agent returns audible speech to a normal user prompt.
3. At least ten consecutive conversational turns work without manual reconnect.
4. User can interrupt the agent and the agent yields without requiring the agent to finish its sentence.
5. Session can be ended immediately, with microphone capture stopped.
6. Recording is not started unless the user explicitly enables it and the relevant participant notice/consent flow has completed.
7. A test session can produce a transcript and an AI-labelled summary, with unsupported/uncertain content marked.
8. Session timestamps and provider usage are captured sufficiently to calculate a reproducible latency and cost estimate.
9. A user can view and delete the stored session artefacts.
10. No provider master key appears in browser code, browser storage, network responses or client logs.
11. A provider failure is displayed gracefully and does not silently enable recording or lose completed artefacts.
12. A repeatable evaluation log captures model/provider, configuration, locale, network, test scenario and results.

Acceptance thresholds for latency, transcript quality and subjective naturalness remain open until a small baseline experiment is run.

## 9. Future call-screening behavior

The eventual call flow may classify a call into: urgent escalation, known/expected call, routine message, spam/sales, or uncertain. These are suggestions to the user, not definitive judgements about a caller. The assistant should ask clarifying questions when intent is unclear and escalate rather than guess.

The account holder must configure:
- Contacts or categories that should always be escalated.
- Topics the assistant may handle.
- Information the assistant may disclose.
- Actions it may or may not perform.
- Quiet hours and escalation channels.
- A human takeover path where the underlying telephony platform supports it.

Emergency, healthcare, legal and financial calls should default to escalation; the PoC should not attempt to triage emergencies.

## 10. Success measures

- Task completion rate for a pre-defined set of routine scenarios.
- Human rating of naturalness and helpfulness.
- Correct interruption recovery rate and false-barge-in rate.
- Median/p95 time-to-first-audio and turn latency.
- Speech recognition error / correction rate on selected languages.
- Session completion and disconnect rates.
- Percentage of summaries with material factual errors.
- Cost per connected minute and per successfully completed scenario.
- User confidence that recording and deletion controls work.

## 11. Open product questions

1. Which language(s) and accent(s) are required for the first evaluation? Tamil and Indian English should be included in a later language test matrix, not assumed to work equally well.
2. Should the PoC support both sides of a conversation transcript, or only the agent and user turns surfaced by the provider?
3. Is recording a must-have acceptance item for the first provider experiment, or can it follow after conversation quality is established?
4. Which notification channel should later call escalation use?
5. What is the default retention period for recordings, transcripts and summaries?
