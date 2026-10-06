# PNote

PNote 是一个自托管的个人内容站点：用 Markdown 写文章，同时把自己的项目挂上去做展示，
访客可以在线浏览源码、下载 Release，管理员还能直接 `git clone` / `git push`。

前后端打包在一个 Docker 镜像里，镜像内已内置 PostgreSQL 和 Redis，宿主机不需要安装 Python、Node 或数据库。

演示站：https://note.hlee.fun

（演示站可能存在版本滞后！）

> **注意**：正在加紧修改已知问题，尚未发布可用版本。

## 功能

### 写文章

- Markdown 编辑：表格、任务列表、脚注、代码高亮（Pygments）、KaTeX 公式、Mermaid 图、自动目录
- 草稿与定时发布，服务端自动保存，每次修改留快照，可随时回滚
- 标签、多级分类（自引用父子）、置顶、浏览量与匿名点赞
- 批量导入导出（带 frontmatter 的 Markdown zip）

### 展示项目

- 项目卡片列表，支持按语言筛选和关键字搜索
- README 渲染，源码目录在线浏览与下载
- 打 Release，把源码目录打成 zip 提供下载
- 每个项目对应一个云端 Git 裸仓，支持 `git clone` / `git push`（smart HTTP，仅管理员，强制 HTTPS）
- 单向同步到 GitHub / GitLab / Gitea / Gitee / Bitbucket / 自建，支持 dry-run 与镜像推送
  （目前仅测试过 GitHub，其他仓库尚在加紧测试中）

### 站点

- 后台控制台：仪表盘（PV/UV、发文热力图、趋势）、文件管理、备份、审计日志、分类、设置
- 界面支持简体中文、繁体中文、英文，后台可控制启用哪些语言以及默认语言
- 全站搜索（PostgreSQL 全文检索 + ILIKE，中文走 ILIKE）
- RSS 2.0 / Atom 订阅，sitemap.xml 与 robots.txt
- 站点开关：整站关闭（可附说明）、隐私站点（仅开放关于页）、公告横幅
- 法律页：条款、隐私、Cookie 说明，正文可在后台编辑

### 安全

- 两步验证（TOTP）与一次性恢复码
- 敏感操作（改密、下载备份、关闭 2FA、同步推送）需二次认证
- API Token（read / write / admin，可设过期时间），走 `X-API-Key` 头
- 会话管理：在线会话列表、按设备吊销、全部下线
- 审计日志、CSP、HSTS、防盗链白名单、上传白名单与 magic bytes 校验

### AI 辅助（可选）

- 生成摘要、标题、续写、中译英（结果只展示可复制，不覆盖正文）
- 支持 Anthropic 与 OpenAI 两种协议，API Key 加密入库
- 在后台填 API Key，不配置也不影响其他功能
- AI辅助性能与接入模型性能有关

## 安装

部署方式只有 Docker Compose 一种：`docker compose` 

前置条件：Docker（含 `docker compose` 插件），Linux / macOS / Windows(WSL2) 都可以。

`SECRET_KEY`、`INIT_PASSWORD`、`CORS_ORIGINS`、`PUBLIC_BASE_URL` 四项必填，缺失会直接启动失败。
直接改 `docker-compose.yml` 里的 `environment` 即可，各参数含义见「配置」。

```bash
# 修改`docker-compose.yml`对应参数
docker compose up -d
```

容器启动脚本会依次拉起 PostgreSQL、Redis 和后端服务，并创建初始管理员。

### 访问地址

前后端同源，都走 8000 端口：

| 入口 | 地址 |
| --- | --- |
| 首页 | http://localhost:8000 |
| 后台 | http://localhost:8000/admin |
| 健康检查 | http://localhost:8000/api/health |

初始账号为 `INIT_USERNAME`（默认 `admin`），口令由 `INIT_PASSWORD` 指定，
**镜像不内置任何默认口令**；若使用了弱口令，登录后会被强制要求修改。

### 常用命令

```bash
docker compose logs -f pnote   # 实时日志
docker compose down            # 停止并移除容器，数据卷保留
docker compose up -d --build   # 更新代码后重新构建
```

## 配置

