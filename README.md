# PNote

## 一、技术栈

| 层 | 选型 |
| --- | --- |
| 后端 | FastAPI · Uvicorn · SQLAlchemy 2.0 · Pydantic 2（Python 3.13） |
| 前端 | Vue 3.5 · TypeScript · Vite 7 · Pinia · Vue Router · vue-i18n · Sass |
| 编辑器 | `md-editor-v3`（内置 CodeMirror，支持 KaTeX / Mermaid / PlantUML） |
| 数据库 | PostgreSQL（`UTF8`，`psycopg2` 驱动） |
| 缓存 / 统计 | Redis（PV 用 `INCR`、UV 用 HyperLogLog；不可用时自动降级为进程内计数） |
| Markdown 渲染 | `markdown-it-py`（GFM）+ Pygments 代码高亮 + `nh3` 白名单消毒 |
| 安全 | PyJWT（HS256）+ bcrypt + 自实现 TOTP（RFC 6238）+ slowapi 限流 + Fernet 加密 |
| 部署 | 单 Docker 镜像，容器内 `start.sh` 编排 PG + Redis + Uvicorn |


---

## 二、功能总览

### 2.1 公开站点（访客可见）



### 2.2 Markdown 与编辑器



### 2.3 认证与安全



### 2.4 文章管理



### 2.5 仓库（项目）管理



### 2.6 在线文件管理



### 2.7 站点设置与个人资料



### 2.8 运维与统计



**多语言**：内置 `zh-CN` / `en-US` / `zh-TW` 

> 设计说明：这是**单管理员**站点。
---
