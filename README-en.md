# PNote

> **A place to write — and a place to keep your code.**
> A self-hosted personal content site: write articles in Markdown, and put your own projects under your own domain. Visitors can read the source online and download releases; you can `git clone` / `git push` straight against it.

Demo: https://note.hlee.fun (the content there is fictional sample material, and the version may lag behind)

Backend, frontend, PostgreSQL, Redis and the built frontend assets are **packaged into one Docker image**. The host needs no Python, no Node, no database, and doesn't even have to keep the source around — the only things you look after are a single `docker-compose.yml` and two data volumes.

---

## Why this

Plenty of people have ended up with the same setup: the blog on one platform, the code on another, images on a third, analytics on a fourth. Each one stands alone, so taken together it's one person maintaining five systems — and none of their uptime, latency, or rules are yours to decide. When a platform's status page adds a line to its timeline, all you can do is refresh; you have no seat at it. The first thing readers say afterwards tends to be "why did you stop posting?"

PNote takes that back: **under one domain, the articles and the project repositories are the same site, the same admin, the same database.**

It also doesn't ask you to disappear from public view in order to stay your own. Behind every project sits a real git bare repository — the source is with you — and it can be **pushed one way out to a mainstream public hosting platform or any self-hosted Git service** (dry-run preview and mirror push included). The copy on the public platform still gets found, still collects stars from people. What changes is that it has stopped being "the only copy" and become "a copy you can rebuild at any time".

Multi-user collaboration is on the agenda. Comments are not, for now. What it sets out to do completely is the quiet business of one person writing things and putting their work on display.

---

## Six things it does properly

**1 · One image, self-bootstrapping.** A single image carries the whole site. `docker compose up -d` and it's running; the site and the API share one origin and one port. The image contains **no fallback secret of any kind** — leave a required value unset and it refuses to start, rather than quietly coming up on something weak.

**2 · A cloud Git repo you can actually use.** The bare repo is the single source of truth; the file tree on the site is a checkout of it. `git push` from your machine and the source on the site is current. Admins have full control, HTTPS is enforced, and the raw git protocol is never exposed. Repos have three visibility states — public, private, draft; anything non-public returns 404 to visitors, and a draft can be scheduled to become public.

**3 · One-way sync out to a public platform.** Six target kinds (mainstream public hosting platforms plus any self-hosted Git service). A dry run shows exactly what would move before anything does. The push itself is classed as a sensitive operation, so re-authentication is required, and the API path demands an admin-scope token.

**4 · The theme system.** Light / dark / auto is orthogonal to the colour layer. Six built-in palettes — or **give it a single brand colour** and the remaining dozen-plus tone steps are derived from it, with text contrast guaranteed to meet WCAG AA by construction rather than by adjusting to taste. Third-party theme packages go through sandboxed extraction, a variable allowlist, and draft → enable → one-click rollback. Dangerous constructs in advanced CSS are sanitized server-side before they're stored, and the admin UI never applies them.

**5 · Security is the default state, not a list of plugins.** Two-factor authentication with one-time recovery codes. Re-authentication is forced on four operations: changing your password, downloading a backup, turning 2FA off, pushing a sync. API tokens come in read / write / admin scopes and can be given an expiry. Sessions can be revoked per device or everywhere at once. Audit logs filter by action, IP and time. The layer facing the public internet has CSP, HSTS, an image hotlink allowlist, an upload allowlist with magic-bytes checks, and Markdown sanitized twice — server-side and again in the browser. The container itself runs a read-only root filesystem, `cap_drop ALL` followed by only the capabilities it needs, with process and resource limits.

**6 · A closed loop in the admin.** PV / UV, a posting heatmap, traffic trends, download counters (anonymous, by IP with a 10-minute dedupe). Files, categories, custom sidebar cards and the legal page bodies are all edited there. Backups can be triggered by hand or scheduled daily / weekly / monthly, and the artifact includes a database dump plus a `git bundle` of every bare repo.

---

## What's in it

**Writing**
Markdown tables, task lists, footnotes; server-side code highlighting; KaTeX math and Mermaid diagrams loaded on demand; automatic table of contents. Drafts and scheduled publishing, server-side autosave, and a snapshot kept for every edit so anything can be rolled back. Tags, multi-level categories (self-referencing parent/child), pinning, view counts, anonymous likes. Batch import and export as a Markdown zip with frontmatter.

**Project showcase**
A project card list, filterable by language and searchable by keyword; README rendering; online browsing of the source tree with per-file download counts; release packaging and download; one cloud git bare repo per project, supporting `git clone` / `git push`; one-way sync to an external Git platform, with dry-run and mirror push; three-state repo visibility.

**Site**
The admin console (dashboard / file manager / backups / audit log / categories / settings); Simplified Chinese, Traditional Chinese and English interfaces, with the admin controlling which locales are enabled and which is the default; site-wide full-text search (articles and repositories returned in groups, `/` focuses the box); RSS 2.0 / Atom / sitemap.xml / robots.txt; site switches (close the whole site with an optional notice, private mode that opens only the about page, announcement banner); terms / privacy / cookie page bodies editable in the admin.

**AI assistance (optional — leaving it unconfigured affects nothing else)**
Generate a summary, generate a title, continue writing, translate Chinese into English — the result is only displayed for copying and **never overwrites your text**. Compatible with both mainstream model wire protocols; the API key is encrypted at rest and you just enter it in the admin. How good it is depends on the model you connect.

---

## Deployment

One prerequisite: Docker (with the `docker compose` plugin). Linux / macOS / Windows (WSL2) all work.

### 1. Take one `docker-compose.yml` from the repository

