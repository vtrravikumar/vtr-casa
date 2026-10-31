# Data Privacy, Consent and Recording

**Status:** Design requirements; legal review required before external telephone use  
**Last reviewed:** 2026-10-31

## 1. Privacy posture

Voice conversations can contain sensitive personal, financial, health, identity and relationship information. Treat audio and transcripts as sensitive personal data even when the product is initially a single-user PoC.

Collect the minimum data necessary for the feature the user enabled. Do not use conversation audio or transcripts to create an account-holder profile or train product models by default.

## 2. Data categories and purpose

| Data | Purpose | Default |
|---|---|---|
| Microphone stream | Real-time conversation | In-memory streaming during an active session |
| Raw audio recording | Replay / audit / transcript source | Off |
| Live transcript | Conversation display and summary source | Do not persist unless user enables transcript history |
| Summary | Convenient session recap | Generate only on user request or explicit setting |
| Session metadata | Lifecycle, latency, cost and errors | Minimized metadata retained |
| Consent events | Demonstrate what the user chose and when | Persist minimal event record |
| Voice profile / source samples | Future opt-in voice personalization | Not collected in PoC |

## 3. Consent model

Separate the following choices; consent to one does not imply consent to all:

1. Microphone access for live interaction.
2. Recording of the conversation.
3. Persistence of a transcript.
4. Generation and persistence of a summary.
5. Future voice profile creation and use.

The browser must explain why microphone access is needed and request permission after a user gesture. If permission is denied, no capture begins.

Recording must be off by default. Before recording:
- show a clear control and explanation to the account holder;
- disclose the recording to all other participants in a clear and timely manner;
- obtain any consent required by applicable law and provider terms;
- provide a clear recording indicator and immediate stop control;
- do not begin or continue if consent/notice requirements are not satisfied.

For the browser PoC, all participants should be able to see or hear the recording notice. Do not implement covert capture or hidden recording. Account-holder consent alone does not automatically satisfy the rights of another participant.

For later telephone use, consent and disclosure requirements vary by jurisdiction and call context. Obtain qualified legal review for India and every target market before enabling call recording. This document is product guidance, not legal advice.

## 4. Recording architecture

- Prefer no recording during initial conversation-quality testing unless the scenario specifically requires it.
- When recording is enabled, capture only the active session and stop automatically on session end or consent withdrawal.
- Encrypt in transit and use private encrypted storage at rest with provider/platform controls.
- Store recording objects in private R2 buckets; never expose bucket credentials or public object paths.
- Use opaque object keys that do not contain phone numbers, names or other direct identifiers.
- Enforce per-session and per-account size/duration quotas.
- Do not put audio bytes in D1 or general application logs.
- Validate uploaded content and expected media types; use streaming/chunked handling to avoid unbounded memory buffering.
- Store the consent version and timestamp linked to the session, but avoid collecting unnecessary information about other participants.

## 5. Transcript and summary handling

Transcripts may contain more sensitive information than the audio because they are searchable. Keep transcript persistence separately configurable. A transcript generated from live provider events should be labelled as such; a post-session transcript should identify the recording/transcription pipeline. Do not treat model transcription as authoritative.

Summaries must be AI-labelled and should:
- distinguish what the speaker said from facts independently verified by the application;
- retain uncertainty for names, dates, amounts and intent;
- avoid adding facts not present in the transcript;
- be editable and deletable by the account holder.

## 6. Data retention

Proposed initial policy (values require owner approval before implementation):

| Data | Proposed default |
|---|---|
| Raw recording | Not stored unless user opts in; if stored, 30 days pending owner confirmation |
| Transcript | Not persisted unless enabled; 90 days pending owner confirmation |
| Summary | 90 days pending owner confirmation |
| Session metadata / consent audit | 180 days pending owner confirmation |
| Operational logs | Shortest period that supports security and troubleshooting; redact content |

Retention must be implemented as an application policy, not only as a provider setting. Provider-side retention, backups, subprocessors and deletion timing must be checked for each chosen vendor.

The initial single-user PoC should expose the retention policy and a delete action even if the user is the only account.

## 7. Deletion

Delete action should:
1. Mark the session as deletion-pending and make it inaccessible to normal UI queries.
2. Delete audio objects and derived exports from R2.
3. Delete or anonymize transcripts, summaries and session records according to the deletion policy.
4. Request deletion from the provider when the provider stores conversation data and supports a deletion API.
5. Record completion or partial failure without retaining the sensitive content.
6. Retry safely and surface failures to the account holder.

Document backup expiry and provider deletion caveats. Avoid claiming instant deletion from all backups or provider systems without evidence.

## 8. Security requirements

- TLS for network transport and platform-managed encryption at rest.
- Short-lived provider credentials; master credentials remain in backend secret storage.
- Authenticate every history, recording, transcript and deletion operation.
- Verify ownership at object-key and database query boundaries.
- Use CORS only as a browser access policy, never as user authentication.
- Protect against CSRF where cookie-based auth is used.
- Apply rate limits, session duration caps and recording quotas.
- Sign and verify provider webhooks; defend against replay and duplicate events.
- Redact transcript content, tokens, phone numbers and audio from default logs.
- Keep audit records for recording consent changes and deletion requests.
- Separate development/test data from real personal conversations.
- Review vendor data-processing terms, subprocessors, regional processing, training use and retention.

## 9. AI voice personalization safeguards (future only)

Not part of the PoC. Before enabling:
- verify the account holder's identity and control of source recordings;
- collect explicit, purpose-specific consent;
- prohibit cloning or imitation of third-party voices without verifiable authorization;
- disclose generated voice use to people interacting with the assistant;
- restrict voice profile creation, access, export and use;
- provide an immediate disable/revoke control and documented deletion;
- prevent use of cloned voice as authentication or authorization;
- monitor abuse and provide reporting/response processes;
- review provider terms and applicable law.

## 10. Provider-specific privacy checks

- **OpenAI:** verify current API data controls, abuse-monitoring retention, regional processing eligibility and any recording/transcription storage used.
- **Deepgram:** verify project retention, transcript/recording settings, model processing region and deletion semantics.
- **ElevenLabs:** explicitly configure audio-saving and conversation-retention settings; current documentation describes a two-year default for conversation data, with configurable override. Verify plan-specific behavior and deletion propagation.

## 11. iOS and telephone privacy

Apple's App Review Guidelines require explicit consent and clear visual and/or audible indication when an app records, logs or otherwise makes a record of user activity, including microphone input. This is relevant to app UX but does not resolve all local call-recording law. The browser PoC should use the same high-transparency standard.

For actual telephone calls, privacy design must consider both the account holder and caller, plus telecom provider terms, data residency and applicable interception/recording laws.

## 12. References

- [Apple App Review Guidelines — 2.5.14 and 5.1](https://developer.apple.com/app-store/review/guidelines/)
- [ElevenLabs Privacy](https://elevenlabs.io/docs/eleven-agents/customization/privacy)
- [ElevenLabs Retention](https://elevenlabs.io/docs/eleven-agents/customization/privacy/retention)
- [OpenAI API data controls](https://developers.openai.com/api/docs/guides/your-data)
- [Cloudflare R2 access from Workers](https://developers.cloudflare.com/r2/api/workers/workers-api-usage/)

## 13. Decisions still required

- Approve or revise retention periods.
- Decide whether Phase 1 must include recording or whether recording follows successful conversation validation.
- Choose consent wording and how it is audibly conveyed to a remote participant.
- Confirm target jurisdictions and obtain qualified legal review.
- Confirm the selected provider's storage, retention, deletion and data-use controls.