改 `docker-compose.yml` 的 `environment` 即可。上线前需要留意的参数：

| 变量 | 说明 | 默认值 |
| --- | --- | --- |
| `SECRET_KEY` | JWT 与数据加密用的密钥，必须换成随机长串 | 无，必填 |
| `INIT_USERNAME` | 初始管理员用户名，仅首次建库生效 | `admin` |
| `INIT_PASSWORD` | 初始管理员口令，仅首次建库生效 | 无，必填 |
| `CORS_ORIGINS` | 允许的来源，逗号分隔，不允许通配符 | 无，必填 |
| `PUBLIC_BASE_URL` | 站点规范地址，用于 feed / sitemap 绝对链接，并作为 Host 与 CSRF 来源白名单 | 无，必填 |
| `ALLOWED_HOSTS` | 额外放行的 Host，逗号分隔 | 空 |
| `COOKIE_SECURE` | 用 HTTPS 时设为 `true` | `true` |
| `ENABLE_DOCS` | 是否开放 `/docs`、`/redoc` | `false` |
| `SECURITY_CONTACT` | 安全联系邮箱，用于 `/.well-known/security.txt`，留空则该路径返回 404 | 空 |
| `TRUSTED_PROXY` | 反向代理后采信 `X-Forwarded-For` | `false` |
| `TRUSTED_HOPS` | 信任的代理跳数 | `1` |
| `PROXY_PEER` | 受信代理的对端地址，支持 CIDR；端口映射模式下通常是 docker 网桥网关（如 `172.17.0.1`） | `127.0.0.1` |
| `ENABLE_GIT` | Git 仓库与同步功能总开关 | `true` |
| `LLM_API_KEY` / `LLM_BASE_URL` / `LLM_MODEL` | AI 配置，作为后台设置的兜底 | 空 / `https://api.openai.com/v1` / `gpt-4o-mini` |

## 数据

数据放在两个命名卷：

| 卷 | 内容 |
| --- | --- |
| `pnote-data` | PostgreSQL 数据目录、Redis 落盘 |
| `pnote-uploads` | 上传的图片、项目源码、Git 裸仓、备份归档 |

`docker compose down` 不会删除卷，只有 `docker compose down -v` 才会。

备份在后台「备份与迁移」页面手动触发（限流 3 次/小时），产物是 `uploads/backups/` 下的 zip，包含：

- `data/postgres-dump.sql`：`pg_dump` 导出
- `uploads/`：上传文件，排除 `backups/`、`git/`、`.tmp/`，以及可由裸仓重建的展示工作树
- `git/<slug>.bundle`：每个裸仓一个 `git bundle`
- `redis/stats-export.json`：Redis 里的统计键
- `README.txt`：恢复指引

下载备份需要二次认证 + admin scope 的 Token。备份里含加密令牌，恢复时必须使用同一份 `SECRET_KEY`。
目前没有定时备份和一键恢复，需按 zip 内 `README.txt` 手工操作。

## 升级

```bash
docker pull
docker compose up -d --build
```

数据库和上传目录都在卷里，不受影响。新增的表和字段会在启动时自动补齐，不需要手动跑迁移。

## 技术栈

后端 FastAPI + SQLAlchemy 2 + PostgreSQL + Redis，Python 3.13，运行时依赖 17 个。
Markdown 由 `markdown-it-py` + Pygments 渲染、nh3 消毒。

前端 Vue 3 + TypeScript + Vite，`md-editor-v3` 编辑，按需加载 KaTeX 与 Mermaid，DOMPurify 二次消毒。
Node 只在构建镜像时用到。

## 已知限制

- 定位是单管理员站点，没有注册和多用户协作。
- 没有评论系统。
- PlantUML 图表需要自己部署渲染服务，并通过前端构建期变量 `VITE_PLANTUML_SERVER` 指定；
  官方镜像构建时未注入该变量，默认降级为提示框。
- 备份仅支持手动触发，没有定时备份与一键恢复。
- 放在反向代理后面时，建议开启 `TRUSTED_PROXY` 并正确配置 `PROXY_PEER`，
  否则拿不到真实客户端 IP，会影响限流和 UV 去重。
