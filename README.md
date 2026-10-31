# VTR-CASA — Call Assistant & Screening Agent

VTR-CASA is an experimental AI-powered personal call assistant. Its long-term goal is to handle selected incoming calls on a user's behalf, understand caller intent and urgency, resolve routine enquiries, and surface calls that need the user's attention.

**Current milestone: Phase 0 — documentation only.** This repository currently contains the product and technical foundation. No application, infrastructure, external provider, or deployment is implemented or configured.

## Start here

- [Product requirements](docs/product-requirements.md)
- [System architecture](docs/system-architecture.md)
- [Voice agent design](docs/voice-agent-design.md)
- [AI provider evaluation](docs/ai-provider-evaluation.md)
- [Privacy, consent and recording](docs/data-privacy-and-recording.md)
- [Roadmap](docs/future-roadmap.md)
- [Architecture decisions](docs/decisions/)

## Current direction

The first proof of concept should be a browser-based voice conversation: microphone input, natural speech output, conversational turn-taking and interruption, explicitly consented recording, transcript and summary generation, and measurable latency and estimated cost.

React + TypeScript and Cloudflare Workers are proposed for the application shell and control plane. OpenAI Realtime, Deepgram Voice Agent and ElevenLabs Conversational AI are candidates for the voice runtime. These are not final vendor decisions. See the provider evaluation and ADRs for trade-offs and the experiments required before selection.

## Scope guardrails

- Do not begin cellular call interception or assume that an iOS app can access ordinary Phone app call audio.
- Do not put long-lived provider secrets in the browser.
- Do not record by default. Recording must be an explicit, understandable choice with a visible/audible indication and a clear stop control.
- Do not build personalized voice cloning in the initial PoC. Future voice personalization requires verified account-holder authorization and anti-impersonation safeguards.
- Keep provider-specific protocols behind an adapter boundary so a provider can be replaced without rewriting product workflows.

## Development model

The project is being developed directly in GitHub. Changes should use focused branches and pull requests, with review and CI checks where available. Until the project owner has a local development environment, do not require local terminal commands or IDE steps.

## Documentation status

Last reviewed: 2026-10-31. Provider prices and platform capabilities change; the evaluation document links to official sources and records the date of review. Recheck pricing, terms, regional availability and platform rules before implementation or launch.
