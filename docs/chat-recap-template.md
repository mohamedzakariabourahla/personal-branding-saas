# Chat Recap Template

Paste this at the start of a new chat to restore context. Include pointers to other docs when relevant.

- **Goal**: what you're building + current milestone (e.g., local-first dev; TikTok paused; tighten auth/session).
- **Stack**: Spring Boot API + Next.js frontend; `SPRING_PROFILES_ACTIVE=local`; mail via Resend HTTP; Redis + Postgres.
- **Env/URLs**: local `https://pb.local` via mkcert/local-ssl-proxy; API base; key env vars set.
- **Recent changes**: secrets moved to env, mail handler hardened, Redis limiter fails closed, prod profile added.
- **Next step**: what you want help with right now (optional if goal already makes it clear).
- **Docs**: mention if you're referencing `environment.md` (env/setup), `phases-and-tasks.md` (roadmap/checklist), `connectors.md` (provider setup), `security.md` (session/CSP/encryption), `product-overview.md` (product context).

Example:
Goal: let's refine the the code, read and review both frontend and backend and make it strictly respecting clean architecture and solid principles and ready for scalability and each part of code should be in it is place.
Stack: Spring Boot + Next.js; Redis/Postgres.
State: schedule publishing integrated andtested with instagram, TikTok sandbox DNS blocked; YouTube untested, the rest platforms not integrated yet.
Env/URLs: https://pb.local (mkcert+proxy), API localhost:8443 (mkcert+proxy).
Recent: schedule posts integrated and tested with instagram.
Next: review and read all the frontend and give me advices in the clean architecture implementation, scalabilty, respecting each layer of codes as example (the services should be in hooks not in components or the records/models should be in models not in components).
Docs: see environment.md (envs/setup), phases-and-tasks.md (roadmap), connectors.md (Meta/TikTok/YouTube), security.md (sessions/CSP/encryption), product-overview.md (product).
