# K12 Space Automation

中文 | [English](README.en.md)

K12 Space Automation 是一个本地运行的 K12 workspace 自动化控制台. 它把邮箱池管理, 邮箱验证码, K12 workspace 加入/切换, Sub2API 入库, access token 测活/修复, 自动补号, 账号 JSON 写出和任务日志集中到一个 Web 控制台里.

本仓库只包含源码, 文档, 配置模板和锁文件. 默认不包含真实运行配置, token, cookie, mailbox refresh token, account JSON 或任务数据. 无论仓库是否公开, 都不要提交任何真实账号凭据或本地运行数据.

## 功能概览

- 邮箱池管理: 支持批量导入, 自动接码, 手动接码, 选择任务邮箱, 批量删除, 状态标记, plus alias 分裂和失败邮箱复用控制.
- 动态 Gmail 接码: 支持 SMSBower Gmail 和 Emailnator Gmail. SMSBower 模式可查看余额和本地花费, 可配置服务码, 域名, 最高价格和 Gmail fission 子任务.
- K12 workspace 流程: 支持 request/accept 两种动作, 支持多个 workspace 顺序执行. 单个邮箱登录一次后可逐个完成 K12, Sub2API 入库和 JSON 写出.
- Sub2API 入库: 支持 OAuth 入库和 noRT 直入模式, 可配置多分组绑定, 代理/IP 管理字段, 账号优先级和入库并发.
- access token 维护: 支持任务级和邮箱级 AT 测活, 批量测活, 一键筛选失活任务, 复制 AT, 401 失活后创建修复任务.
- 自动补号: 支持按 Sub2API 分组统计正常账号数, 低于阈值时自动创建补号任务, 支持手动触发, 定时检测, 深度测活和补号历史日志.
- JSON 写出: 支持 SUB2API 和 CPA 两类账号 JSON 格式, 可配置输出目录. 即使不执行 Sub2API 入库, 也可以按任务结果写出账号 JSON.
- 数据迁移: 支持导出/导入本地配置, 邮箱池, 任务和 token 数据包. 导入前会自动备份当前数据.
- 任务管理: 支持批量启动, 取消, 删除, 重试, 清理失败任务, 分页查看, 日志弹窗, 限流冷却和失败后重试闭环.

## 核心能力详解

### 邮箱池与接码

邮箱可以从固定邮箱池导入, 也可以使用动态 Gmail 模式临时生成. 固定邮箱池适合已有邮箱资源的场景, 动态 Gmail 适合不想长期维护邮箱池的场景. 手动接码模式只要求导入邮箱, 任务等待验证码时在日志弹窗里填写 6 位验证码.

### 多 workspace K12 流程

`workspaceIds` 支持一行一个或逗号分隔. 多个 workspace 时, 系统会尽量复用同一次登录上下文, 依次执行加入/切换, 入库和 JSON 写出, 避免每个 workspace 都重复登录.

### Sub2API 和 noRT

默认模式会走 Sub2API OAuth 入库. noRT 直入模式会跳过 Sub2API OAuth 链路, 在注册/登录并切到 K12 后, 用 K12 上下文 AT 创建或更新 noRT 账号. 分组字段支持多个分组, 适合同时投放到多个 Sub2API 分组.

### AT 测活与修复

任务列表和邮箱池都可以发起 AT 测活. 当账号失活或接口返回 401 时, 可以创建 AT 修复任务, 重新走邮箱接码登录并更新对应 Sub2API 账号. 如果 Sub2API 中没有对应账号, 修复流程会按当前配置创建新账号.

### 自动补号

自动补号会定时检查目标 Sub2API 分组的正常账号数量. 当正常账号数低于预警线时, 系统从空闲邮箱池创建补号任务. 开启深度测活后, 会对账号执行真实请求, 只有真实可用的账号才计入正常数量.

### 动态 Gmail 和 fission

SMSBower Gmail 模式可以租用 Gmail 接码, 成功后可按配置创建 `+alias` 子任务. 子任务会逐个执行, 用来降低验证码串号风险. Emailnator 模式支持 plusGmail, googleMail, dotGmail 和 domain 等生成类型.

### 数据备份与任务恢复

