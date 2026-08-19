# grok-reg-tool

`grok-reg-tool` 是一个可自部署的 Grok 注册机 Web 控制台，支持 Docker 一键构建、DrissionPage 自动化注册、邮件验证码读取、SSO 输出和本地账号号池管理。

适合搜索的关键词：Grok 注册机、Grok 自动注册、Grok Web 控制台、Grok 账号号池、Grok SSO、xAI 自动化、DrissionPage 自动化、Docker 自部署注册工具。

> 本项目与 xAI、Grok、X 没有官方关联。请仅在合法、合规、获得许可的研究、学习或自托管实验环境中使用。

## 功能

- Web 控制台：启动、停止、查看注册任务和实时日志。
- 内置注册机：Python 注册脚本已放在 `register/`，Docker 构建时自动安装依赖。
- 邮件后端：支持对接 `cloudflare_temp_email` 读取验证码。
- 号池管理：本地保存账号记录和 SSO 输出。
- Docker 部署：运行数据默认保存到 `docker/data/`。
- 在线更新：对接「夏稚Auth Studio / 新云Auth」授权系统的 Open API，检查新版本、下载完整更新包并校验 SHA256，由宿主机脚本一键应用更新。

## 快速部署

```bash
git clone https://github.com/FengZi1221/grok-reg-tool.git
cd grok-reg-tool/docker
cp .env.example .env
```

编辑 `docker/.env`：

```env
WEB_PORT=6657
RUN_COUNT=10

MAIL_API_BASE=
MAIL_ADMIN_AUTH=
MAIL_DOMAIN=

HTTP_PROXY=
BROWSER_PROXY=
COOKIE_SECURE=
```

启动：

```bash
docker compose up -d --build
```

访问：

```text
http://你的服务器IP:6657
```

初始 Web 登录信息：

```bash
docker logs grok-reg-tool
```

首次登录后请修改默认用户名和密码。

直接用 `http://服务器IP:6657` 访问时，`COOKIE_SECURE` 请留空。只有放在 HTTPS 反向代理后面时，才建议设置 `COOKIE_SECURE=1`。

## 邮件后端配置

本项目推荐对接 [dreamhunter2333/cloudflare_temp_email](https://github.com/dreamhunter2333/cloudflare_temp_email)。

请先按它的官方文档部署好 Cloudflare 临时邮箱服务。本 README 不重复它的部署教程，只说明本项目要填哪些值：

| 配置项 | 填写内容 |
| --- | --- |
| `MAIL_API_BASE` | cloudflare_temp_email 的后端 Worker/API 根地址 |
| `MAIL_ADMIN_AUTH` | cloudflare_temp_email 的网站 admin 密码，会作为 `x-admin-auth` 使用 |
| `MAIL_DOMAIN` | 已在 cloudflare_temp_email 中配置并可收信的域名 |

调用关系很简单：本项目会请求 `MAIL_API_BASE + /admin/new_address` 创建邮箱，再用返回的 `jwt` 轮询邮件并提取验证码。

## 在线更新

本项目支持通过「夏稚Auth Studio / 新云Auth」授权系统的 Open API 实现在线更新。

### 1. 授权系统后台准备

- 在授权系统创建一个应用（例如「Grok-GPT注册机」），记录 `application_id`。
- 在「系统设置 → Open API」生成鉴权密钥（`xzak_` 开头）。
- 在「应用 → 版本」中发布更新包（完整项目 zip）：
  - `version_name` 例如 `0.1.3`
  - `version_code` 为数字，规则：主版本×100 + 次版本×10 + 修订，例如 `0.1.3 → 13`、`1.2.0 → 120`
  - 上传完整安装包（kind = `package`）

### 2. 配置更新源

打开 Web 控制台 →「设置」→「授权系统在线更新」：

| 配置项 | 说明 |
| --- | --- |
| 启用开关 | 开启后首页「检查更新」优先走授权系统 |
| 授权系统地址 | 例如 `https://power.1tim.com` |
| Open API Key | `xzak_` 开头的密钥 |
| 应用 ID | 授权系统中创建的应用 ID |
| 本地版本 code | 本地版本号对应的数字 code |

### 3. 检查与下载

首页点击「检查更新」：
- 若有新版本会显示 `v0.1.3`、强制更新标识和更新日志。
- 点击「下载更新包」将完整包下载到持久目录 `docker/data/updates/`，并校验 SHA256。

### 4. 应用更新（宿主机执行）

下载完成后，在宿主机项目目录执行：

```bash
cd /root/grok-reg-tool
./apply-update.sh 0.1.3
```

脚本会解压更新包、同步项目文件（保留 `docker/data` 与 `docker/.env`）、重新构建并重建容器。

> 更新包下载目录：`docker/data/updates/`（容器内 `/data/updates/`）。

### 5. 一键发布更新包（开发者使用）

每次改完代码、重建容器后，可在宿主机执行发布脚本，自动打包并上传到授权服务器、填写更新日志、发布版本：

```bash
cd /root/grok-reg-tool
AUTH_PASS='<授权系统管理员密码>' ./release-update.sh 0.1.4 14 '【新增】
1. ...
【修复】
1. ...'
```

参数说明：
- 第一个参数：版本名，如 `0.1.4`
- 第二个参数：版本 code，规则为 主版本×100 + 次版本×10 + 修订，如 `0.1.4 → 14`
- 第三个参数：更新日志文本，可用 `\n` 换行

可选环境变量：`AUTH_SERVER`（默认 `https://power.1tim.com`）、`AUTH_USER`（默认 `admin`）、`APP_ID`（默认 `1`）。

## 常用路径

| 用途 | 路径 |
| --- | --- |
| 容器内注册脚本 | `/app/register` |
| Python 入口 | `/app/register/runner.py` |
| 容器内数据目录 | `/data` |
| 宿主机数据目录 | `docker/data` |
| SSO 输出 | `docker/data/sso` |

## 本地开发

```bash
npm install
npm run server:build
npm run server:dev
```

本地运行 Python 注册入口：

```bash
python -m pip install -r register/requirements.txt
python register/runner.py --count 1
```

## 注意事项

- 不要提交 `.env`、`docker/.env`、`docker/data/`、SSO token、邮件凭据或代理密钥。
- 如果部署到公网，请设置强密码并限制访问来源。
- 使用前请确认目标平台条款和所在地法律法规。

## 致谢

- 感谢 [ReinerBRO/grok-register](https://github.com/ReinerBRO/grok-register)，本项目的自动化注册思路和 Python 注册流程受其启发。
- 感谢 [dreamhunter2333/cloudflare_temp_email](https://github.com/dreamhunter2333/cloudflare_temp_email)，本项目默认面向该邮件后端适配验证码读取流程。

## 开源协议

本项目基于 [MIT 开源协议](LICENSE) 开源。
