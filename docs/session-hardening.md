# Session Hardening Roadmap

- Migrate browser refresh tokens to HttpOnly cookies (pb_refresh_token) scoped to /api/auth, paired with a double-submit header (e.g. X-CSRF-Token) so SPA requests are distinguishable from CSRF attempts.
- Restore the in-memory session provider to keep access tokens out of localStorage; refresh tokens remain server-side while access tokens live only in memory.
- Extend the security config to enforce the anti-CSRF header and continue rotating refresh tokens on every use.
- Update the frontend HTTP client to send the anti-CSRF header and drop local storage persistence once cookies are enabled.
- Add regression tests (Playwright/Cypress) covering login, refresh, onboarding, and logout under the cookie-based flow.
- Document rollout steps (staging ? production) and wire dashboards to monitor refresh failures in real time.