控制台可以导出本地数据包, 覆盖配置, 邮箱池, 任务和 token. 导入数据包前会自动备份当前数据到 `data/backups/`. 服务重启后, 未完成任务会被标记为失败或保留队列状态, 便于继续重试和排查.

## 架构组成

- `src/`: Vue 3 Web 控制台入口和页面逻辑.
- `server/index.ts`: 本地 HTTP API 服务, 负责任务调度, 配置读写, 邮箱池状态, K12 流程, Sub2API 调用和 JSON 写出.
- `codex_register/`: 底层注册, OAuth, 邮箱, SMS, Sentinel, Sub2API, CPA 等自动化能力与独立 Web 工具.
- `codex_register/config.example.json`: 可提交的配置模板. 复制为 `codex_register/config.json` 后再填写真实值.
- `public/`, `index.html`, `vite.config.ts`: Vite 前端资源和构建配置.
- `data/`, `json/`, `pool_tokens.txt`, `config.json`: 运行时生成或本地保存的数据, 默认被忽略, 不属于仓库交付内容.

## 环境要求

- Node.js 20+, 建议 Node.js 22+.
- npm 10+.
- 可访问所配置服务的网络环境.
- 如需代理, 自行准备 HTTP 或 SOCKS 代理.

## 安装启动

本节面向公开仓库的小白用户, 目标是先把项目跑起来. 如果你只是想在自己电脑上体验或调试, 走 `本机运行`. 如果你想长期在线使用, 走 `VPS 部署`, 跑通后再看 PM2 和 Nginx 可选章节.

### 1. 先选运行方式

| 场景 | 推荐方式 | 访问地址 |
| --- | --- | --- |
| 本机体验或开发调试 | `npm run dev` | `http://127.0.0.1:5174/` |
| 本机普通运行 | `npm run build` 后 `npm run start` | `http://127.0.0.1:8796/` |
| VPS 长期部署 | 先 `npm run build` 和 `npm run start` 验证, 再用 PM2 后台运行 | `http://服务器IP:8796/` 或域名 |

说明:

- `npm run dev` 会同时启动 API 服务和 Vite 前端服务, 适合调试.
- `npm run start` 只启动 API 服务, 并直接托管 `dist/` 前端产物, 更适合普通运行或部署.
- 生产/普通运行前必须先执行 `npm run build`.

### 2. 环境要求

你需要先准备:

- Node.js 20+, 推荐 Node.js 22+.
- npm 10+.
- Git.
- 一个现代浏览器, 例如 Chrome, Edge, Firefox 或 Safari.

检查命令:

```bash
node -v
npm -v
git --version
```

如果 `node`, `npm` 或 `git` 提示找不到命令, 先按下面对应系统安装.

### 3. Windows 从零运行

#### 3.1 安装 Node.js 和 Git

新手推荐图形安装:

1. 打开 Node.js 下载页: <https://nodejs.org/en/download>
2. 下载并安装 LTS 版本.
3. 打开 Git 下载页: <https://git-scm.com/downloads/win>
4. 下载并安装 Git for Windows.
5. 安装完成后, 关闭当前终端, 重新打开 PowerShell.

如果你熟悉 `winget`, 也可以用命令安装:

```powershell
winget install OpenJS.NodeJS.LTS
winget install Git.Git
```

检查版本:

```powershell
node -v
npm -v
git --version
```

#### 3.2 下载项目并安装依赖

```powershell
cd $HOME\Desktop
git clone https://github.com/BFanSYe/K12-Space-Automation.git
cd K12-Space-Automation
npm install
```

#### 3.3 启动方式

开发调试:

```powershell
npm run dev
```

然后打开:

```text
http://127.0.0.1:5174/
```

普通运行:

```powershell
npm run build
npm run start
```

然后打开:

```text
http://127.0.0.1:8796/
```

### 4. macOS 从零运行

#### 4.1 安装 Node.js 和 Git

如果你已经安装 Homebrew, 推荐:

```bash
brew install node git
```

如果没有 Homebrew, 可以使用图形安装:

1. 打开 Node.js 下载页: <https://nodejs.org/en/download>
2. 下载并安装 LTS 版本.
3. Git 通常随 Xcode Command Line Tools 安装. 如果 `git --version` 不存在, 执行 `xcode-select --install`.

