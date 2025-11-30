# Environment & Local Setup

This project has two apps:

- `personal-branding-saas-api` (Spring Boot)
- `personal-branding-saas-web` (Next.js)

All secrets/config come from environment variables. Keep `.env` files out of git; use placeholders only in `.env.example`.

## Backend env vars (API)

Required core:

- `SPRING_PROFILES_ACTIVE` — `local` on dev laptops, `prod` in deploys.
- `SPRING_DATASOURCE_URL` / `SPRING_DATASOURCE_USERNAME` / `SPRING_DATASOURCE_PASSWORD` — Postgres.
- `REDIS_HOST` / `REDIS_PORT` / `REDIS_PASSWORD` — Redis.
- `SECURITY_JWT_SECRET`, `SECURITY_JWT_ISSUER`.
- `APP_ENCRYPTION_KEY` (32-byte AES-GCM), `APP_ENCRYPTION_KEY_ID`.
- `MAIL_PROVIDER_API_KEY`, `MAIL_PROVIDER_BASE_URL` (Resend default), `MAIL_FROM`.
- `APP_MAIL_RESET_BASE_URL`, `APP_MAIL_VERIFY_BASE_URL` — URLs embedded in auth emails.
- `APP_CORS_ALLOWED_ORIGINS`, `APP_SECURITY_CSP`, `APP_SECURITY_FRAME_ANCESTORS`.

OAuth providers:

- Meta: `META_CLIENT_ID`, `META_CLIENT_SECRET`, `META_REDIRECT_URI`, `META_SCOPES`, `META_AUTH_BASE_URL`, `META_GRAPH_BASE_URL`.
- TikTok: `TIKTOK_CLIENT_KEY`, `TIKTOK_CLIENT_SECRET`, `TIKTOK_REDIRECT_URI`, `TIKTOK_SCOPES`, `TIKTOK_AUTH_BASE_URL`, `TIKTOK_API_BASE_URL`.
- YouTube: `YOUTUBE_CLIENT_ID`, `YOUTUBE_CLIENT_SECRET`, `YOUTUBE_REDIRECT_URI`, `YOUTUBE_SCOPES`, `YOUTUBE_AUTH_BASE_URL`, `YOUTUBE_TOKEN_URL`, `YOUTUBE_API_BASE_URL`.

Optional tuning:

- `SECURITY_LOGIN_*`, `SECURITY_REFRESH_*` (rate limits), `SECURITY_JWT_ACCESS_TTL`, `SECURITY_JWT_REFRESH_TTL`.
- `SERVER_PORT`, `SERVER_ADDRESS`.

## Frontend env vars (Web)

- `NEXT_PUBLIC_API_URL` — API base for Axios. For local `http://pb.local:8080/api` (backend on 8080, cookie domain `pb.local`); for Vercel set to deployed API origin.

## Asset storage

- `APP_ASSETS_DIR` — where uploads are stored (default `./tmp-home/uploads` locally).
- `APP_ASSETS_BASE_URL` — optional public base URL (e.g., CDN/object storage). If empty, URLs are built from request origin at `/uploads/{filename}`.

## Local dev setup (pb.local with mkcert)

1. Map host: add `127.0.0.1 pb.local` to your hosts file.
2. Certs: in `personal-branding-saas-web`, generate with mkcert:
   ```
   mkcert -cert-file certs/pb.local.pem -key-file certs/pb.local-key.pem pb.local
   ```
3. Frontend: `npm install` then `npm run dev`.
4. HTTPS proxy:
   ```
   npx local-ssl-proxy --source 443 --target 3000 --cert certs/pb.local.pem --key certs/pb.local-key.pem
   npm run dev -- --hostname 0.0.0.0 --port 3000
   npx local-ssl-proxy --source 443 --target 3000 --cert certs/pb.test.pem --key certs/pb.test-key.pem --hostname 0.0.0.0
   npx local-ssl-proxy --source 8443 --target 8080 --cert certs/pb.test.pem --key certs/pb.test-key.pem --hostname 0.0.0.0
   .\coredns.exe -conf Corefile
   ```
   Visit `https://pb.local`.
5. Backend: `mvn spring-boot:run` with `SPRING_PROFILES_ACTIVE=local` and your `.env` loaded. For cookie-only auth locally, keep `APP_CORS_ALLOWED_ORIGINS` including `https://pb.local` (and `http://pb.local:3000` if you skip the SSL proxy) and set `SECURITY_REFRESH_COOKIE_DOMAIN=pb.local`. Redirect URIs should use `https://pb.local/...`.
6. Infra: run Postgres/Redis locally (docker compose in `infra/` if available) or point to managed instances.

## Deployment notes

- Set `SPRING_PROFILES_ACTIVE=prod` in Render (or target platform); supply real secrets (no placeholders).
- Frontend on Vercel: set `NEXT_PUBLIC_API_URL` to your deployed API.
- Mail: `MAIL_FROM` must be verified with provider; auth URLs must be reachable (no localhost).
- Tests: the `test` profile uses in-memory H2 with dummy mail/Redis; no external services required.
