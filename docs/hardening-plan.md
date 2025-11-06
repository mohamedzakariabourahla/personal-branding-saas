# Hardening Roadmap

This file tracks the stabilisation work before new SaaS features land.

> **Suggested starting point:** begin with Phase 5 so every upcoming feature sits on a hardened auth/session foundation. Move directly into Phase 6 afterwards to unblock schema work for content, scheduling, and analytics.

## Completed Phases

### Phase 1 - Mapping Cleanup
- [x] Introduce shared mapper layer for persistence adapters (`infrastructure/mapping`).
- [x] Port remaining adapters to use the shared mappers.
- [x] Add mapper-focused unit tests.

### Phase 2 - Asynchronous Messaging & Rate Limiting
- [x] Dispatch emails via async events (with retries/backoff).
- [x] Add Redis-backed rate limiter implementation.
- [x] Emit structured audit events for auth flows.

### Phase 3 - Configuration, Secrets & Tests
- [x] Split environment configuration, remove sample secrets.
- [x] Add unit/integration coverage for auth + verification flows.
- [x] Wire tests into CI.

### Phase 4 - Frontend Stabilisation
- [x] Prefill `/verify-email` from query parameters and hide redundant inputs.
- [x] Centralise error-code to copy mapping.
- [x] Add unit + Playwright smoke tests for auth flows.
  - Playwright smoke suite now covers login redirects, API contract checks, onboarding gating, and runs in CI (`npm run test:e2e`, requires seeded `E2E_TEST_EMAIL` and optional mutation flags).

## Upcoming Phases

### Phase 5 - Auth & Session Hardening *(start here)*
- [x] Move browser session handling to secure cookies or a backend session store; keep refresh tokens off `localStorage`.
- [x] Normalise and lower-case emails at persistence boundaries; add database constraint + migration.
- [x] Redesign refresh token storage as device-scoped so multiple sessions (web, mobile, assistants) can coexist.
- [x] Replace in-memory login/refresh rate limiters with Redis-backed implementations and surface metrics in Grafana.
- [x] Expand frontend error handling to reuse `resolveAuthError` for password reset failures and future contexts.

### Phase 6 - Data Model Foundations
- [ ] Produce ERD + migrations covering content templates, generated content assets, scheduled posts, platform connections, analytics facts.
- [ ] Extend onboarding/person profile with voice, pillar priorities, asset preferences, example posts.
- [ ] Seed foundational data for templates, tones, and recommended cadences.
- [ ] Stand up background job infrastructure (Redis queue, Temporal, or Quartz) for async generation and scheduling workflows.
- [ ] Document data retention/encryption requirements for external platform tokens and user-generated media.

### Phase 7 - Platform Integration Readiness
- [ ] Build connector abstraction for Meta, TikTok, YouTube, Google Drive (shared interface, per-provider adapters).
- [ ] Store third-party credentials encrypted (KMS / Key Vault) with rotation policy and audit logging.
- [ ] Implement token refresh + webhook/cron ingestion flows per provider; cover with contract and integration tests.
- [ ] Provide staging sandboxes and feature flags to gate early adopters during integration rollout.

### Phase 8 - Content Creation Experience
- [ ] Implement AI idea generation pipeline that blends onboarding profile + template prompts; persist state for retries.
- [ ] Build UI surfaces for generated scripts, carousel outlines, captions/hashtags with editing and feedback loop.
- [ ] Cache generation history and allow bookmarking/re-generation; expose usage metrics per user.
- [ ] Add guardrails (rate limits, cost tracking) around model consumption.

### Phase 9 - Publishing & Scheduling
- [ ] Deliver unified scheduling UI (calendar + queue) across supported platforms.
- [ ] Wire background workers to publish on schedule, handle retries, and surface delivery receipts/errors.
- [ ] Support asset management (uploads, Google Drive linking, repurposed content) with validation and preview.
- [ ] Introduce optional approval workflow for teams and assistants (role-based).

### Phase 10 - Analytics & Learning
- [ ] Ingest platform analytics nightly; persist raw facts and expose aggregated views (rolling 7/30-day, growth curves).
- [ ] Build dashboard widgets for performance, audience growth, content mix effectiveness.
- [ ] Integrate learning hub (headless CMS + MDX renderer) with contextual recommendations based on analytics gaps.
- [ ] Implement notification framework for insights (weekly digest, anomaly alerts, "what to do next" nudges).

---
*Updated automatically as tasks complete.*
