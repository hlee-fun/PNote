# PNote

PNote is a self-hosted personal content site: write articles in Markdown while showcasing your own projects,
letting visitors browse the source code online, download releases, and letting the admin `git clone` / `git push` directly.

The frontend and backend are packaged into a single Docker image with PostgreSQL and Redis built in,
so there is no need to install Python, Node, or any database on the host machine.

Demo site: https://note.hlee.fun (Note: the demo site may be running an outdated version!)

> **Note**: Known issues are being fixed on an urgent basis; no usable version has been released yet.

## Features

### Writing articles

- Markdown editing: tables, task lists, footnotes, code highlighting (Pygments), KaTeX math, Mermaid diagrams, auto TOC
- Drafts and scheduled publishing, server-side autosave; every edit is snapshotted and can be rolled back at any time
- Tags, multi-level categories (self-referencing parent/child), pinning, view counts, and anonymous likes
- Batch import/export (Markdown zip with frontmatter)

### Showcasing projects

- Project card list with language filtering and keyword search
- README rendering, online source code browsing and downloading
- Publish releases and package the source directory as a zip for download
- One cloud Git bare repo per project, supporting `git clone` / `git push` (smart HTTP, admin only, HTTPS enforced)
- One-way sync to GitHub / GitLab / Gitea / Gitee / Bitbucket / self-hosted, with dry-run and mirror push
  (Only GitHub has been tested so far; the other providers are still being tested)

### Site

- Admin console: dashboard (PV/UV, posting heatmap, trends), file management, backups, audit logs, categories, settings
- Interface supports Simplified Chinese, Traditional Chinese, and English; the admin panel controls which locales are enabled and the default one
- Site-wide search (PostgreSQL full-text search + ILIKE; Chinese falls back to ILIKE)
- RSS 2.0 / Atom feeds, sitemap.xml, and robots.txt
- Site switches: close the whole site (with a notice), private site (only the about page is reachable), announcement banner
- Legal pages: terms, privacy, and cookie notice, with editable body text

### Security

- Two-factor authentication (TOTP) with one-time recovery codes
- Sensitive operations (password change, backup download, disabling 2FA, sync push) require re-authentication
- API tokens (read / write / admin, with optional expiry), sent via the `X-API-Key` header
- Session management: list active sessions, revoke per device, or sign out everywhere
- Audit logs, CSP, HSTS, hotlink-protection allowlist, upload allowlist with magic-bytes validation

### AI assistance (optional)

- Generate summaries, titles, continuations, and Chinese→English translations (results are shown for copying only and never overwrite the body)
- Supports both the Anthropic and OpenAI protocols; the API key is stored encrypted
- Fill in the API key in the admin panel; other features work fine without it
- AI assistance performance depends on the performance of the connected model

## Installation

Docker Compose is the only deployment method: `docker compose`

Prerequisites: Docker (with the `docker compose` plugin); Linux / macOS / Windows (WSL2) all work.

`SECRET_KEY`, `INIT_PASSWORD`, `CORS_ORIGINS`, and `PUBLIC_BASE_URL` are required — startup fails if any is missing.
Just fill them in under `environment` in `docker-compose.yml`; see "Configuration" for what each one means.

```bash
# Edit the corresponding parameters in docker-compose.yml
docker compose up -d
```

The container startup script brings up PostgreSQL, Redis, and the backend service in order, and creates the initial admin account.

### Access URLs

The frontend and backend share the same origin, all through port 8000:

| Entry | URL |
| --- | --- |
| Home | http://localhost:8000 |
| Admin | http://localhost:8000/admin |
| Health check | http://localhost:8000/api/health |

The initial username comes from `INIT_USERNAME` (default `admin`); the password is set via the required `INIT_PASSWORD`
— **the image ships no default password**. If a weak password is used, you will be forced to change it after the first login.

### Common Commands

```bash
docker compose logs -f pnote   # Live logs
docker compose down            # Stop and remove the container, data volumes are kept
docker compose up -d --build   # Rebuild after updating the code
```

## Configuration

