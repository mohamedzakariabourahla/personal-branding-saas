# Phases & Tasks (Checklist)

Clean architecture and scalability drive every slice (API, frontend, future workers). Current mode: local-first; deploy when useful.

## Current Snapshot
- [x] Meta connector working.
- [ ] TikTok: blocked by sandbox DNS; resume when reachable or with post-review prod creds.
- [ ] YouTube: untested (no developer account yet).
- [x] Auth/onboarding/email verification; mail via Resend HTTP; Redis rate limiting; Postgres via Flyway; AES-GCM for platform tokens.
- [x] Secrets moved to envs; prod vs local profiles explicit; mail/Redis handling hardened.
- [x] Auth cookies: refresh tokens now cookie-only; access tokens stay in-memory on the client.
- [x] Publishing scheduler backbone (Postgres persistence + Redis delayed queue; worker stub feature-flagged).

## Completed
- [x] Mapping cleanup with shared mappers + tests.
- [x] Config split; auth/verification tests in CI.
- [x] Frontend auth UX fixes and Playwright smoke tests.
- [x] Device-scoped refresh tokens; Redis-backed login/refresh/email throttling; email normalization.

## In Progress / Next
### Phase 5 - Auth & Session Hardening
- [x] Move refresh tokens out of browser storage; rely on secure cookies/backend sessions.
- [x] Re-verify `/auth/refresh` cookie-only path and clear any stored refresh tokens on logout.
- [ ] Add regression coverage for cookie-only auth (API + Playwright smoke).
- [ ] Surface email dispatch failures to users (frontend) when provider rejects mail.

### Phase 6 - Social Platform Connectivity
- [ ] Resume TikTok once DNS/creds are available.
- [ ] Obtain YouTube dev account and test OAuth.
- [ ] Build publishing UI shell (nav, provider cards, callback pages); surface `/privacy` and `/terms`.
- [ ] Define Kafka topics/worker skeleton for publish requests/results.

### Phase 7 - Publishing Orchestrator & Scheduling
- [ ] Wire queue consumer to PlatformPublisherRegistry; record attempts/retries/dead-letter; expose job list endpoints/UI (jobs API with attempts + retry added; worker resolves connection/auth, calls publishers, records attempts/metrics; needs media handling + provider responses).
- [ ] Calendar/queue UI reflecting scheduled + delivered posts.
- [ ] Asset primitives (uploads) and validation.

### Phase 8 - Content Intelligence & Generation
- [ ] Template catalog schema + seeds (tones, pillars, cadences).
- [ ] `ContentGenerationEngine` + OpenAI adapter behind async workflow.
- [ ] Editing UX (scripts/captions/carousels) with history; guardrails: quotas, cost tracking, redaction.

### Phase 9 - Analytics & Insights
- [ ] Fact tables per platform + ingestion jobs.
- [ ] Aggregations (7/30-day growth, engagement funnels) feeding dashboard.
- [ ] Learning/insight recommendations based on analytics signals.

### Phase 10 - Teaming & Learning Experience
- [ ] Approvals, roles/permissions, assistant access.
- [ ] Expanded learning content; notifications (digest, anomaly alerts, next-best actions).

## Environment & Security
- [x] Backend envs: Postgres/Redis, mail (Resend) API key/from, JWT secrets, AES master key+id, OAuth creds per provider; set via `.env`/Render.
- [x] Frontend envs: only `NEXT_PUBLIC_API_URL` needed.
- [x] Profiles: set `SPRING_PROFILES_ACTIVE` explicitly (`local` for dev, `prod` for deploy).
- [ ] CORS/CSP/frame-ancestors: ensure configured per environment (keep filters aligned to security guidance).
- [x] Encryption: AES-GCM for platform tokens; store ciphertext + IV + key id.
- [x] Mail: HTTP API (Resend); verified sender required.

## Connector Setup (abridged)
- Meta/Instagram: Business app with “Manage messaging and content on Instagram”; perms `business_management, instagram_basic, instagram_content_publish, pages_manage_posts, pages_read_engagement, pages_show_list`; link IG business account to a Page; add Instagram testers; redirect `http(s)://<host>/meta/callback`.
- TikTok: Sandbox endpoints currently unreachable; when possible use client key/secret + redirect `https://<host>/tiktok/callback`; testers required pre-review.
- YouTube: Google Cloud project + YouTube Data API; web OAuth client with origin/redirect `http://localhost:3000` and `/youtube/callback`; add test users while consent screen in Testing.

## Testing
- [x] Unit/integration for auth/verification, mappers, rate limiters.
- [x] Playwright smoke: login redirects, onboarding gating, API contract checks (`npm run test:e2e`, requires seeded `E2E_TEST_EMAIL` and data).

## Blockers
- TikTok: sandbox DNS not resolvable on current networks; awaiting reachable DNS or production credentials.
- YouTube: no developer account yet to exercise OAuth flow.
- Auth: add automated checks for cookie-only refresh/logout.
