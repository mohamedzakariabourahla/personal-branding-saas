# Publishing Scheduling & Worker Skeleton

Current state (code):
- Domain contracts: `PublishingJob`, `PublishingJobStatus`, `PublishingJobRepository`, `PublishingJobQueue`, `PublishingAttempt`, `PublishingAttemptRepository`.
- Application service: `PublishingJobService` (`schedule`, `claimDue`, `markCompleted`, `markFailed` with optional retry/dead-letter, `recordAttempt`, `retryNow`).
- Defaults: Postgres + JPA repository by default; Redis delayed queue for non-test profiles; in-memory repo/queue only in the `test` profile.
- Worker: `PublishingWorker` (feature-flagged via `app.scheduling.worker.enabled=true`, interval `app.scheduling.worker.interval`) polls due jobs, resolves platform + connection + auth, calls platform publishers, records attempts, retries with backoff, and dead-letters after `app.scheduling.worker.max-attempts`. Metrics counters emitted.

How to use locally:
1) Inject/Autowire `PublishingJobService`.
2) Call `schedule(...)` with user/platform/connection, media asset ids, caption, and an optional scheduled time.
3) Periodically call `claimDue(max)` to pull due jobs (status flips to `IN_PROGRESS`), then process and call `markCompleted(jobId)` or `markFailed(jobId, reason, retryDelay)`. You can enable the built-in worker for local testing by setting `app.scheduling.worker.enabled=true`.
4) Attempts API: `/publishing/jobs/{id}/attempts` returns recent attempts; `/publishing/jobs/{id}/retry` re-queues immediately. Frontend publishing screen shows jobs, attempts, and retry button.

Next steps (to productionize):
- Wire the worker to load platform/connection + payload and call `PlatformPublisherRegistry`, capturing provider responses.
- Expose health/metrics for queue depth, lag, success/fail counts; add UI refinements and manual retry/cancel controls.
- Add Kafka handoff later for horizontal workers (`publish.request`/`publish.result`) after the Redis baseline is stable.
- Instagram publishing notes:
  - Scopes used by default: `instagram_basic, instagram_content_publish, pages_show_list, pages_manage_posts, business_management, pages_read_engagement, instagram_manage_comments, instagram_manage_insights, instagram_manage_messages, instagram_branded_content_brand, instagram_branded_content_creator`.
  - Connection metadata must include `instagramBusinessAccountId`; worker uses the first `mediaAssetIds` entry as an image URL when publishing via Graph `/media` + `/media_publish`.
