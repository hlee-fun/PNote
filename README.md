# PNote

PNote 是一个自托管的个人笔记与项目展示站点。文章用 Markdown 写，项目可以挂上去做 GitHub 风格的展示，
支持在线浏览源码、发布 Release，还能直接 `git clone` / `git push`。

前后端打进同一个 Docker 镜像，镜像内自带 PostgreSQL 和 Redis，启动后不需要再装任何依赖。

- 演示站：https://note.hlee.fun
- （演示站可能存在版本滞后！）
- 文档：[中文](README-zh.md) · [English](README-en.md)

> **注意**：正在加紧修改已知问题，尚未发布可用版本。

## 快速开始

需要 Docker（带 `docker compose` 插件）。

`SECRET_KEY`、`INIT_PASSWORD`、`CORS_ORIGINS`、`PUBLIC_BASE_URL` 四个变量必填，缺失会直接启动失败。
直接改 `docker-compose.yml` 里的 `environment` 即可（参数说明见完整文档「配置」）。

首次构建会编译前端，需要几分钟。之后打开 http://localhost:8000 即可，后台在 `/admin`。

初始账号为 `INIT_USERNAME`（默认 `admin`），口令由 `INIT_PASSWORD` 指定，
**镜像不内置任何默认口令**；若使用了弱口令，登录后会被强制要求修改。

```bash
docker compose logs -f pnote   # 看日志
docker compose down            # 停止，数据卷保留
```

数据存放在 `pnote-data`（数据库与 Redis）和 `pnote-uploads`（上传文件、项目源码、Git 仓库、备份）两个命名卷里，删除容器不会丢。

更完整的说明见 [README-zh.md](README-zh.md)。
