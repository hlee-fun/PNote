# PNote


**[中文](README-zh.md)** ｜ **[English](README-en.md)**


> **写字的地方，也是放代码的地方。**
>
> _A place to write — and a place to keep your code._
>
> 一个自托管的个人内容站点：用 Markdown 写文章，同时把自己的项目挂在自己的域名下——访客能在线读源码、下 Release，你能直接 `git clone` / `git push`。
>
> _A self-hosted personal content site: write articles in Markdown, and put your own projects under your own domain. Visitors can read the source online and download releases; you can `git clone` / `git push` straight against it._

演示站：https://note.hlee.fun （内容为虚构样例，版本可能滞后）

_Demo: https://note.hlee.fun (the content there is fictional sample material, and the version may lag behind)_


## 部署

_Deployment_

前置条件只有一个：Docker（含 `docker compose` 插件），Linux / macOS / Windows(WSL2) 皆可。

_One prerequisite: Docker (with the `docker compose` plugin). Linux / macOS / Windows (WSL2) all work._

### 1. 从仓库取一份 `docker-compose.yml`

_Take one `docker-compose.yml` from the repository_

不需要 clone 整个仓库，也不需要源码——把仓库根目录的 `docker-compose.yml` 这**一个文件**下载到你的服务器上就够了。

_You don't need to clone the repo, and you don't need the source at all. Downloading the **single file** `docker-compose.yml` from the repository root to your server is enough._

```bash
# 在仓库页面直接下载 docker-compose.yml 单个文件
# Download just the docker-compose.yml file from the repository page
```

### 2. 改四项必填

_Set the four required values_

打开这个文件，定位到 `services.pnote.environment`，把四项填好。文件里默认写成 `${SECRET_KEY:?...}` 这种「从宿主环境取值」的形式，两种写法都可以：要么改成字面值，要么保持原样、在 shell 里导出同名变量。

_Open the file, find `services.pnote.environment`, and fill in the four. In the file they are written in the `${SECRET_KEY:?...}` form, i.e. "read from the host environment" — both styles work: replace them with literal values, or leave them as they are and export variables of the same name in your shell._

改完之后 `environment` 这一段像这样（**示意值**）：

_When you're done the `environment` block looks something like this (**illustrative values**):_

```yaml
    environment:
      SECRET_KEY: "3f9a1c7d84e2b6a05c11df73a9e4b28c6d0f51a3b7c92e48a1d6f0b3c5e87a21"  # openssl rand -hex 32 的输出 / output of `openssl rand -hex 32`
      INIT_USERNAME: "admin"                                                            # 可省略，默认 admin / optional, defaults to admin
      INIT_PASSWORD: "N0te-My-First-Passphrase"                                          # 首次建账用，建账后可删 / used to create the first account, can be removed afterwards
      CORS_ORIGINS: "https://note.example.com"
      PUBLIC_BASE_URL: "https://note.example.com"
      ALLOWED_HOSTS: "note.example.com,www.note.example.com"                             # 可选，多域名写这里 / optional, extra domains go here
```

文件里其余部分——`read_only`、`cap_drop` / `cap_add`、`no-new-privileges`、`pids_limit` / `mem_limit` / `cpus`、`tmpfs`、`healthcheck`、日志轮转——是交付基线，**照原样留着，不要精简**。

_Everything else in the file — `read_only`, `cap_drop` / `cap_add`, `no-new-privileges`, `pids_limit` / `mem_limit` / `cpus`, `tmpfs`, `healthcheck`, log rotation — is the delivery baseline. **Leave it exactly as it is; don't trim it.**_

| 变量<br>_Variable_ | 是什么<br>_What it is_ | 怎么填<br>_How to set it_ |
| --- | --- | --- |
| `SECRET_KEY` | JWT 与数据加密密钥<br>_Key for JWT and data encryption_ | `openssl rand -hex 32` 生成，别复用、别提交到仓库<br>_Generate with `openssl rand -hex 32`; don't reuse it, don't commit it_ |
| `INIT_PASSWORD` | 初始管理员口令（仅首次建库生效）<br>_Initial admin password (takes effect only when the database is first created)_ | 自己定；**镜像不含默认口令**，用了弱口令登录后会被强制修改<br>_Your choice; **the image ships no default password**, and a weak one forces a change after the first login_ |
| `CORS_ORIGINS` | 允许的来源<br>_Allowed origins_ | 你的域名，逗号分隔，**不允许通配符 `*`**<br>_Your domains, comma-separated, **the wildcard `*` is not accepted**_ |
| `PUBLIC_BASE_URL` | 站点规范地址<br>_Canonical site URL_ | **单个** `https://note.example.com`，用于 feed / sitemap 绝对链接，并作为 Host 与 CSRF 来源白名单；多域名写进 `ALLOWED_HOSTS`<br>_A **single** `https://note.example.com`; used for absolute feed / sitemap links and as the Host and CSRF origin allowlist; extra domains go in `ALLOWED_HOSTS`_ |

上面的 `note.example.com` 与那串密钥都是示意值，替换成你自己的即可。四项之外都有合理默认或可在后台配置，起站时不需要动。放在反向代理后面时，建议开启 `TRUSTED_PROXY` 并正确配置 `PROXY_PEER`，否则拿不到真实客户端 IP，会影响限流与 UV 去重。

_The `note.example.com` above and that key string are illustrative — replace them with your own. Everything beyond the four has a sensible default or is configurable in the admin, and you don't need to touch it to bring the site up. Behind a reverse proxy, enable `TRUSTED_PROXY` and set `PROXY_PEER` correctly; otherwise the real client IP isn't available, which affects rate limiting and UV deduplication._

### 3. 起站

_Bring it up_

```bash
docker compose up -d
```

容器启动脚本会依次拉起 PostgreSQL、Redis 和后端服务，并创建初始管理员（用户名 `INIT_USERNAME`，默认 `admin`）。

_The container's startup script brings up PostgreSQL, Redis and the backend service in order, and creates the initial admin account (username from `INIT_USERNAME`, default `admin`)._

| 入口<br>_Entry_ | 地址<br>_URL_ |
| --- | --- |
| 首页<br>_Home_ | http://localhost:8000 |
| 后台<br>_Admin_ | http://localhost:8000/admin |
| 健康检查<br>_Health check_ | http://localhost:8000/api/health |

```bash
docker compose logs -f pnote   # 实时日志 / live logs
docker compose down            # 停止并移除容器，数据卷保留 / stop and remove the container; the data volumes are kept
```

> 当前版本仍为**测试版本**：功能与接口可能继续调整，也难免有你先踩到的坑。欢迎在个人站和试验环境里试用，更期待你的意见与建议——每一条反馈都会被认真看。
>
> _The current version is still a **test release**: features and APIs may keep changing, and there will be rough edges you hit before anyone else does. Try it on a personal site or in a sandbox — and opinions and suggestions are especially welcome. Every piece of feedback gets read carefully._

---