检查版本:

```bash
node -v
npm -v
git --version
```

#### 4.2 下载项目并安装依赖

```bash
cd ~/Desktop
git clone https://github.com/BFanSYe/K12-Space-Automation.git
cd K12-Space-Automation
npm install
```

#### 4.3 启动方式

开发调试:

```bash
npm run dev
```

然后打开 `http://127.0.0.1:5174/`.

普通运行:

```bash
npm run build
npm run start
```

然后打开 `http://127.0.0.1:8796/`.

### 5. Ubuntu/Linux 从零运行

本节同时适用于本机 Linux 和 Ubuntu VPS. 如果是 VPS, 先 SSH 登录服务器.

安装基础工具:

```bash
sudo apt update
sudo apt install -y git curl ca-certificates
```

安装 Node.js 22 和 npm:

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x -o /tmp/nodesource_setup.sh
sudo -E bash /tmp/nodesource_setup.sh
sudo apt install -y nodejs
```

检查版本:

```bash
node -v
npm -v
git --version
```

下载项目并安装依赖:

```bash
mkdir -p ~/apps
cd ~/apps
git clone https://github.com/BFanSYe/K12-Space-Automation.git
cd K12-Space-Automation
npm install
```

开发调试:

```bash
npm run dev
```

本机打开 `http://127.0.0.1:5174/`.

普通运行或 VPS 前台验证:

```bash
npm run build
npm run start
```

访问地址:

- 本机 Linux: `http://127.0.0.1:8796/`
- VPS: `http://服务器IP:8796/`

如果 VPS 访问不到, 先在服务器上检查服务是否正常:

```bash
curl http://127.0.0.1:8796/api/health
```

然后确认云服务器安全组和系统防火墙已放行 TCP `8796`.

### 6. 第一次打开后怎么配置

第一次打开 Web 控制台后, 进入 Settings 页面, 填写本地运行所需配置, 例如代理, workspace, Sub2API, 接码和 JSON 写出目录.

保存后会在本地生成:

- `data/config.json`
- `config.json`

如需使用 `codex_register/` 下的独立工具, 再复制模板:

```bash
cp codex_register/config.example.json codex_register/config.json
```

不要把真实配置提交到 Git, 包括 token, cookie, mailbox refresh token, account JSON, `config.json`, `data/`, `json/`, `pool_tokens.txt`.

### 7. 可选: VPS 后台运行 PM2

如果你只在本机临时使用, 可以跳过本节. VPS 长期运行推荐使用 PM2.

安装 PM2:

```bash
sudo npm install -g pm2
```

在项目根目录启动:

```bash
cd ~/apps/K12-Space-Automation
pm2 start npm --name k12-space-automation -- run start
```

常用命令:

```bash
pm2 status
pm2 logs k12-space-automation
pm2 restart k12-space-automation
pm2 stop k12-space-automation
```

开机自启:

```bash
pm2 save
pm2 startup
```

`pm2 startup` 会输出一条以 `sudo env ...` 开头的命令, 按输出复制执行一次即可.

更新代码后:

```bash
cd ~/apps/K12-Space-Automation
git pull
npm install
npm run build
pm2 restart k12-space-automation
```

### 8. 可选: 端口, 防火墙和域名访问

默认端口:

- `8796`: API 服务和生产页面.
- `5174`: Vite 开发页面, 只用于 `npm run dev`.

如需临时改 API 端口:

```bash
PORT=8899 npm run start
```

PM2 中改端口:

```bash
PORT=8899 pm2 start npm --name k12-space-automation -- run start
```

Ubuntu 防火墙放行端口:

```bash
sudo ufw allow 8796/tcp
sudo ufw status
```

如果需要域名访问, 可以用 Nginx 反向代理到本地 `8796`. 将 `example.com` 替换为你的域名:

```bash
sudo apt install -y nginx
sudo tee /etc/nginx/sites-available/k12-space-automation >/dev/null <<'NGINX'
server {
    listen 80;
    server_name example.com;

    client_max_body_size 50m;

    location / {
        proxy_pass http://127.0.0.1:8796;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
NGINX

sudo ln -sf /etc/nginx/sites-available/k12-space-automation /etc/nginx/sites-enabled/k12-space-automation
sudo nginx -t
sudo systemctl reload nginx
```

