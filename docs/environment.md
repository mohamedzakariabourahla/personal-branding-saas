# Environment Configuration

This project has two deployable apps:

- `personal-branding-saas-api` (Spring Boot)
- `personal-branding-saas-web` (Next.js)

Every secret or endpoint must be provided via environment variables. Use this file as the single source of truth when creating `.env` files, configuring Render/Vercel, or sharing credentials with teammates.

> **Tip:** copy the sample files (`personal-branding-saas-api/.env.example`, `personal-branding-saas-web/.env.example`) and fill in the values that apply to your environment. Never commit the populated `.env` files.

---

## Backend (`personal-branding-saas-api`)

| Variable | Description | Required | Example |
| --- | --- | --- | --- |
| `SPRING_DATASOURCE_URL` | JDBC URL for Postgres | ✅ | `jdbc:postgresql://localhost:5432/personal-branding-saas` |
| `SPRING_DATASOURCE_USERNAME` / `SPRING_DATASOURCE_PASSWORD` | Postgres credentials | ✅ | `postgres` |
| `REDIS_HOST` / `REDIS_PORT` / `REDIS_PASSWORD` | Redis connection info | ✅ (host/port) | `localhost` / `6379` |
| `APP_CORS_ALLOWED_ORIGINS` | Comma-separated list of allowed front-end origins | ✅ | `https://pb.local,https://app.vercel.app` |
| `SECURITY_JWT_SECRET` | Signing key used for access tokens | ✅ | `super-secret` |
| `SECURITY_JWT_ISSUER` | Issuer claim for tokens | ⛔ (defaults to `personal-branding-saas`) | `upersona-prod` |
| `APP_ENCRYPTION_KEY` | 32-byte AES-GCM key for encrypting platform tokens | ✅ | `0123456789abcdef0123456789abcdef` |
| `APP_ENCRYPTION_KEY_ID` | Identifier for the encryption key | ✅ | `prod-key` |
| `MAIL_HOST` / `MAIL_PORT` / `MAIL_USERNAME` / `MAIL_PASSWORD` | SMTP settings for transactional emails | ✅ for environments that send mail | `smtp.mailtrap.io` etc. |
| `MAIL_FROM` | From address used in notification emails | ✅ | `no-reply@upersona.app` |
| `APP_MAIL_RESET_BASE_URL` / `APP_MAIL_VERIFY_BASE_URL` | Front-end URLs embedded in auth emails | ✅ | `https://app.upersona.app/reset-password?token=` |

### OAuth providers

| Variable | Description |
| --- | --- |
| `META_CLIENT_ID`, `META_CLIENT_SECRET`, `META_REDIRECT_URI`, `META_SCOPES`, `META_AUTH_BASE_URL`, `META_GRAPH_BASE_URL` | Instagram Graph API credentials + redirect URL registered in Facebook Developer Portal. |
| `TIKTOK_CLIENT_KEY`, `TIKTOK_CLIENT_SECRET`, `TIKTOK_REDIRECT_URI`, `TIKTOK_SCOPES`, `TIKTOK_AUTH_BASE_URL`, `TIKTOK_API_BASE_URL` | TikTok Business API credentials + redirect URL registered in TikTok for Developers. |
| `YOUTUBE_CLIENT_ID`, `YOUTUBE_CLIENT_SECRET`, `YOUTUBE_REDIRECT_URI`, `YOUTUBE_SCOPES`, `YOUTUBE_AUTH_BASE_URL`, `YOUTUBE_TOKEN_URL`, `YOUTUBE_API_BASE_URL` | Google Cloud OAuth client for YouTube Data API. |

All redirect URIs must exactly match what is registered in each provider console. For local HTTPS testing you can use a custom domain (e.g., `https://pb.local/tiktok/callback`) as long as it resolves to your machine.

### Optional tuning

The following have safe defaults but can be overridden:

- `SECURITY_LOGIN_*`, `SECURITY_REFRESH_*` — rate limiter thresholds
- `SECURITY_JWT_ACCESS_TTL`, `SECURITY_JWT_REFRESH_TTL`
- `APP_SECURITY_CSP`, `APP_SECURITY_FRAME_ANCESTORS`
- `SERVER_PORT`, `SERVER_ADDRESS`

---

## Frontend (`personal-branding-saas-web`)

The front-end only needs to know where the API lives. Create `.env.local` (or use Vercel’s env UI) with:

```bash
NEXT_PUBLIC_API_URL=https://api.upersona.app
```

When developing locally against the Spring Boot dev server, point it to `https://pb.local/api` (or whatever domain your HTTPS proxy exposes).

---

## Sample files

- `personal-branding-saas-api/.env.example` — copy to `.env` (or export the values manually) before running Spring Boot locally.
- `personal-branding-saas-web/.env.example` — copy to `.env.local` for Next.js.

Keep the `.example` files up to date whenever the code introduces new configuration knobs.
