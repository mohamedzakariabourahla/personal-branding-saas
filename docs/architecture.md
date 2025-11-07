# Architecture Guidelines

This document captures how the personal-branding SaaS codebase is structured so new features (content templates, scheduling, analytics, etc.) can scale without entangling infrastructure details.

## Layered Design

| Layer | Responsibilities | Allowed Dependencies |
| --- | --- | --- |
| **Domain** (`saas.personal_branding.api.domain.*`) | Entities, value objects, domain events, repository/service interfaces. No framework or infrastructure code. | None |
| **Application** (`saas.personal_branding.api.application.*`) | Use-case services orchestrating domain interfaces (e.g., `GenerateContentService`). Contains input/output DTOs and transaction boundaries. | Domain |
| **Infrastructure** (`saas.personal_branding.api.infrastructure.*`) | Concrete adapters (JPA repositories, Redis rate limiters, Kafka publishers/consumers, external API clients). | Domain, Application (only to implement interfaces) |
| **Presentation** (`saas.personal_branding.api.presentation.*`) | HTTP controllers, GraphQL resolvers, scheduled endpoints. Maps transport models to application commands. | Application |

Rules:
- Domain must remain tool-agnostic (no Spring annotations, no Redis/Kafka/Postgres imports).
- Application depends only on domain. Inject interfaces; never instantiate infrastructure classes directly.
- Infrastructure beans are wired in Spring `@Configuration` classes, which bind concrete adapters to domain/application interfaces.
- Tests use the smallest necessary slice: pure unit tests at domain/application, adapter tests with Testcontainers/Fake services inside infrastructure, and limited end-to-end tests that go through presentation.

## Infrastructure Adapters

| Concern | Adapter Package | Notes |
| --- | --- | --- |
| Persistence (Postgres) | `infrastructure.persistence.*` | JPA entities + repository adapters. Any schema change must flow through Flyway. |
| Cache / Distributed Locks (Redis) | `infrastructure.cache.*` | Encapsulate RedisTemplate usage; expose domain interfaces like `LoginRateLimiter`. Keep serialization isolated. |
| Messaging (Kafka) | `infrastructure.messaging.*` | Producers/consumers live here. Topics defined via configuration classes. Application layer sees only interfaces such as `ContentEventPublisher`. |
| External APIs (Meta/TikTok/YouTube/Google) | `infrastructure.provider.*` | HTTP clients + auth token handling. Secrets injected via configuration. |
| Background Workers | Separate module or `infrastructure.worker.*` | Contains consumers / schedulers triggered by Kafka or Redis streams. Keeps long-running jobs out of web tier. |

## Flyway & Data Strategy

1. **Versioned migrations** (`V__`) capture structural changes in chronological order. Never skip numbers; renumbering is only acceptable when everyone can drop their schema (dev phase).
2. **Repeatable migrations** (`R__`) hold reference/seed data that may evolve (tones, templates). They run after every successful migration.
3. When a new slice lands:
   - Update ERD diagram (Figma or dbdiagram.io).
   - Add the structural migration (e.g., `V6__template_catalog.sql`).
   - Extend/adjust repeatable seeds if necessary.
   - Provide rollback guidance if the change affects production.

## Worker / Async Model

- HTTP controllers enqueue work via application services that publish commands to Kafka (`GenerateContentCommand`, `SchedulePublishCommand`).
- A dedicated worker process (shared codebase or separate module) hosts Kafka consumers. Consumers:
  1. Deserialize message → domain command.
  2. Execute application service (which may use Redis locks, Postgres repositories, external APIs).
  3. Emit status events for analytics/monitoring.
- Scheduling can start with Redis-delayed queues (sorted sets) but the API must hide the implementation so swapping to Quartz or Temporal later is trivial.

## Configuration & Profiles

- `application.properties` defines placeholders; `application-local.properties` and `application-test.properties` give concrete values for local dev and tests.
- Each infrastructure adapter should expose a minimal configuration bean (e.g., `KafkaProducerConfig`) that reads from Spring configuration properties (`spring.kafka.*`, `app.redis.*`).
- Feature flags / phased rollout belong under `app.features.*` to keep toggles discoverable.
- Sensitive values (OAuth tokens, API keys) must be encrypted before persistence using the `app.encryption.*` settings (AES-GCM master key provided via `APP_ENCRYPTION_KEY`). Store only ciphertext + IV + fingerprint in Postgres.

## Testing Strategy

1. **Domain/Application**: Plain JUnit tests, no Spring context.
2. **Infrastructure adapters**: Use Testcontainers (Postgres, Redis, Kafka) or embedded fakes. Verify serialization, indexes, and failure modes.
3. **Contract tests**: For messaging topics and external provider clients, define consumer-producer contracts in `infrastructure/messaging`.
4. **End-to-end**: Limited Spring Boot tests that spin up the real context with `local` profile overrides. Ensure Flyway runs against ephemeral Postgres.

Following this structure keeps Redis/Kafka/Postgres concerns encapsulated, allowing us to grow to 100k+ users, replace tools, or add new workers without rewriting business logic.
