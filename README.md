# Personal Branding SaaS

This repository contains:

- `personal-branding-saas-api`: Spring Boot backend that manages authentication, platform OAuth flows (Meta/TikTok/etc.), and encrypted credential storage.
- `personal-branding-saas-web`: Next.js + MUI front-end dashboard for creators.

## Local Setup

1. Install dependencies in both subprojects (`npm install` / `mvn clean install`).
2. Run the infrastructure dependencies via Docker (Postgres, Redis, Kafka) as described in `docs/hardening-plan.md`.
3. Start the backend (`mvn spring-boot:run -pl personal-branding-saas-api`).
4. Start the frontend (`npm run dev` inside `personal-branding-saas-web`).

Environment variables for both apps are documented in [`docs/environment.md`](docs/environment.md). Copy the provided `.env.example` files in each package and populate them before running locally or wiring CI/CD.

## Meta / Instagram Connector

The Meta connector requires a dedicated Instagram Graph API app and specific Business Manager configuration (linking professional IG accounts to pages, tester roles, redirect URIs, etc.). Follow the checklist in **docs/meta-connector-setup.md** before attempting the OAuth flow locally.

## YouTube Connector

Before using the new YouTube integration, run through **docs/youtube-connector-setup.md** to provision a Google Cloud project, enable the YouTube Data API, and register the localhost redirect URI. Once those env vars are set, the `/publishing` screen can start the Google consent flow just like TikTok/Meta.

Additional providers (TikTok, YouTube, Threads) will follow the same shared OAuth DTO introduced in the current codebase, so refer to Meta’s implementation when wiring new connectors.