You don't need to clone the repo, and you don't need the source at all. Downloading the **single file** `docker-compose.yml` from the repository root to your server is enough.

```bash
# Download just the docker-compose.yml file from the repository page
```

### 2. Set the four required values

Open the file, find `services.pnote.environment`, and fill in the four. In the file they are written in the `${SECRET_KEY:?...}` form, i.e. "read from the host environment" — both styles work: replace them with literal values, or leave them as they are and export variables of the same name in your shell.

When you're done the `environment` block looks something like this (**illustrative values**):

```yaml
    environment:
      SECRET_KEY: "3f9a1c7d84e2b6a05c11df73a9e4b28c6d0f51a3b7c92e48a1d6f0b3c5e87a21"  # output of `openssl rand -hex 32`
      INIT_USERNAME: "admin"                                                            # optional, defaults to admin
      INIT_PASSWORD: "N0te-My-First-Passphrase"                                         # used to create the first account, can be removed afterwards
      CORS_ORIGINS: "https://note.example.com"
      PUBLIC_BASE_URL: "https://note.example.com"
      ALLOWED_HOSTS: "note.example.com,www.note.example.com"                            # optional, extra domains go here
```

Everything else in the file — `read_only`, `cap_drop` / `cap_add`, `no-new-privileges`, `pids_limit` / `mem_limit` / `cpus`, `tmpfs`, `healthcheck`, log rotation — is the delivery baseline. **Leave it exactly as it is; don't trim it.**

| Variable | What it is | How to set it |
| --- | --- | --- |
| `SECRET_KEY` | Key for JWT and data encryption | Generate with `openssl rand -hex 32`; don't reuse it, don't commit it |
| `INIT_PASSWORD` | Initial admin password (takes effect only when the database is first created) | Your choice; **the image ships no default password**, and a weak one forces a change after the first login |
| `CORS_ORIGINS` | Allowed origins | Your domains, comma-separated, **the wildcard `*` is not accepted** |
| `PUBLIC_BASE_URL` | Canonical site URL | A **single** `https://note.example.com`; used for absolute feed / sitemap links and as the Host and CSRF origin allowlist; extra domains go in `ALLOWED_HOSTS` |

The `note.example.com` above and that key string are illustrative — replace them with your own. Everything beyond the four has a sensible default or is configurable in the admin, and you don't need to touch it to bring the site up. Behind a reverse proxy, enable `TRUSTED_PROXY` and set `PROXY_PEER` correctly; otherwise the real client IP isn't available, which affects rate limiting and UV deduplication.

### 3. Bring it up

```bash
docker compose up -d
```

The container's startup script brings up PostgreSQL, Redis and the backend service in order, and creates the initial admin account (username from `INIT_USERNAME`, default `admin`).

| Entry | URL |
| --- | --- |
| Home | http://localhost:8000 |
| Admin | http://localhost:8000/admin |
| Health check | http://localhost:8000/api/health |

```bash
docker compose logs -f pnote   # live logs
docker compose down            # stop and remove the container; the data volumes are kept
```

> The current version is still a **test release**: features and APIs may keep changing, and there will be rough edges you hit before anyone else does. Try it on a personal site or in a sandbox — and opinions and suggestions are especially welcome. Every piece of feedback gets read carefully.

---

## Data, backups, upgrades

Your data lives in exactly two named volumes, and `docker compose down` does not remove them (only `down -v` does):

| Volume | Contents |
| --- | --- |
| `pnote-data` | PostgreSQL data directory, Redis persistence |
| `pnote-uploads` | Uploaded images, project source, Git bare repos, backup archives |

**Backups** are triggered from the "Backups" page in the admin. Supported: daily / weekly / monthly schedules with run time and timezone, retention by number of copies, multi-select scope (database / uploaded files / cloud repos / Redis), and an in-site record of past runs. The artifact is a zip containing the `pg_dump` export, the uploaded files, a `git bundle` for every bare repo, an export of the statistics keys, and a set of restore instructions. Downloading a backup requires re-authentication plus an admin-scope token; restoring must be done with the same `SECRET_KEY`.

**Upgrades**:

```bash
docker compose pull && docker compose up -d
```

The database and everything uploaded sit in the volumes and are unaffected; new tables and columns are added automatically at startup, so no manual migration is needed.

**Leaving**: articles export in batch as a Markdown zip with frontmatter; the repositories are standard git bare repos, so `git clone` takes all of it with you. Your data is never bound to a proprietary format.

---

## Known limitations (stated plainly)

- Positioned as a **single-admin site**: no registration, no multi-user collaboration. No comment system — visitors can only leave an anonymous like.
- "Linked collaboration" (several people completing one private project inside your own deployment) is at the **concept stage and not implemented**, with no timeline.
- Backups can be scheduled, but there is **no one-click restore**.
- Vector semantic search is **off by default**; turning it on requires a real embedding model plus validation before it opens up to visitors.
- PlantUML diagrams need your own rendering service, specified through a frontend build-time variable. The official image build doesn't inject that variable, so they degrade to a notice box by default.
- Sync targets are validated platform by platform. At the moment one of the mainstream public hosting platforms is fully working; the rest are still being verified.
- If `PUBLIC_BASE_URL` / `SITE_HOST` / `ALLOWED_HOSTS` are left unset, the Host and CSRF origin checks are bypassed for the whole layer with a warning, and absolute links in the feed / sitemap can be poisoned through the Host header.
- Rate limiting is implemented in code (by real IP / account) and has no standalone environment variable to tune; the 2GB limit on git smart HTTP request bodies is hard-coded.
- The delivery form is a single container plus docker-compose, nothing else. Multi-node and cluster orchestration are out of scope.

---

> **Your domain deserves to carry things that are yours.**
