# Vercel Deployment Preparation

Status: planning only. Do not deploy this repository to production until the blockers below are resolved.

## Target

Create a standalone Vercel project named `ai-goofish-monitor` linked to this repository. Keep the existing `jev-chat` project and its environment variables untouched.

## Current compatibility blockers

- The application starts an APScheduler and process manager in the FastAPI lifespan. Vercel request-driven instances are not a durable home for an always-running scheduler or crawler subprocesses.
- The application uses SQLite at `data/app.sqlite3` and writes account login state, prompts, logs, and images to local directories. Vercel compute filesystems must not be treated as durable storage.
- The Vue app's Vite development proxy targets `127.0.0.1:8000`; production must route API requests to the deployed backend.
- The app includes WebSocket routes; production behavior and reconnection must be validated against the selected Vercel runtime.
- Playwright requires Chromium and its system dependencies. The deployment image/runtime and bundle limits must be validated before enabling crawling.

## Proposed Vercel-compatible architecture

1. **Web UI** — build the Vue app from `web-ui/`; use same-origin routing or a configured backend URL rather than the Vite development proxy.
2. **FastAPI API** — request-driven endpoints; remove startup assumptions that require a permanent process or local durable files.
3. **Worker** — move crawl execution out of API request processes. Use a Vercel-supported background worker/queue design only after confirming feature availability for this team. The worker must be idempotent and safely restartable.
4. **Postgres from Vercel Marketplace** — migrate task, result, price-history, and job state from SQLite. Preserve migration/import support and add database integration tests before switching production reads/writes.
5. **Private object storage** — use Vercel Blob or a Marketplace object store for sensitive account-state files and durable assets. Never use public storage for Xianyu cookies/session state. Limit access to server-side credentials.
6. **Secrets and access control** — configure AI provider credentials and non-default web credentials through Vercel environment variables. Do not commit `.env`, cookies, API keys, or database credentials.

## Deployment gates

- [ ] Vercel project creation permission is available for the target team.
- [ ] Confirm which Vercel backend/daemon features are enabled on the team's plan before choosing runtime configuration.
- [ ] Provision and connect Postgres and private object storage to the new project.
- [ ] Add tests for database persistence, session-file read/write, worker restart/idempotency, auth, and API routing.
- [ ] Run `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 pytest`.
- [ ] Run `cd web-ui && npm ci && npm run build`.
- [ ] Deploy a preview only after the above checks pass; test login, task CRUD, crawl execution, result persistence, images, logs, and worker restart behavior.
- [ ] Promote to production only after preview acceptance.

## Current state

A preparation branch is being used so the default `master` branch is not changed. No Vercel project or storage resource has been created yet. No production deployment has been made.
