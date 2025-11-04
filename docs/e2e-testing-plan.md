# E2E Testing Plan

1. Add Playwright as a dev dependency (
pm install -D @playwright/test).
2. Generate a baseline config via 
px playwright install and create 	ests/e2e with login, refresh, and onboarding specs mirroring the REST contracts.
3. Wire an npm script (e.g., 	est:e2e) and configure CI to run the suite against a seeded backend.
4. Capture artifacts (traces, screenshots) on failure to simplify debugging.
5. Gate PRs on the suite in a separate branch before merging into main.
