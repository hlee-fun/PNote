# PNote


**[中文](README-zh.md)** ｜ **[English](README-en.md)**


> **写字的地方，也是放代码的地方。**
> 一个自托管的个人内容站点：用 Markdown 写文章，同时把自己的项目挂在自己的域名下——访客能在线读源码、下 Release，你能直接 `git clone` / `git push`。

演示站：https://note.hlee.fun （内容为虚构样例，版本可能滞后）


## 部署

前置条件只有一个：Docker（含 `docker compose` 插件），Linux / macOS / Windows(WSL2) 皆可。

### 1. 从仓库取一份 `docker-compose.yml`

不需要 clone 整个仓库，也不需要源码——把仓库根目录的 `docker-compose.yml` 这**一个文件**下载到你的服务器上就够了。

```bash
# 在仓库页面直接下载 docker-compose.yml 单个文件
```

### 2. 改四项必填

打开这个文件，定位到 `services.pnote.environment`，把四项填好。文件里默认写成 `${SECRET_KEY:?...}` 这种「从宿主环境取值」的形式，两种写法都可以：要么改成字面值，要么保持原样、在 shell 里导出同名变量。

改完之后 `environment` 这一段像这样（**示意值**）：

```yaml
    environment:
      SECRET_KEY: "3f9a1c7d84e2b6a05c11df73a9e4b28c6d0f51a3b7c92e48a1d6f0b3c5e87a21"  # openssl rand -hex 32 的输出
      INIT_USERNAME: "admin"                                                            # 可省略，默认 admin
      INIT_PASSWORD: "N0te-My-First-Passphrase"                                          # 首次建账用，建账后可删
      CORS_ORIGINS: "https://note.example.com"
      PUBLIC_BASE_URL: "https://note.example.com"
      ALLOWED_HOSTS: "note.example.com,www.note.example.com"                             # 可选，多域名写这里
```

文件里其余部分——`read_only`、`cap_drop` / `cap_add`、`no-new-privileges`、`pids_limit` / `mem_limit` / `cpus`、`tmpfs`、`healthcheck`、日志轮转——是交付基线，**照原样留着，不要精简**。

| 变量 | 是什么 | 怎么填 |
| --- | --- | --- |
| `SECRET_KEY` | JWT 与数据加密密钥 | `openssl rand -hex 32` 生成，别复用、别提交到仓库 |
| `INIT_PASSWORD` | 初始管理员口令（仅首次建库生效） | 自己定；**镜像不含默认口令**，用了弱口令登录后会被强制修改 |
| `CORS_ORIGINS` | 允许的来源 | 你的域名，逗号分隔，**不允许通配符 `*`** |
| `PUBLIC_BASE_URL` | 站点规范地址 | **单个** `https://note.example.com`，用于 feed / sitemap 绝对链接，并作为 Host 与 CSRF 来源白名单；多域名写进 `ALLOWED_HOSTS` |

上面的 `note.example.com` 与那串密钥都是示意值，替换成你自己的即可。四项之外都有合理默认或可在后台配置，起站时不需要动。放在反向代理后面时，建议开启 `TRUSTED_PROXY` 并正确配置 `PROXY_PEER`，否则拿不到真实客户端 IP，会影响限流与 UV 去重。

### 3. 起站

```bash
docker compose up -d
```

容器启动脚本会依次拉起 PostgreSQL、Redis 和后端服务，并创建初始管理员（用户名 `INIT_USERNAME`，默认 `admin`）。

| 入口 | 地址 |
| --- | --- |
| 首页 | http://localhost:8000 |
| 后台 | http://localhost:8000/admin |
| 健康检查 | http://localhost:8000/api/health |

```bash
docker compose logs -f pnote   # 实时日志
docker compose down            # 停止并移除容器，数据卷保留
```

> 当前版本仍为**测试版本**：功能与接口可能继续调整，也难免有你先踩到的坑。欢迎在个人站和试验环境里试用，更期待你的意见与建议——每一条反馈都会被认真看。

---
