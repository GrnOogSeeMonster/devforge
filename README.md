# DevForge

A self-hosted browser development environment — projects, per-project Docker containers,
a Monaco editor and an AI assistant backed by a local Ollama model. The Replit shape,
running entirely on your own hardware with no code leaving the box.

---

## Status

**Working prototype. Auth, projects and container lifecycle are real; the editor is a
shell and collaboration was never started.** Last worked on 10 August 2025.

| | |
|---|---|
| Size | ~8,400 lines TypeScript across backend, frontend and four microservices |
| Working | Registration/login/refresh/logout, project CRUD, container start/stop/logs/stats, AI chat against Ollama |
| Partial | Editor page renders Monaco but is not wired to project files; console panel is a placeholder |
| Not started | Real-time collaboration, terminal, database provisioning UI, deployment pipeline |
| Tests | 4 files (1 backend, 3 frontend) — smoke level, not coverage |
| Verified | Individual services run; the full 12-container compose stack has not been confirmed healthy end to end |

Read `DEVELOPMENT_CHECKLIST.md` for the phase-by-phase state at the point work stopped:
Phases 1 and 2 are largely done, Phases 3 and 4 are open.

---

## What is built

**Backend** (`backend/src`, Node + Express + TypeScript)

| Area | Detail |
|---|---|
| `routes/authRoutes.ts` (567 lines) | Register, login, refresh, logout, password reset, email verification. Refresh tokens stored in Redis and rotated. |
| `routes/projectRoutes.ts` (258 lines) | Project CRUD, plus container create / start / stop / delete / logs / stats, proxied to the workspace-manager service. |
| `routes/aiRoutes.ts` (119 lines) | Conversation CRUD and message send, forwarded to Ollama. Model listing. |
| `middleware/authMiddleware.ts` (391 lines) | JWT verification, role checks, request context. |
| `services/RedisService.ts` (384 lines) | Caching, sessions, refresh-token store. |
| `services/DatabaseService.ts` (253 lines) | PostgreSQL pool and query layer. |

Commented-out route registrations in `index.ts` mark the endpoints that were planned
and not written: `users`, `files`, `containers`, `databases`, `deployments`.

**Frontend** (`frontend/src`, React 18 + TypeScript + Mantine)

Auth flow (login, register, forgot, reset), an app shell with navigation, a dashboard
with activity feed and quick stats, project list / detail / create, and an editor page.
Routes: `/dashboard`, `/projects`, `/projects/new`, `/projects/:id`, `/editor`,
`/editor/:projectId`, `/databases`.

**Microservices** (`services/`)

| Service | Lines | State |
|---|---|---|
| `workspace-manager` | 115 | Working — Docker SDK container lifecycle |
| `ai-assistant` | 57 | Working — thin Ollama proxy |
| `realtime-server` | 18 | Stub |
| `database-provisioner` | 10 | Stub |

**Infrastructure** — a 12-service Docker Compose stack: postgres, redis, ollama, backend,
frontend, the four microservices, prometheus, grafana, nginx and mailhog. Backup and
restore scripts, secret generation on first setup, self-signed certs.

---

## Stack

React 18 · TypeScript · Mantine UI · Monaco · Vite · Node · Express · Socket.IO ·
PostgreSQL · Redis · Ollama · Docker · Prometheus · Grafana · nginx

---

## Running it

```bash
cp .env.example .env
./scripts/setup.sh        # generates JWT, session and encryption secrets
docker compose up --build
```

Frontend on `:3000`, API on `:8000`, Grafana on `:3001`, Mailhog on `:8025`.

`INSTALL.md` and `SETUP.md` have the longer path; `docs/API.md` documents the endpoints
that exist.

---

## Notes

The repo contains `storage/secrets.json` and a self-signed keypair under `ssl/`.
Both are local development artifacts — regenerate them before this goes anywhere real.

Unrelated to the `NexusCore` repo alongside this one, which also used the name DevForge
for a while.

MIT licensed.
