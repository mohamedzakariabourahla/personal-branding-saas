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

### Phase 5 - Auth & Session Hardening *(complete)*
- [x] Move browser session handling to secure cookies or a backend session store; keep refresh tokens off `localStorage`.
- [x] Normalise and lower-case emails at persistence boundaries; add database constraint + migration.
- [x] Redesign refresh token storage as device-scoped so multiple sessions (web, mobile, assistants) can coexist.
- [x] Replace in-memory login/refresh rate limiters with Redis-backed implementations and surface metrics in Grafana.
- [x] Expand frontend error handling to reuse `resolveAuthError` for password reset failures and future contexts.

### Phase 6 - Social Platform Connectivity *(in progress)*
- [ ] Finalise domain contracts for `PlatformPublisher`, `PlatformCredentialRepository`, and `PlatformPublisherRegistry`; document wiring rules in `docs/architecture.md`.
- [ ] Add Flyway migrations for platform credentials (encrypted columns, audit trails) and OAuth state tracking.
- [ ] Build infrastructure adapters for Meta/Instagram first (OAuth, publishing, analytics pull) behind the shared interfaces; add feature-flagged controllers/services.
- [ ] Expose `/home/publishing` UI to list connected platforms and surface “Connect Instagram/TikTok/YouTube” flows; persist connection status via new APIs.
- [ ] Establish Kafka topics for `platform.publish.request` / `platform.publish.result` and add worker skeleton that can fan out publish commands per provider.

**Recommended execution order**
1. Schema + encryption groundwork (Flyway + property-based encryption wiring).
2. Domain/application contracts + Spring configuration (registry, credential services).
3. Provider integrations slice-by-slice:
   - TikTok OAuth + connections (done), reuse same controllers + DTOs.
   - Meta/Instagram OAuth (reuse shared DTOs, swap properties), then YouTube.
   - Add provider-specific `PlatformPublisher` implementations that publish to Kafka worker topics.
4. Frontend `/home/publishing` connection UI wired to the generic `/api/platforms/*` endpoints.
5. Kafka topics + worker skeleton for publish commands, then extend adapters for analytics ingestion.

### Phase 7 - Publishing Orchestrator & Scheduling
- [ ] Implement `PublishingJobScheduler` and delivery receipt models; persist scheduled posts, retries, and audit events.
- [ ] Extend worker module to handle queued jobs (Redis delayed queue initially, swappable for Quartz/Temporal).
- [ ] Deliver dashboard calendar/queue that reflects scheduled + delivered posts from multiple providers.
- [ ] Add asset management primitives (media uploads, Google Drive connectors) and validation hooks before publishing.

### Phase 8 - Content Intelligence & Generation
- [ ] Build Template Catalog schema (templates, tones, pillars, cadences) plus seed data via repeatable migrations.
- [ ] Implement `ContentGenerationEngine` abstraction and OpenAI adapter (ChatGPT-5) behind Kafka-based workflows.
- [ ] Surface editing UX for scripts/captions/carousels with history + bookmarking.
- [ ] Add guardrails: generation quotas, cost tracking, redaction of sensitive prompt data.

### Phase 9 - Analytics & Insights
- [ ] Model raw fact tables per platform and nightly ingestion jobs (Kafka consumers + batch fetchers).
- [ ] Produce aggregated views (7/30-day growth, engagement funnels) feeding the dashboard widgets.
- [ ] Integrate contextual learning hub recommendations based on analytics gaps and deliver proactive insights/notifications.

### Phase 10 - Teaming & Learning Experience
- [ ] Introduce approval workflows, roles/permissions, and assistant access control.
- [ ] Expand learning hub content (filming guides, tonality tips) with personalization hooks.
- [ ] Implement notification framework (weekly digest, anomaly alerts, “next best action”) powered by analytics signals.

---
*Updated automatically as tasks complete.*
