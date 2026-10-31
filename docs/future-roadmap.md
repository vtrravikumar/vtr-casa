# Future Roadmap

**Status:** Proposed sequence  
**Last reviewed:** 2026-10-31

This roadmap is ordered to validate the core voice interaction before investing in telephone routing or native mobile capabilities. Phase exits depend on evidence, not calendar dates.

## Phase 0 — Documentation (current)

**Scope**
- Product requirements and exclusions.
- Proposed system architecture and provider boundary.
- Voice-agent interaction design.
- Provider and platform evaluation.
- Recording, consent, privacy and retention requirements.
- Architecture Decision Records.

**Exit criteria**
- Owner reviews requirements and open questions.
- Provider evaluation sources and prices are current.
- Initial architecture and security boundaries are understood.
- Phase 1 experiment criteria and test scripts are agreed.

## Phase 1 — Browser voice PoC

**Scope**
- React + TypeScript browser UI.
- User-initiated microphone access.
- Real-time listen/speak conversation.
- Turn-taking and interruption handling.
- Session start/end and resilient error states.
- Consent-gated recording.
- Transcript and AI-labelled summary.
- Session timing, provider usage and estimated cost.
- Minimal session history and deletion.
- One provider selected for the first implementation, with adapter contract tests.

**Exit criteria**
- Meet PoC acceptance criteria in product requirements.
- Repeat tests on agreed browsers and target network conditions.
- Cost and latency results are reproducible.
- Privacy, recording, deletion and provider terms reviewed.

## Phase 2 — Voice agent improvements

**Scope**
- Stronger conversational policies and configurable personality.
- Better interruption and endpointing tuning.
- Multilingual support and target accent evaluation.
- Improved transcript quality and summary controls.
- User-specific knowledge and preferences with explicit configuration.
- Human escalation simulation and notification prototype.

**Exit criteria**
- Measurable improvement in task completion and conversation quality.
- Language-specific quality baselines available.
- No increase in unsafe disclosure or false escalation beyond agreed thresholds.

## Phase 3 — Personalized assistant

**Scope**
- Opt-in account-holder voice personalization, using only properly authorized source samples.
- Behavioral preferences (brevity, greeting, tone, language and escalation rules) configured by the user.
- Voice preview, use permissions, disable/revoke and deletion.
- Misuse monitoring and AI voice disclosure.

**Exit criteria**
- Identity/authorization, provider terms, anti-impersonation and legal reviews complete.
- Voice profile access and use auditable.
- Revocation and deletion tested end to end.

**Not a cloning feature for arbitrary people.** The account holder's own voice is in scope only with explicit authorization.

## Phase 4 — Telephone integration

**Scope**
- Dedicated AI telephone number.
- Cloud telephony/PSTN or SIP provider evaluation.
- Inbound call webhook and media stream integration.
- Call routing, transfer, voicemail/message capture and escalation.
- Caller disclosure, call recording consent and jurisdiction controls.
- Provider failover and emergency fallback.

**Exit criteria**
- End-to-end tests with a controlled test number.
- Legal and telecom-provider approval for target markets.
- Human takeover path and service outage behavior tested.
- Recording and retention controls verified with the telephony provider.

## Phase 5 — iOS application

**Scope**
- Companion app for account configuration and session history.
- Push notifications for calls requiring attention.
- Call-forwarding setup guidance where supported by carriers.
- VoIP calling integration for calls owned by VTR-CASA's service.
- Call history and takeover UX.
- Call Directory capabilities only where appropriate and supported.

**Exit criteria**
- App Review and entitlement requirements understood.
- Supported functionality distinguished from carrier-level call forwarding.
- User permissions and privacy labels reviewed.
- No claims of direct ordinary cellular audio access.

## Phase 6 — Advanced call screening

**Scope**
- Integrations with supported carrier, telephony and iOS mechanisms.
- Trusted-contact bypass and configurable call categories.
- Improved spam/sales handling with uncertainty and review.
- Warm transfer / call bridge / callback mechanisms where providers support them.
- Regional telecom and privacy adaptations.

**Exit criteria**
- Each integration has an explicit platform/provider capability basis.
- Screening results can be overridden by the user.
- Unknown or uncertain cases escalate rather than silently block.
- Reliability, security, privacy and legal requirements met per market.

## Cross-cutting work

- Provider adapter and capability matrix.
- Authentication, abuse prevention and audit.
- Cost budgets and monitoring.
- Data retention and deletion.
- Accessibility and multilingual quality.
- Incident response and provider outage behavior.
- Documentation and ADR updates.

## Key external dependency

Ordinary cellular call audio is not generally available to a third-party iOS app through public APIs. CallKit supports integration and management of the app's own VoIP calls; Call Directory provides caller identification/blocking experiences, not a general-purpose cellular audio tap. Therefore, telephone screening must be evaluated via carrier call forwarding, cloud telephony/PSTN, supported system features and any region-specific mechanisms—not assumed to be a direct iOS microphone feature.
