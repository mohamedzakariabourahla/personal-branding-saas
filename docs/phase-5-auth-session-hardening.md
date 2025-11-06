# Phase 5 - Auth & Session Hardening Plan

This document breaks Phase 5 into actionable work items while keeping the existing architecture boundaries intact (presentation → application services → domain → infrastructure).

## Objectives

1. **Secure session handling** — eliminate browser `localStorage` as a source of truth, move to HttpOnly cookies or a backend session store, and minimise credential exposure.
2. **Canonical user identities** — normalise emails before persistence, enforce database constraints, and block duplicates regardless of casing or whitespace.
3. **Device-scoped refresh tokens** — support concurrent sessions (web, mobile, assistants) without revoking existing tokens; maintain full auditability.
4. **Horizontally scalable rate limiting** — replace in-memory login/refresh throttles with Redis-backed implementations and expose metrics.
5. **Consistent error surfaces** — ensure the frontend reuses central error mapping for password reset and future auth contexts.

## Target Architecture

```
Next.js (client) --XHR--> Spring Boot API
                      ↑      │
         HttpOnly Cookies <──┘
                      │
                   Session Store (Redis/Postgres)
```

- **Refresh tokens** move into HttpOnly cookies (e.g. `pb_refresh`) issued by `/auth/login` and rotated on refresh.
- **Access tokens** remain short-lived JWTs returned in the response body for in-memory use, or optionally a second HttpOnly cookie if we later add SSR.
- **Session lookups** on the frontend fetch from `/auth/refresh` (or a future `/auth/session`) and hydrate `AuthSessionProvider` without touching `localStorage`.
- **Repositories/services** remain layered; new adapters extend existing interfaces.

## Work Breakdown

### 1. Session Handling Overhaul

1. **✔ Done** Add API support for HttpOnly refresh cookies.
2. **✔ Done** Update `/auth/refresh` to read cookies when the body token is absent and rotate atomically.
3. **⏳ Optional** Add `/auth/session` endpoint for SSR use-cases (cookies already bootstrap SPA hydration).
4. **✔ Done** Frontend now hydrates via cookies, updates tokens through `/auth/refresh`, and clears cookies on logout.

### 2. Email Normalisation

1. **✔ Done** Introduced `EmailNormalizer` and applied it across controllers, services, and repositories.
2. **✔ Done** Added Flyway migration `V5__normalize_user_emails.sql` to clean existing data and enforce case-insensitive uniqueness.
3. **✔ Done** Updated mappers/tests to cover mixed-case duplicates.

### 3. Device-Scoped Refresh Tokens

1. **✔ Done** Extended the `RefreshToken` aggregate and persistence layer with device metadata and `last_used_at`.
2. **✔ Done** Added migration `V6__device_scoped_refresh_tokens.sql` and per-device uniqueness constraints.
3. **✔ Done** AuthService now issues per-device tokens, enforces session limits, and exposes `GET/DELETE /api/auth/sessions`.
4. **✔ Done** Frontend login submits `deviceName`; new API responses surface device metadata.

### 4. Redis Rate Limiters

1. **✔ Done** Implemented `RedisLoginRateLimiter` and `RedisRefreshRateLimiter`, gated via configuration.
2. **✔ Done** Added configuration toggles (`security.login.rate-limiter`, `security.refresh.rate-limiter`) with Redis defaults and in-memory fallbacks for tests.
3. **✔ Done** Added unit coverage for Redis-backed limiters and wired metrics.

### 5. Frontend Error Handling

1. **✔ Done** Added a `"password-reset"` context to `resolveAuthError`.
2. **✔ Done** Updated the password reset hook to surface token-expired/invalid messages.

## Sequencing & Release Notes

- **Sprint 1** (complete): Cookie-based sessions and frontend integration.
- **Sprint 2** (complete): Email normalisation and device-scoped refresh tokens.
- **Sprint 3** (complete): Redis-backed rate limiters and observability hooks.
- **Sprint 4** (current/future): Additional polish (optional `/auth/session`, UX refinements).

Each sprint shipped with:
- Migration scripts and rollback strategies.
- Unit/integration coverage (AuthService, controllers, repositories).
- Manual verification checklist (login/logout, refresh, password reset, multi-device scenarios).

## Risks & Mitigations

- **Cookie vs SPA routing** — ensure Next.js proxies API on the same origin or set `withCredentials` on Axios (already handled).
- **Migration safety** — normalising emails required duplicate audits; migrations are idempotent.
- **Legacy sessions** — requests with JSON refresh tokens remain supported during the transition.
- **Redis availability** — rate limiters degrade gracefully when Redis is unavailable; metrics help monitor impact.

---
**Next action:** With Phase 5 tasks closed out, move towards Phase 6 (schema groundwork for content and scheduling). Tracking continues in `docs/hardening-plan.md`.
