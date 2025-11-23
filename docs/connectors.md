# Connector Setup (Meta, TikTok, YouTube)

## Meta / Instagram
Use case: **Manage messaging and content on Instagram** with Facebook login.
Steps:
1) Create a Business app in developers.facebook.com.
2) Enable the Instagram API use case; add perms `business_management, instagram_basic, instagram_content_publish, pages_manage_posts, pages_read_engagement, pages_show_list, instagram_manage_comments, instagram_manage_insights, instagram_manage_messages, instagram_branded_content_brand, instagram_branded_content_creator`.
3) Facebook Login setup: authorize redirect = `https://pb.local/meta/callback` for local (`<host>/meta/callback` in other environments); configure deauth/data deletion if prompted.
4) Link assets: convert IG to Professional; link IG to a Facebook Page under the same Business Manager; verify in Business Settings → Instagram accounts → Connected assets.
5) Testers: App roles → Instagram Testers; invite your IG handle; accept in the Instagram app (Account Center → Apps and websites → Tester invites).
6) Env vars: set `META_CLIENT_ID`, `META_CLIENT_SECRET`, `META_REDIRECT_URI`, `META_SCOPES`, `META_AUTH_BASE_URL`, `META_GRAPH_BASE_URL`.

## TikTok
Current status: sandbox hosts unreachable on some networks; plan for both flows:
- Sandbox: `https://www-sandbox.tiktok.com` / `https://open-api-sandbox.tiktok.com` with sandbox client key/secret; requires DNS that can resolve those hosts.
- Prod (pre-approval testing): `https://www.tiktok.com` / `https://open.tiktokapis.com` with prod host + sandbox key may fail; once prod creds are available, add your account as a tester in Sandbox → Target Users and use prod endpoints.
Common setup:
1) Add redirect URI `https://pb.local/tiktok/callback` in TikTok Login Kit for local (swap `<host>` per environment).
2) Add tester accounts under Sandbox → Target Users; accept the invite in the TikTok app.
3) Env vars: `TIKTOK_CLIENT_KEY`, `TIKTOK_CLIENT_SECRET`, `TIKTOK_REDIRECT_URI`, `TIKTOK_SCOPES`, `TIKTOK_AUTH_BASE_URL`, `TIKTOK_API_BASE_URL`.

## YouTube
Requires a Google Cloud project with YouTube Data API enabled (currently no dev account available to test).
Steps:
1) In console.cloud.google.com, create a project; enable **YouTube Data API v3** (and optional People API).
2) Create OAuth client (Web):
   - Authorized JS origins: `https://pb.local`
   - Authorized redirect URIs: `https://pb.local/youtube/callback`
3) Set env vars: `YOUTUBE_CLIENT_ID`, `YOUTUBE_CLIENT_SECRET`, `YOUTUBE_REDIRECT_URI`, `YOUTUBE_SCOPES`, `YOUTUBE_AUTH_BASE_URL`, `YOUTUBE_TOKEN_URL`, `YOUTUBE_API_BASE_URL`.
4) Keep consent screen in Testing; add your Google account as a test user.
Notes:
- We request offline access + PKCE; Google returns refresh tokens once per client/user unless revoked.
- Channel metadata is stored in `PlatformConnection.accountMetadata` for debugging/analytics.
