# grok-reg-tool

一个可自部署的 **Grok / GPT 双注册机 Web 控制台**，将复杂的浏览器自动化注册流程封装为可视化操作，支持 Docker 一键部署。

> 本项目与 xAI、Grok、OpenAI、ChatGPT 均无官方关联。请仅在合法、合规、获得许可的研究、学习或自托管实验环境中使用。

## 项目简介

`grok-reg-tool` 解决了「手动注册 Grok / ChatGPT 账号繁琐、易失败」的问题，提供一条完整的自动化链路：

**邮箱接码 → 浏览器自动注册 → 账号落盘 → SSO/AccessToken 输出 → 账号验活 → 支付链接提炼**

无论是自用注册，还是批量管理号池，都可以通过一个 Web 控制台完成，无需接触底层脚本。

## 核心特性

### 🤖 双注册机

| 注册机 | 目标 | 技术 | 产出 |
| --- | --- | --- | --- |
| Grok 注册机 | xAI / grok.com | DrissionPage + Chromium | SSO token |
| GPT 注册机 | ChatGPT / OpenAI | Playwright + Stealth | AccessToken（完整 session） |

登录页可一键切换，各自拥有独立的任务台、配置项与日志。

### 🌐 Web 控制台

- 实时查看注册进度、轮次、成功/失败统计与日志。
- 任务可随时启动、停止，支持多轮次运行。
- 首次登录强制修改默认账号密码，保障安全。

### ✉️ 邮箱接码

支持两种收信方式，可灵活组合：

- **Cloudflare 临时邮箱**：自动创建邮箱、轮询验证码，全自动。
- **Remail 人工邮箱**：支持 `outlook.com`、`hotmail.com`、`icloud.com` 等后缀，可批量下单、取件、取码。

### 💳 支付链接提炼

- 对接 `pay.153.ink`，一键提炼支付链接（支持 Hosted / PH / PayPal / PIX 等通道）。
- 免费账号也可提炼，返回 `cs_live_...` 结算链接。
- 支持手动粘贴、CDK 服务、HTTP API、浏览器驱动四种模式。

### ✨ Plus 试用检测

GPT 注册成功后自动检测 ChatGPT Plus 试用资格，使用浏览器内请求绕过 Cloudflare 拦截。

### 🗂️ 号池管理

- 本地保存账号、密码、SSO / AccessToken。
- 支持一键验活（存活/失效/额度信息）。
- 支持导出 SSO / 完整账号，可上传至 grok2api。

### 🌍 代理管理

- 手动代理（HTTP/HTTPS/SOCKS4/SOCKS5）。
- 机场订阅解析（Clash / V2Ray）。
- 两级延迟测试（本机→节点、节点→目标站）。

### 📦 Docker 部署

- 多阶段构建，内置 Chromium + Xvfb + VNC/noVNC（可远程查看浏览器画面，用于人工过验证）。
- 数据持久化到 `docker/data/`，重建容器不丢失。

### 🔄 在线更新

- 内置授权系统配置，开箱即用，一键检查、下载、应用更新。
- 更新包自动校验 SHA256，支持宿主机脚本一键重建。

## 技术架构

| 层 | 技术 |
| --- | --- |
| 后端 | Node.js 20 + Express + TypeScript + WebSocket |
| 前端 | Vue 3 + Pinia + Arco Design |
| 自动化 | Python 3.13 + DrissionPage + Playwright |
| 部署 | Docker + Docker Compose |
| 存储 | 文件系统 JSON（`docker/data/`） |

## 快速部署

### 1. 获取代码

```bash
git clone https://github.com/FengZi1221/grok-reg-tool.git
cd grok-reg-tool/docker
cp .env.example .env
```

### 2. 配置环境变量

编辑 `docker/.env`：

```env
WEB_PORT=6657          # Web 控制台端口
RUN_COUNT=10           # 默认注册轮数

MAIL_API_BASE=         # Cloudflare 邮箱后端地址
MAIL_ADMIN_AUTH=       # 邮箱后端 admin 密码
MAIL_DOMAIN=           # 可收信的邮箱域名

HTTP_PROXY=            # 出站 HTTP 代理（可选）
BROWSER_PROXY=         # 浏览器代理（可选）
COOKIE_SECURE=         # HTTPS 反向代理时设为 1
```

### 3. 启动

```bash
docker compose up -d --build
```

首次构建需安装 Node/Python/Chromium 依赖，约 5~15 分钟。

### 4. 访问

```text
http://你的服务器IP:6657
```

初始账号密码通过日志查看：

```bash
docker logs grok-reg-tool
```

## 使用流程

1. **登录**：首次使用 `admin/admin`，登录后强制修改密码。
2. **配置邮箱**：在「配置」页填写 Cloudflare 邮箱后端，或配置 Remail API Key。
3. **开始注册**：进入「Grok 注册机」或「GPT 注册机」，点击「开始注册」，实时查看日志。
4. **号池查看**：注册成功后，在「号池」页查看、验活、导出账号。
5. **支付提炼**：GPT 注册机页面点击「提炼支付链接」，自动提炼出支付链接。

## 在线更新

本项目通过「新云Auth Studio」授权系统的 Open API 实现在线更新。

### 内置配置

在线更新所需的**授权系统地址、Open API Key、应用 ID、版本 code 均已内置**，全新安装后无需手动填写。打开「更新」页面点击「检查更新」即可。

### 应用更新（宿主机执行）

```bash
cd /root/grok-reg-tool
./apply-update.sh 0.1.8
```

脚本会解压更新包、同步项目文件（保留 `docker/data` 与 `docker/.env`）、重新构建并重建容器。

> 更新包下载目录：`docker/data/updates/`（容器内 `/data/updates/`）。

### 一键发布更新包（开发者）

```bash
cd /root/grok-reg-tool
AUTH_PASS='<授权系统管理员密码>' ./release-update.sh 0.1.9 19 '【优化】
1. ...'
```

- 第一个参数：版本名，如 `0.1.9`
- 第二个参数：版本 code（主版本×100 + 次版本×10 + 修订）
- 第三个参数：更新日志文本

可选环境变量：`AUTH_SERVER`（默认 `https://power.1tim.com`）、`AUTH_USER`（默认 `admin`）、`APP_ID`。

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

- 不要提交 `.env`、`docker/.env`、`docker/data/`、SSO / AccessToken、邮件凭据或代理密钥。
- 部署到公网时，请设置强密码并限制访问来源。
- 自动化注册可能违反目标平台条款，使用前请确认所在地法律法规。

## 致谢

- 感谢 [ReinerBRO/grok-register](https://github.com/ReinerBRO/grok-register)，本项目自动化注册思路受其启发。
- 感谢 [dreamhunter2333/cloudflare_temp_email](https://github.com/dreamhunter2333/cloudflare_temp_email)，本项目默认适配其验证码读取流程。

## 开源协议

本项目基于 [MIT 开源协议](LICENSE) 开源。
