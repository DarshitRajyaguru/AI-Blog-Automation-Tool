# AI Blog Automation System Architecture

## Layered Design

- `app/api`: HTTP interface layer for FastAPI routes and request validation.
- `app/application`: orchestration layer for use cases, dispatching, duplicate prevention, publication flow, and content assembly.
- `app/repositories`: persistence layer wrapping SQLAlchemy access for keywords, websites, rules, jobs, posts, and logs.
- `app/ai_engine`, `app/image_engine`, `app/seo_engine`, `app/publisher`: infrastructure adapters for external systems and specialized engines.
- `app/database`: ORM models and session management.
- `app/utils`: pure helper functions without framework coupling.

## Core Flow

1. FastAPI receives keywords, websites, automation rules, or manual run requests.
2. The automation dispatcher validates the keyword state and marks it as `queued`.
3. Celery processes the queued job using the application workflow.
4. The workflow validates duplicate keyword usage, content hashes, and optional embedding similarity.
5. OpenAI generates structured content and images.
6. The content assembly service injects inline image references into markdown and regenerates HTML.
7. The publication service pushes the content to one or more publishers with retry tracking.
8. PostgreSQL stores the post, generated media, automation jobs, and publication logs.
9. Celery Beat scans active rules and enqueues pending keywords on schedule.

## Scalability Decisions

- Repository pattern keeps SQL isolated from HTTP routes and worker code.
- Workflow orchestration keeps complex business logic in one reusable application layer.
- Keyword `queued` state reduces duplicate scheduling under load.
- Added database indexes on high-traffic workflow fields like status, priority, lifecycle, and publication status.
- Publishers remain pluggable, so new CMS targets can be added without touching workflow logic.