确认 HTTP 可访问后, 再按自己的证书方案或 certbot 配置 HTTPS.

### 9. 常见启动问题

- `git: command not found`: 没安装 Git, 先安装 Git 后重新打开终端.
- `node` 或 `npm` 找不到: 没安装 Node.js, 或安装后没有重新打开终端.
- Windows PowerShell 无法执行 npm: 先重开 PowerShell. 如果仍失败, 换 Command Prompt 或 Git Bash 执行同样命令.
- `npm install` 失败: 先确认 `node -v` 是 20 或更高, 再检查网络后重试.
- 页面打不开: `npm run dev` 对应 `5174`, `npm run start` 对应 `8796`, 先确认你访问的是正确端口.
- 页面 404 或空白: 普通运行模式需要先执行 `npm run build`.
- 端口占用: 停掉旧进程, 或设置 `PORT` 换端口.
- Settings 保存后出现配置文件: 这是正常行为. `config.json`, `data/`, `json/`, `pool_tokens.txt` 已被 `.gitignore` 忽略, 不要提交真实数据.

## 基础配置

主要配置通过 Web 控制台的 Settings 页面保存. 运行时会写入:

- `data/config.json`: 当前控制台配置.
- `config.json`: 兼容旧流程使用的根配置.

`codex_register/` 下的独立工具读取 `codex_register/config.json`. 初次使用时可以复制模板:

```bash
cp codex_register/config.example.json codex_register/config.json
```

常见配置项:

| 配置项 | 说明 |
| --- | --- |
| `port` | API 服务端口, 默认 `8796`. |
| `defaultProxyUrl` | OpenAI/Auth 请求代理, 支持 `direct`, HTTP, SOCKS. |
| `openaiProxyUrls` | 可轮换的 OpenAI/Auth 代理列表. |
| `mailApiBaseUrl` | 四段邮箱接码接口基础地址. |
| `workspaceIds` | K12 workspace ID 列表. |
| `route` | K12 workspace 路径, `request` 或 `accept`. |
| `taskConcurrency` | 任务并发数. |
| `runWorkspaceJoin` | 是否执行 K12 加入/切换流程. |
| `runSub2Api` | 是否执行 Sub2API 入库. |
| `sub2apiNoRtMode` | 是否使用 noRT 直入模式. |
| `sub2apiUrl` | Sub2API 服务地址. |
| `sub2apiEmail` | Sub2API 管理员账号. |
| `sub2apiPassword` | Sub2API 管理员密码. |
| `sub2apiGroupName` | Sub2API 目标分组. |
| `sub2apiProxyName` | Sub2API 代理名称. |
| `sub2apiAccountPriority` | Sub2API 账号优先级. |
| `sub2apiConcurrency` | Sub2API 入库并发数. |
| `sub2apiAutoRefillEnabled` | 是否开启 Sub2API 自动补货. |
| `sub2apiRefillGroupName` | 自动补货检查分组. |
| `sub2apiRefillThreshold` | 自动补货触发阈值. |
| `sub2apiRefillEmailCount` | 自动补货邮箱数量. |
| `sub2apiRefillIntervalMs` | 自动补货检查间隔. |
| `sub2apiRefillDeepCheckEnabled` | 是否开启自动补货深度测活. |
| `gmailMailProvider` | 动态 Gmail 渠道, `smsbower` 或 `emailnator`. |
| `smsBowerMailEnabled` | 是否开启 SMSBower Gmail 接码. |
| `smsBowerApiKey` | SMSBower API Key. |
| `smsBowerMailBaseUrl` | SMSBower 邮箱 API 地址. |
| `smsBowerMailService` | SMSBower 邮箱服务名. |
| `smsBowerMailDomain` | SMSBower 邮箱域名. |
| `smsBowerMailMaxPrice` | SMSBower 邮箱最高价格. |
| `smsBowerGmailFissionEnabled` | 是否开启 SMSBower Gmail fission 子邮箱任务. |
| `smsBowerGmailFissionCount` | 单个 Gmail fission 数量. |
| `emailnatorBaseUrl` | Emailnator 服务地址. |
| `emailnatorEmailType` | Emailnator 邮箱类型. |
| `requireChatgptAccountId` | 是否要求 access token 中存在 ChatGPT account ID. |
| `tokenOut` | access token 输出文件, 默认 `pool_tokens.txt`. |
| `jsonOutDir` | 账号 JSON 写出目录, 默认 `json/`. |
| `jsonOutFormat` | JSON 写出格式, `sub2api` 或 `cpa`. |

