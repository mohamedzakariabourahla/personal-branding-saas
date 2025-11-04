# Security Headers

- Content-Security-Policy now defaults to disallow inline scripts/styles. Any inline snippets must use nonce/hash and update pp.security.csp explicitly.
- X-Frame-Options is DENY unless pp.security.frame-ancestors is configured.
- X-Content-Type-Options=nosniff, Referrer-Policy=no-referrer, X-XSS-Protection=0, and a restrictive Permissions-Policy are set on every response.
- Add third-party hosts (fonts, analytics, APIs) to the CSP via pp.security.csp; avoid reintroducing 'unsafe-inline' unless absolutely required.
