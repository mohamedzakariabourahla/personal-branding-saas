# YouTube Connector Setup (Local + Sandbox)

These steps mirror the Meta/TikTok onboarding guide so every contributor can run the YouTube OAuth flow locally with their own Google Cloud project.

## 1. Google Cloud project & APIs

1. Visit [console.cloud.google.com](https://console.cloud.google.com) and create a new project (or reuse an existing sandbox project dedicated to Personal Branding SaaS).
2. Enable the **YouTube Data API v3** for that project.
3. (Optional but recommended) enable the **People API** so you can extend the connector later with profile metadata.

## 2. OAuth client

1. Go to **APIs & Services → Credentials → Create Credentials → OAuth client ID**.
2. App type: **Web application**.
3. Authorized JavaScript origins:
   - `http://localhost:3000`
4. Authorized redirect URIs:
   - `http://localhost:3000/youtube/callback`
5. Save the generated **Client ID** and **Client secret** – these map to `YOUTUBE_CLIENT_ID` / `YOUTUBE_CLIENT_SECRET`.

## 3. Local environment variables

In `personal-branding-saas-api`, set (via `.env`, shell exports, or `infra/env.txt`):

```
YOUTUBE_CLIENT_ID=xxx.apps.googleusercontent.com
YOUTUBE_CLIENT_SECRET=xxxx
YOUTUBE_REDIRECT_URI=http://localhost:3000/youtube/callback
YOUTUBE_SCOPES=https://www.googleapis.com/auth/youtube.readonly
```

If you need to hit staging/preview backends, override the redirect URI per environment.

## 4. Test accounts

During early development leave the OAuth consent screen in **Testing** mode and add your personal Google accounts as test users. Once the app is verified you can switch to Production and remove the test-user restriction.

## 5. Running the flow locally

1. Start infra (`docker compose -f infra/docker-compose.yml up -d`).
2. Run the backend with the `local` profile so the YouTube properties above are picked up.
3. `npm run dev` in `personal-branding-saas-web` and sign in as any seeded user.
4. Head to `/publishing`, choose the YouTube card, and complete the OAuth prompts. If the Google account owns multiple channels you’ll get the same selection UI used for Meta.

## 6. Notes & troubleshooting

- We request `offline` access + PKCE, so Google returns refresh tokens. You must revoke the connector or delete the connection row if you want to trigger another consent screen (Google only returns the refresh token the first time per client + user).
- The backend stores channel metadata (title, handle, thumbnail) inside the `PlatformConnection.accountMetadata` JSON column — useful for debugging and future analytics.
- Rotating credentials: update the environment variables and restart the Spring Boot API; the Next.js front-end pulls only the redirect URL from the backend.

Refer to `docs/hardening-plan.md` for the broader roadmap around publishing + analytics once the connector is live.
