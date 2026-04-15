# AI Blog Automation System

Production-ready AI blog automation backend built with FastAPI, PostgreSQL, Celery, Redis, Docker, and OpenAI.

## Project Overview

This service generates SEO-optimized blog posts from keywords, creates contextual images, publishes to one or more websites, schedules recurring runs, and prevents duplicate keyword or content usage. It is implemented as a standalone Python service so it can integrate with WordPress or generic publishing APIs without interfering with existing WordPress plugin code.

## Features

- Keyword-based long-form blog generation with title, slug, meta title, meta description, structured headings, and FAQs
- SEO enhancement pipeline with keyword extraction, heading validation, readability scoring, and internal-link placeholders
- Featured and inline image generation with SEO alt text
- Multi-website publishing for WordPress REST API and generic external APIs
- Celery task queue with background execution and Celery Beat scheduling
- Duplicate prevention using normalized keyword tracking, SHA256 content hashes, and optional embedding similarity checks
- Configurable post status, word count, image count, and featured image resolution
- Dockerized local deployment with PostgreSQL and Redis
- API endpoints for keywords, websites, automation rules, manual triggers, and logs
- Refactored service-based architecture with repositories, application workflows, and pluggable publishers

## Project Structure

```text
ai-blog-automation-system/
├── app/
│   ├── api/
│   ├── application/
│   ├── repositories/
│   ├── ai_engine/
│   ├── image_engine/
│   ├── publisher/
│   ├── scheduler/
│   ├── seo_engine/
│   ├── database/
│   └── utils/
├── config/
├── docker/
├── scripts/
├── .env.example
├── docker-compose.yml
├── README.md
└── requirements.txt
```

## Installation

1. Create and activate a Python 3.12 virtual environment.
2. Install dependencies with `pip install -r requirements.txt`.
3. Copy `.env.example` to `.env` and set `OPENAI_API_KEY`.
4. Start PostgreSQL and Redis, or use Docker Compose.
5. Run the API with `python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload`.
6. Start Celery worker with `celery -A app.celery_app.celery_app worker --loglevel=info`.
7. Start Celery Beat with `celery -A app.celery_app.celery_app beat --loglevel=info`.

## Refactor Summary

- API routes are thin and delegate to application services.
- Repositories isolate database access from orchestration logic.
- The automation workflow owns end-to-end business flow for content generation, media handling, duplicate prevention, and publishing.
- Publishers are swappable adapters for WordPress and generic APIs.
- Scheduler dispatch now uses a `queued` keyword state to reduce duplicate job creation.
- Database indexes were added to improve lookup performance on common operational queries.

## Docker

Run `docker compose up --build`.

## Core API Endpoints

- `POST /api/v1/keywords`
- `GET /api/v1/keywords`
- `POST /api/v1/websites`
- `GET /api/v1/websites`
- `PUT /api/v1/websites/{website_id}`
- `POST /api/v1/automation/rules`
- `POST /api/v1/automation/run`
- `GET /api/v1/logs/publication`
- `GET /api/v1/logs/automation`

## Scheduling Setup

- Create pending keywords.
- Create one or more automation rules.
- Run Celery Beat.
- The scheduler scans rules and queues matching jobs for pending keywords.

## Deployment Notes

- Replace the default `SECRET_KEY`.
- Use managed PostgreSQL and Redis in production.
- Disable `AUTO_CREATE_TABLES` if you switch to migrations.
- Put the service behind a reverse proxy.

## Production SaaS Improvements

- Add multi-tenant support with `tenant_id` on keywords, websites, rules, posts, assets, and logs.
- Replace direct secret storage with encrypted credentials using a KMS or Vault-backed secret manager.
- Add proper Alembic migrations and stop relying on runtime table creation in production.
- Move generated assets to object storage like S3 or Cloudflare R2 instead of local disk.
- Add provider abstraction for LLM and image vendors so you can fail over between providers.
- Introduce usage metering for tokens, images, tasks, and published posts per tenant.
- Add rate limits, API keys, RBAC, and audit logs for account-level security.
- Add observability with structured logs, tracing, metrics, dead-letter queues, and alerting.
- Use webhooks and callback events so customers can react to publish success or failure.
- Add a similarity cache or vector database if embedding-based duplication grows large.
- Introduce tenant-aware scheduling fairness so one customer cannot starve the worker pool.
- Add resumable workflows and idempotency keys for retries across API, queue, and publisher boundaries.

## Optional Next Step

The backend is ready for a React dashboard or a WordPress bridge plugin if you want a UI layer next.
