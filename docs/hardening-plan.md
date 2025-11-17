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
- Before wiring additional providers, make sure you’ve completed the checklist in [`docs/meta-connector-setup.md`](meta-connector-setup.md); it captures the required Meta/TikTok Business Manager steps, tester roles, and redirect URIs used by the shared OAuth flows.
- [x] Finalise domain contracts for `PlatformPublisher`, credential stores, and registry wiring (`docs/architecture.md` updated).
- [x] Ship Flyway migration for platform connections/tokens + AES-GCM encryption service.
- [x] Implement TikTok OAuth + REST client + `/api/platforms/tiktok/*` endpoints (reusable DTOs).
- [x] Build Meta/Instagram adapter (OAuth + publish scaffolding) and extend the shared flow to cover YouTube (PKCE + multi-channel selection, documented in [`docs/meta-connector-setup.md`](meta-connector-setup.md) and [`docs/youtube-connector-setup.md`](youtube-connector-setup.md)).
- [ ] Add left-rail in-app navigation, `/home/publishing` UI, provider cards, and OAuth callback pages; surface `/privacy` and `/terms` for partner reviews.
- [ ] Establish Kafka topics for `platform.publish.{request,result}` and add worker skeleton so adapters can enqueue publishes.

**Recommended execution order**
1. ✅ Backend foundation (schema + encryption + shared contracts).
2. ✅ First provider slice (TikTok OAuth + connection APIs).
3. **Current**: Minimal deployable UX
   - Drawer-based app shell for logged-in users.
   - Publishing screen with provider cards, connection list, and CTA hooks.
   - Static `/privacy` + `/terms` pages + documentation of staging URLs.
4. Add Meta/Instagram + YouTube adapters and extend UI buttons once review credentials are ready.
5. Stand up Kafka worker + analytics ingestion to fan out publish/sync jobs.

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
