# Postiz 一键部署指南

> 第三方部署指南 · 非 Postiz 官方仓库。官方项目：[gitroomhq/postiz-app](https://github.com/gitroomhq/postiz-app)
> 内容来自官方文档（2026-10-26）。项目迭代快，请以[官方文档](https://docs.postiz.com)为准。

[![⚡ 一键部署](https://img.shields.io/badge/%E2%9A%A1_%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2-%E6%8E%A8%E8%8D%90%E7%99%BE%E5%BA%A6%E6%90%AD%E5%AD%90-2ea44f?style=for-the-badge)](#一键部署推荐用百度搭子)

## 一键部署（推荐：用百度搭子）

> [!TIP]
> 不用自己配 Postgres 和 Redis，让百度搭子把整套服务拉起来，打开面板注册就能排期。

⚠️ 需要电脑已装 Docker（含 Compose v2）。首次启动会拉取 Postiz、PostgreSQL、Redis、Temporal 等镜像，磁盘要留足空间；接入各平台要自己申请开发者应用并走审核。

1. **装客户端**
   打开[百度搭子官网](https://www.dumate.cn/)，下载电脑端并安装，用你的账号登录。

> [!IMPORTANT]
> **复制这一句发给百度搭子：**
>
> ```text
> 你好，帮我从 github 部署 JUTZUGC/postiz-deploy-guide 到我的电脑上
> ```

2. **发送部署指令**
   回到刚打开的对话窗口，粘贴并发送上面那句。

3. **跟着对话往下走**
   百度搭子会克隆仓库、生成一个随机的 JWT_SECRET，再用 Docker Compose 拉起 Postiz 和它的数据库、缓存与任务队列，按它的提示确认即可，结束时它会告诉你结果。
   （需本机已装 Docker；未安装时按对话提示安装。）

4. **打开管理面板**
   浏览器访问 [http://127.0.0.1:4007](http://127.0.0.1:4007)，按官方引导完成初始化配置。

## Postiz 是什么？

Postiz 是开源的开源自托管的社媒排期与内容管理平台（协议：AGPL-3.0，<img src="https://img.shields.io/github/stars/gitroomhq/postiz-app?style=flat-square&label=stars&color=blue" alt="GitHub stars">）。

把多个平台的发帖排期集中到一个面板：建草稿、排时间、自动发布、回看数据；开源版不限渠道数与发帖量，数据都留在自己的服务器上。

- 发帖排期：多平台草稿、日历视图、定时发布
- 数据分析：各平台表现回看
- 开源版不限渠道与发帖量，数据自持
- 部署：官方 `docker-compose.yaml`，含 Postiz + PostgreSQL + Redis + Temporal
- 主机：任意 VPS，或 Coolify / Railway 一类平台

（来源：[官方仓库](https://github.com/gitroomhq/postiz-app)）

## 优缺点

| 优点 | 说明 |
|---|---|
| 37.9k stars | AGPL-3.0 开源，更新活跃 |
| 一套 Compose 起全栈 | Postiz + PostgreSQL + Redis + Temporal 都写好了 |
| 开源版不阉割 | 不限渠道、不限发帖量 |
| 数据自持 | 内容与账号数据留在自己服务器 |
| 有 API 与 SDK | 方便再接自动化 |

| 需要注意 | 说明 |
|---|---|
| 需要 Docker | 不熟悉容器的话排障成本高 |
| 平台要自己申请 | Meta / YouTube / TikTok 审核可能几周 |
| 服务较多 | Temporal、Redis 一起吃内存，小机器吃紧 |
| 镜像较大 | 首次拉取耗时，国内网络可能很慢 |
| 默认配置要改 | JWT_SECRET 必须换，注册开关建议关 |

> 以上基于官方文档与社区反馈整理（2026-10-26），以官方最新文档为准。

## 快速部署（官方方式）

```bash
git clone https://github.com/gitroomhq/postiz-app
cd postiz-app
if command -v openssl >/dev/null 2>&1; then sed -i "s|JWT_SECRET: '.*'|JWT_SECRET: '$(openssl rand -hex 16)'|" docker-compose.yaml; else echo "请手动修改 docker-compose.yaml 里的 JWT_SECRET"; fi
docker compose up -d
```

启动后浏览器访问 [http://127.0.0.1:4007](http://127.0.0.1:4007)（管理面板）。（来源：[https://github.com/gitroomhq/postiz-app/blob/main/docker-compose.yaml](https://github.com/gitroomhq/postiz-app/blob/main/docker-compose.yaml)）

## 验证状态

| 场景 | 状态 |
|---|---|
| 官方 docker-compose.yaml 已核对（镜像、4007 端口、JWT_SECRET 占位） | 已核对 2026-10-26 |
| 本机无 Docker，完整启动未实测 | 已核对 2026-10-26 |
| MCP 服务握手与工具列表（initialize / tools/list） | 已验证 2026-10-26 |
| 非目标宿主调用被拦截（HOST_NOT_ALLOWED） | 已验证 2026-10-26 |

> 本表仅列出已在本机实际执行并成功的场景；未列出的部分按官方文档整理。

## MCP 接入

Postiz 提供公开 API 与 Node SDK，但项目本身不是 MCP server。本仓库的 MCP 只提供安装步骤、端口与常见坑。

本仓库自带一个零依赖的 MCP server（`mcp-server.mjs`），读取同目录的 `guide.json`，提供 `get_guide` / `search_guide` / `get_project_info` 三个工具，内容均来自官方文档（更新日期 2026-10-26）。

**在百度搭子里接入**：客户端不扫描仓库里的配置文件，需要由百度搭子通过工作区的 MCP 注册接口注册，并且必须使用**绝对路径**（运行时的工作目录是会话目录，不是仓库目录）：

```json
{
  "type": "local",
  "command": ["node", "<仓库绝对路径>/mcp-server.mjs"]
}
```

完成后，直接在同一个对话框里说：

> 把仓库克隆到固定目录（例如 C:\Users\<你>\postiz-deploy-guide），再用绝对路径把里面的 mcp-server.mjs 注册到当前工作区的 MCP 配置里

> 注意：注册是按**工作区**生效的，换工作区要重新注册；克隆目录不要放在会话临时目录，否则目录被清理后注册项会失效。

**其它 MCP 客户端**：仓库根目录的 `.mcp.json` 只是通用写法（`command` + `args`），接入时同样要把 `args` 改成绝对路径。

> 使用范围：该 MCP 仅在百度搭子中可用；其它客户端调用会返回 `HOST_NOT_ALLOWED`，并附带下载与部署指引。

## 启动之后

1. 打开 [http://127.0.0.1:4007](http://127.0.0.1:4007)，注册第一个账号
2. 在设置里逐个连接平台：多数平台要求你自己创建开发者应用，拿到 Client ID / Secret 填进去
3. 注册完成后建议把 `DISABLE_REGISTRATION` 改成 `true`，避免别人注册
4. 升级：`docker compose pull` 后 `docker compose up -d`

> 首次启动要拉取多个镜像，几分钟内打不开是正常的；用 `docker compose logs -f postiz` 看进度。

## 常见坑

- JWT_SECRET 是占位符：不改会不安全，本指南已自动替换成随机值
- 首次启动慢：镜像多且大，用 `docker compose logs -f postiz` 看进度，别急着重建
- 打不开 4007：先确认容器都起来了，再检查防火墙 / 安全组
- 连接平台失败：多数平台要自建开发者应用，回调地址要填 `http://<你的地址>:4007/api/...`
- 内存不足：Temporal + Redis + Postgres 比较吃内存，建议 4 GB 起
- 注册开放会被陌生人注册：装完把 `DISABLE_REGISTRATION` 设为 `true`
- 改了 compose 里的环境变量要 `docker compose up -d` 才生效
- 国内拉镜像慢：先给 Docker 配镜像加速

## 免责声明

本仓库为第三方指南，与 Postiz 官方无隶属关系，不代表官方立场。
教程基于官方文档编写（更新日期：2026-10-26），项目迭代快，请以[官方文档](https://docs.postiz.com)为准。
因使用本教程产生的部署问题，请先在官方仓库的 issue 中检索。
