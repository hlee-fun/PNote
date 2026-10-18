# PNote

**中文** ｜ **English**

PNote 是一个自托管的个人笔记与项目展示站点。文章用 Markdown 写，项目可以挂上去做 GitHub 风格的展示，支持在线浏览源码、发布 Release，还能直接 `git clone` / `git push`。

PNote is a self-hosted personal note-taking and project showcase site. Articles are written in Markdown, and projects can be mounted as GitHub-style showcases, supporting online source browsing, Release publishing, and direct `git clone` / `git push`.

前后端、PostgreSQL、Redis 与前端 `dist` 全部打进**同一个 Docker 镜像**，启动后不需要再装任何依赖。

The frontend, backend, PostgreSQL, Redis and the frontend `dist` are all bundled into **a single Docker image**, so no additional dependencies are needed after startup.

- 演示站 / Demo：https://note.hlee.fun（演示站可能存在版本滞后！ / The demo site may lag behind the latest version!）
- 文档 / Docs：[中文](README-zh.md) · [English](README-en.md)

## 快速开始（单镜像 Compose 部署）/ Quick Start (single-image Compose deployment)

需要 Docker（带 `docker compose` 插件）。

Requires Docker (with the `docker compose` plugin).

```bash
docker pull hleefun/pnote:latest
docker pull ghcr.io/hlee-fun/pnote:latest
# 修改`docker-compose.yml`对应参数
docker compose up -d                               # 启动（后台）/ start (detached)
```

之后打开 http://localhost:8000 即可，后台在 `/admin`。

The first build compiles the frontend and takes a few minutes. After that, open http://localhost:8000; the admin panel is at `/admin`.

初始账号为 `INIT_USERNAME`（默认 `admin`），口令由 `INIT_PASSWORD` 指定，**镜像不内置任何默认口令**；若使用了弱口令，登录后会被强制要求修改。

The initial account is `INIT_USERNAME` (default `admin`), and the password is set by `INIT_PASSWORD`. **The image ships with no default password**; if a weak password is used, you will be forced to change it after logging in.

```bash
docker compose logs -f pnote   # 看日志 / View logs
docker compose down            # 停止，数据卷保留 / Stop, volumes preserved
```

数据存放在 `pnote-data`（数据库与 Redis）和 `pnote-uploads`（上传文件、项目源码、Git 仓库、备份）两个命名卷里，删除容器不会丢。

Data is stored in two named volumes: `pnote-data` (database and Redis) and `pnote-uploads` (uploaded files, project source, Git repositories, backups). Deleting the container will not lose the data.

更完整的说明见 [README-zh.md](README-zh.md) ／ See [README-en.md](README-en.md) for the full documentation.
