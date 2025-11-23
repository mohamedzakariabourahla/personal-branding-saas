# Security & Headers

Focus areas:
- Keep secrets out of git; load via env only.
- Use HttpOnly cookies for refresh tokens; keep them out of browser storage.
- Harden CORS/CSP/frame-ancestors per environment.
- Encrypt platform tokens (AES-GCM).
- Rate limit auth flows with Redis; fail safely when dependencies break.

## Sessions & Tokens
- Refresh tokens should live in HttpOnly cookies (`pb_refresh`) set by the API; avoid persisting them in `localStorage`/`sessionStorage`.
- Access tokens can remain short-lived in memory; optional second cookie if SSR is added later.
- Device-scoped refresh tokens supported; list/revoke via `/api/auth/sessions`.
- Endpoints: `/auth/login` issues cookies; `/auth/refresh` reads cookies when body is absent; logout clears cookie.

## CORS / CSP / Frame Ancestors
- Configure `APP_CORS_ALLOWED_ORIGINS` per environment.
- Set Content-Security-Policy to restrict scripts/styles/connect accordingly.
- Set `frame-ancestors` to block embedding except trusted origins.
- Security headers are enforced via filters; confirm values at deploy time.

## Encryption
- AES-GCM master key (`APP_ENCRYPTION_KEY`, `APP_ENCRYPTION_KEY_ID`) encrypts platform tokens; store ciphertext + IV + key id.
- Rotate keys by issuing a new key id and re-encrypting tokens; keep old key until migration is done.

## Rate Limiting & Resilience
- Login/refresh/email verification throttled via Redis; metrics emitted.
- On dependency failures (mail, Redis), surface clear errors; avoid silent fail-open for abuse-sensitive paths.

## Mail
- Using HTTP provider (Resend); `MAIL_FROM` must be verified.
- Map provider errors to 503-like responses; avoid blocking auth flows without feedback.

## Secrets & Profiles
- Set `SPRING_PROFILES_ACTIVE` explicitly (`local` or `prod`).
- Provide Postgres/Redis/Mail/JWT/Encryption/OAuth creds via env or platform secret store; never commit real values.