Edit the `environment` section in `docker-compose.yml`. Parameters to pay attention to before going live:

| Variable | Description | Default |
| --- | --- | --- |
| `SECRET_KEY` | Key used for JWT and data encryption; must be replaced with a random long string | none, required |
| `INIT_USERNAME` | Initial admin username, only effective on first database creation | `admin` |
| `INIT_PASSWORD` | Initial admin password, only effective on first database creation | none, required |
| `CORS_ORIGINS` | Allowed origins, comma-separated, wildcards not allowed | none, required |
| `PUBLIC_BASE_URL` | Canonical site URL, used for absolute feed / sitemap links and as the Host and CSRF origin allowlist | none, required |
| `ALLOWED_HOSTS` | Extra allowed hosts, comma-separated | empty |
| `COOKIE_SECURE` | Set to `true` when using HTTPS | `true` |
| `ENABLE_DOCS` | Whether to expose `/docs` and `/redoc` | `false` |
| `SECURITY_CONTACT` | Security contact email for `/.well-known/security.txt`; the path returns 404 when empty | empty |
| `TRUSTED_PROXY` | Trust `X-Forwarded-For` when behind a reverse proxy | `false` |
| `TRUSTED_HOPS` | Number of trusted proxy hops | `1` |
| `PROXY_PEER` | Peer address of the trusted proxy, CIDR supported; in port-mapping mode this is usually the docker bridge gateway (e.g. `172.17.0.1`) | `127.0.0.1` |
| `ENABLE_GIT` | Master switch for Git repos and sync | `true` |
| `LLM_API_KEY` / `LLM_BASE_URL` / `LLM_MODEL` | AI settings, used as a fallback for the admin panel configuration | empty / `https://api.openai.com/v1` / `gpt-4o-mini` |

## Data

Data is stored in two named volumes:

| Volume | Contents |
| --- | --- |
| `pnote-data` | PostgreSQL data directory, Redis persistence |
| `pnote-uploads` | Uploaded images, project source code, Git bare repos, backup archives |

`docker compose down` does not remove the volumes; only `docker compose down -v` does.

Backups are triggered manually on the "Backup & Migration" page in the admin panel (rate limited to 3/hour).
The output is a zip under `uploads/backups/` containing:

- `data/postgres-dump.sql`: a `pg_dump` export
- `uploads/`: uploaded files, excluding `backups/`, `git/`, `.tmp/`, and the display worktrees that can be rebuilt from the bare repos
- `git/<slug>.bundle`: one `git bundle` per bare repo
- `redis/stats-export.json`: the statistics keys in Redis
- `README.txt`: restore instructions

Downloading a backup requires re-authentication plus an admin-scope token. Backups contain encrypted tokens,
so restoring must use the same `SECRET_KEY`. There is currently no scheduled backup or one-click restore;
follow the `README.txt` inside the zip and restore manually.

## Upgrade

```bash
docker pull
docker compose up -d --build
```

The database and uploads directory are stored in volumes and are unaffected. New tables and columns are automatically added at startup, so there is no need to run migrations manually.

## Tech Stack

Backend: FastAPI + SQLAlchemy 2 + PostgreSQL + Redis, Python 3.13, with 17 runtime dependencies.
Markdown is rendered by `markdown-it-py` + Pygments and sanitized with nh3.

Frontend: Vue 3 + TypeScript + Vite, editing via `md-editor-v3`, KaTeX and Mermaid loaded on demand, sanitized again with DOMPurify.
Node is only used when building the image.

## Known Limitations

- Designed as a single-admin site; no registration or multi-user collaboration.
- No comment system.
- PlantUML diagrams require you to deploy your own rendering service and specify it via the frontend build-time variable `VITE_PLANTUML_SERVER`; the official image build does not inject it, so it degrades to a notice box by default.
- Backups can only be triggered manually; there is no scheduled backup or one-click restore.
- When placed behind a reverse proxy, it is recommended to enable `TRUSTED_PROXY` and configure `PROXY_PEER` correctly; otherwise the real client IP cannot be obtained, which affects rate limiting and UV deduplication.