## 任务流程

1. 打开 Web 控制台, 先在 Settings 中保存代理, workspace, Sub2API, 接码和 JSON 写出配置.
2. 选择邮箱来源: 导入固定邮箱池, 选择手动接码, 或开启 SMSBower/Emailnator 动态 Gmail.
3. 配置 K12 workspace 列表, request/accept 动作, 并发, 是否执行 Sub2API 入库, 是否启用 noRT 和 JSON 写出.
4. 如果需要保持 Sub2API 分组库存, 开启自动补号, 设置目标分组, 预警线, 补号数量和深度测活.
5. 启动任务后, 在任务列表查看状态, K12 结果, Sub2API 账号, AT 摘要, JSON 文件和日志.
6. 对失败或失活账号, 先执行 AT 测活, 再按结果选择修复 AT, 重试任务, 清理失败任务或降低并发后重新运行.
7. 迁移或备份本地状态时, 使用导出/导入数据功能. 不要通过 Git 保存运行数据.

## 敏感文件边界

以下文件或目录可能包含密码, API Key, mailbox refresh token, access token, cookie, OAuth 数据, 邮箱池, 账号 JSON 或任务日志. 无论仓库是否公开都不应提交:

```text
config.json
codex_register/config.json
data/
json/
pool_tokens.txt
auth/
k12-basic-auth*
.env
.env.*
*.pem
*.key
*.crt
*.log
```

提交前检查:

```bash
git status --short --ignored
git ls-files | rg '(^|/)(data|json|auth|pool_tokens|config\.json|k12-basic-auth|\.env|.*\.pem|.*\.key)'
```

第二条命令应没有输出. `codex_register/config.example.json` 是模板文件, 可以提交.

## 常见问题

### `EmailOtpValidate wrong_email_otp_code`

OpenAI 判定提交的邮箱验证码错误. 常见原因是接码源返回旧邮件, 广告邮件中的 6 位数字, 或验证码已过期. 处理方式是更换邮箱, 清理接码源旧邮件, 或改用手动验证码确认.

### 停在 `accounts.google.com`

该邮箱被引导到 Google OAuth 登录, 不是普通邮箱验证码流程. 当前工具不会自动登录 Google 账号. 处理方式是更换可走邮箱验证码流程的邮箱.

### `CreateAccount HTTP 500 Request timeout`

通常是远端服务波动, 代理慢, 请求超时, 或并发过高. 处理方式是重试, 更换代理, 或降低并发.

### 取消任务后没有立刻停止

任务会在当前可取消边界尽快停止. 如果正在等待网络请求返回或超时, 状态更新会延后到该请求结束后.

### Sub2API 入库失败

先确认 `sub2apiUrl`, `sub2apiEmail`, `sub2apiPassword`, `sub2apiGroupName` 和代理配置有效. 再查看任务日志中的 HTTP 状态码和响应体摘要.

### JSON 没有写出

确认 `jsonOutDir`, `jsonOutFormat` 配置正确, 当前任务已拿到有效 access token, 且进程对目标目录有写权限.

## 构建验证

提交前执行:

```bash
npm run build
npx tsc --noEmit -p codex_register/tsconfig.json
git status --short --ignored
git ls-files | rg '(^|/)(data|json|auth|pool_tokens|config\.json|k12-basic-auth|\.env|.*\.pem|.*\.key)'
```

`npm run build` 和 `npx tsc` 应成功. 敏感文件检查命令应没有输出.

## 致谢

[LINUX DO](https://linux.do) — 社区交流与支持
[lxh77721/k12-reg](https://github.com/lxh77721/k12-reg) — 注册机原仓库

## License

This project is licensed under the MIT License. See `LICENSE`.
