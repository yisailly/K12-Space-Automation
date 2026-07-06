# K12 Space Automation

[中文](README.md) | English

K12 Space Automation is a local K12 workspace automation console. It brings mailbox pools, email OTP, K12 workspace join/switch flows, Sub2API imports, access-token liveness/repair, automatic refills, account JSON export, and task logs into one web console.

This repository contains source code, documentation, configuration templates, and lock files only. It does not include real runtime configuration, tokens, cookies, mailbox refresh tokens, account JSON files, or task data by default. Do not commit real credentials or local runtime data, regardless of repository visibility.

## Features

- Mailbox pool management: batch import, automatic OTP, manual OTP, mailbox selection, batch deletion, status marking, plus-alias splitting, and failed-mailbox reuse control.
- Dynamic Gmail OTP: SMSBower Gmail and Emailnator Gmail. SMSBower mode can show balance/local spend and supports service code, domain, max price, and Gmail fission child tasks.
- K12 workspace flow: request/accept modes and sequential multi-workspace execution. One mailbox login can be reused to finish K12, Sub2API import, and JSON output for multiple workspaces.
- Sub2API import: OAuth import and noRT import, with multi-group binding, proxy/IP management fields, account priority, and import concurrency.
- Access-token maintenance: task-level and mailbox-level AT liveness checks, batch checks, inactive-task selection, AT copy, and repair-task creation after 401/inactive results.
- Automatic refill: counts normal accounts in a target Sub2API group, creates refill tasks below a threshold, and supports manual trigger, scheduled checks, deep liveness checks, and refill history logs.
- JSON output: SUB2API and CPA account JSON formats with configurable output directory. Account JSON can be written from task results even when Sub2API import is disabled.
- Data migration: export/import local configuration, mailbox pool, tasks, and token bundles. Current data is backed up automatically before import.
- Task management: batch start, cancel, delete, retry, clear failed tasks, pagination, log modal, rate-limit cooldown, and retry workflow after failures.

## Core Capabilities

### Mailbox Pool and OTP

Mailboxes can come from a fixed mailbox pool or a dynamic Gmail provider. Fixed pools are useful when you already have mailbox resources. Dynamic Gmail is useful when you do not want to maintain a long-lived pool. Manual OTP mode only requires mailbox addresses; when a task waits for OTP, enter the six-digit code in the task log modal.

### Multi-Workspace K12 Flow

`workspaceIds` supports one workspace per line or comma-separated values. With multiple workspaces, the system reuses the same login context where possible, then runs join/switch, import, and JSON output sequentially instead of logging in separately for every workspace.

### Sub2API and noRT

The default mode uses Sub2API OAuth import. noRT mode skips the Sub2API OAuth chain; after registration/login and K12 context switching, it uses the K12-context AT to create or update noRT accounts. The group field supports multiple groups for publishing the same account to several Sub2API groups.

### AT Liveness and Repair

Both task lists and mailbox pools can run AT liveness checks. When an account is inactive or returns 401, repair tasks can re-login through email OTP and update the matching Sub2API account. If no matching Sub2API account exists, the repair flow can create one with the current configuration.

### Automatic Refill

Automatic refill periodically checks the normal account count in a target Sub2API group. When the count is below the configured threshold, the system creates refill tasks from free mailboxes. With deep liveness enabled, only accounts that pass a real request are counted as normal.

### Dynamic Gmail and Fission

SMSBower Gmail mode can rent Gmail mailboxes for OTP. After a parent Gmail task succeeds, it can create configured `+alias` child tasks. Child tasks run sequentially to reduce OTP cross-use risk. Emailnator mode supports plusGmail, googleMail, dotGmail, and domain generation types.

### Data Backup and Task Recovery

The console can export a local data bundle covering configuration, mailbox pool, tasks, and tokens. Before importing a bundle, current data is backed up to `data/backups/`. After server restarts, unfinished tasks are marked failed or kept queued, making retry and investigation easier.

## Architecture

- `src/`: Vue 3 web console entry and UI logic.
- `server/index.ts`: local HTTP API server for task scheduling, configuration persistence, mailbox state, K12 flows, Sub2API calls, and JSON output.
- `codex_register/`: lower-level automation toolkit for registration, OAuth, mailboxes, SMS, Sentinel, Sub2API, CPA, and standalone web tools.
- `codex_register/config.example.json`: committable configuration template. Copy it to `codex_register/config.json` before filling real values.
- `public/`, `index.html`, `vite.config.ts`: Vite frontend assets and build configuration.
- `data/`, `json/`, `pool_tokens.txt`, `config.json`: runtime data and local configuration. These paths are ignored and are not part of the repository payload.

## Requirements

- Node.js 20+, Node.js 22+ recommended.
- npm 10+.
- Network access to the services you configure.
- Optional HTTP or SOCKS proxy.

## Install and Run

This section is written for beginners using the public repository. The goal is to get the project running first. Use `local run` for trying or debugging on your own computer. Use `VPS deployment` for long-running online usage, then add PM2 and Nginx only if needed.

### 1. Choose a Run Mode First

| Scenario | Recommended command | URL |
| --- | --- | --- |
| Local trial or development | `npm run dev` | `http://127.0.0.1:5174/` |
| Local normal run | `npm run build`, then `npm run start` | `http://127.0.0.1:8796/` |
| VPS deployment | Verify with `npm run build` and `npm run start`, then use PM2 | `http://SERVER_IP:8796/` or a domain |

Notes:

- `npm run dev` starts both the API server and the Vite frontend server. Use it for development and debugging.
- `npm run start` starts only the API server and serves the built `dist/` frontend. Use it for normal runs and deployments.
- Run `npm run build` before normal or production usage.

### 2. Requirements

Prepare these first:

- Node.js 20+, Node.js 22+ recommended.
- npm 10+.
- Git.
- A modern browser, such as Chrome, Edge, Firefox, or Safari.

Check versions:

```bash
node -v
npm -v
git --version
```

If `node`, `npm`, or `git` is missing, install it for your operating system first.

### 3. Windows from Scratch

#### 3.1 Install Node.js and Git

Beginner-friendly GUI path:

1. Open the Node.js download page: <https://nodejs.org/en/download>
2. Download and install the LTS version.
3. Open the Git download page: <https://git-scm.com/downloads/win>
4. Download and install Git for Windows.
5. Close the current terminal and open a new PowerShell window.

If you already use `winget`, you can install them with:

```powershell
winget install OpenJS.NodeJS.LTS
winget install Git.Git
```

Check versions:

```powershell
node -v
npm -v
git --version
```

#### 3.2 Clone and Install Dependencies

```powershell
cd $HOME\Desktop
git clone https://github.com/BFanSYe/K12-Space-Automation.git
cd K12-Space-Automation
npm install
```

#### 3.3 Start the App

Development mode:

```powershell
npm run dev
```

Open:

```text
http://127.0.0.1:5174/
```

Normal run:

```powershell
npm run build
npm run start
```

Open:

```text
http://127.0.0.1:8796/
```

### 4. macOS from Scratch

#### 4.1 Install Node.js and Git

If Homebrew is already installed, use:

```bash
brew install node git
```

If you do not use Homebrew, use the GUI path:

1. Open the Node.js download page: <https://nodejs.org/en/download>
2. Download and install the LTS version.
3. Git is usually installed with Xcode Command Line Tools. If `git --version` is missing, run `xcode-select --install`.

Check versions:

```bash
node -v
npm -v
git --version
```

#### 4.2 Clone and Install Dependencies

```bash
cd ~/Desktop
git clone https://github.com/BFanSYe/K12-Space-Automation.git
cd K12-Space-Automation
npm install
```

#### 4.3 Start the App

Development mode:

```bash
npm run dev
```

Open `http://127.0.0.1:5174/`.

Normal run:

```bash
npm run build
npm run start
```

Open `http://127.0.0.1:8796/`.

### 5. Ubuntu/Linux from Scratch

This section works for both local Linux and Ubuntu VPS. If you are deploying on a VPS, SSH into the server first.

Install basic tools:

```bash
sudo apt update
sudo apt install -y git curl ca-certificates
```

Install Node.js 22 and npm:

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x -o /tmp/nodesource_setup.sh
sudo -E bash /tmp/nodesource_setup.sh
sudo apt install -y nodejs
```

Check versions:

```bash
node -v
npm -v
git --version
```

Clone and install dependencies:

```bash
mkdir -p ~/apps
cd ~/apps
git clone https://github.com/BFanSYe/K12-Space-Automation.git
cd K12-Space-Automation
npm install
```

Development mode:

```bash
npm run dev
```

Open `http://127.0.0.1:5174/` on local Linux.

Normal run or VPS foreground smoke test:

```bash
npm run build
npm run start
```

Open:

- Local Linux: `http://127.0.0.1:8796/`
- VPS: `http://SERVER_IP:8796/`

If the VPS page is unreachable, check the service locally first:

```bash
curl http://127.0.0.1:8796/api/health
```

Then confirm that the cloud security group and system firewall allow TCP `8796`.

### 6. First Configuration

After opening the web console for the first time, go to Settings and fill in the local runtime configuration, such as proxy, workspace, Sub2API, OTP, and JSON output directory.

Saved configuration creates local files:

- `data/config.json`
- `config.json`

For standalone tools under `codex_register/`, copy the template only when needed:

```bash
cp codex_register/config.example.json codex_register/config.json
```

Do not commit real configuration or runtime data, including tokens, cookies, mailbox refresh tokens, account JSON files, `config.json`, `data/`, `json/`, or `pool_tokens.txt`.

### 7. Optional: Keep a VPS Running with PM2

Skip this section for temporary local usage. PM2 is recommended for long-running VPS deployments.

Install PM2:

```bash
sudo npm install -g pm2
```

Start from the project root:

```bash
cd ~/apps/K12-Space-Automation
pm2 start npm --name k12-space-automation -- run start
```

Common commands:

```bash
pm2 status
pm2 logs k12-space-automation
pm2 restart k12-space-automation
pm2 stop k12-space-automation
```

Enable startup on boot:

```bash
pm2 save
pm2 startup
```

`pm2 startup` prints a `sudo env ...` command. Copy and run that command once.

After updating code:

```bash
cd ~/apps/K12-Space-Automation
git pull
npm install
npm run build
pm2 restart k12-space-automation
```

### 8. Optional: Ports, Firewall, and Domain Access

Default ports:

- `8796`: API server and production web page.
- `5174`: Vite development web page, only used by `npm run dev`.

Change the API port temporarily:

```bash
PORT=8899 npm run start
```

Change the API port with PM2:

```bash
PORT=8899 pm2 start npm --name k12-space-automation -- run start
```

Allow the port on Ubuntu firewall:

```bash
sudo ufw allow 8796/tcp
sudo ufw status
```

For domain access, use Nginx to proxy to local port `8796`. Replace `example.com` with your domain:

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

After HTTP works, add HTTPS with certbot or your existing certificate setup.

### 9. Startup Troubleshooting

- `git: command not found`: Git is not installed. Install Git and reopen the terminal.
- `node` or `npm` is missing: Node.js is not installed, or the terminal was opened before installation finished. Reopen the terminal.
- Windows PowerShell cannot run npm: reopen PowerShell first. If it still fails, use Command Prompt or Git Bash.
- `npm install` fails: confirm `node -v` is 20 or newer, check your network, and retry.
- Page unreachable: `npm run dev` uses `5174`; `npm run start` uses `8796`. Check that you are opening the correct port.
- 404 or blank page: normal run mode requires `npm run build` first.
- Port already in use: stop the old process or set another `PORT`.
- Configuration files appear after saving Settings: this is expected. `config.json`, `data/`, `json/`, and `pool_tokens.txt` are ignored by `.gitignore`; do not commit real data.

## Configuration

The main configuration is saved from the Settings page in the web console. Runtime writes:

- `data/config.json`: current console configuration.
- `config.json`: root-level compatibility configuration for legacy flows.

Standalone tools under `codex_register/` read `codex_register/config.json`. For first use:

```bash
cp codex_register/config.example.json codex_register/config.json
```

Common fields:

| Field | Description |
| --- | --- |
| `port` | API server port, default `8796`. |
| `defaultProxyUrl` | Proxy for OpenAI/Auth requests. Supports `direct`, HTTP, and SOCKS. |
| `openaiProxyUrls` | Rotating proxy list for OpenAI/Auth requests. |
| `mailApiBaseUrl` | Base URL for four-part mailbox OTP APIs. |
| `workspaceIds` | K12 workspace ID list. |
| `route` | K12 workspace route, `request` or `accept`. |
| `taskConcurrency` | Task concurrency. |
| `runWorkspaceJoin` | Whether to run the K12 join/switch flow. |
| `runSub2Api` | Whether to import accounts into Sub2API. |
| `sub2apiNoRtMode` | Whether to use noRT import mode. |
| `sub2apiUrl` | Sub2API service URL. |
| `sub2apiEmail` | Sub2API admin email. |
| `sub2apiPassword` | Sub2API admin password. |
| `sub2apiGroupName` | Target Sub2API group. |
| `sub2apiProxyName` | Sub2API proxy name. |
| `sub2apiAccountPriority` | Sub2API account priority. |
| `sub2apiConcurrency` | Sub2API import concurrency. |
| `sub2apiAutoRefillEnabled` | Enable automatic Sub2API refill. |
| `sub2apiRefillGroupName` | Group checked by automatic refill. |
| `sub2apiRefillThreshold` | Automatic refill threshold. |
| `sub2apiRefillEmailCount` | Mailbox count used for automatic refill. |
| `sub2apiRefillIntervalMs` | Automatic refill interval. |
| `sub2apiRefillDeepCheckEnabled` | Enable deep liveness checks for automatic refill. |
| `gmailMailProvider` | Dynamic Gmail provider, `smsbower` or `emailnator`. |
| `smsBowerMailEnabled` | Enable SMSBower Gmail OTP. |
| `smsBowerApiKey` | SMSBower API key. |
| `smsBowerMailBaseUrl` | SMSBower mail API URL. |
| `smsBowerMailService` | SMSBower mail service name. |
| `smsBowerMailDomain` | SMSBower mail domain. |
| `smsBowerMailMaxPrice` | Maximum SMSBower mail price. |
| `smsBowerGmailFissionEnabled` | Enable SMSBower Gmail fission child-mailbox tasks. |
| `smsBowerGmailFissionCount` | Fission count per Gmail mailbox. |
| `emailnatorBaseUrl` | Emailnator service URL. |
| `emailnatorEmailType` | Emailnator mailbox type. |
| `requireChatgptAccountId` | Require ChatGPT account ID in access tokens. |
| `tokenOut` | Access-token output file, default `pool_tokens.txt`. |
| `jsonOutDir` | Account JSON output directory, default `json/`. |
| `jsonOutFormat` | JSON output format, `sub2api` or `cpa`. |

## Task Flow

1. Open the web console, then save proxy, workspace, Sub2API, OTP, and JSON output settings from Settings.
2. Choose a mailbox source: import a fixed mailbox pool, use manual OTP, or enable SMSBower/Emailnator dynamic Gmail.
3. Configure K12 workspace IDs, request/accept mode, concurrency, Sub2API import, noRT, and JSON output.
4. If you need to maintain Sub2API group inventory, enable automatic refill and configure target group, threshold, refill count, and deep liveness.
5. Start tasks and inspect status, K12 results, Sub2API accounts, AT summaries, JSON files, and logs in the task list.
6. For failed or inactive accounts, run AT liveness checks first, then repair AT, retry tasks, clear failed tasks, or lower concurrency before rerunning.
7. Use data export/import for local state migration or backup. Do not use Git for runtime data.

## Sensitive File Boundary

The following files or directories may contain passwords, API keys, mailbox refresh tokens, access tokens, cookies, OAuth data, mailbox pools, account JSON files, or task logs. They should not be committed, regardless of repository visibility:

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

Before committing, check:

```bash
git status --short --ignored
git ls-files | rg '(^|/)(data|json|auth|pool_tokens|config\.json|k12-basic-auth|\.env|.*\.pem|.*\.key)'
```

The second command should produce no output. `codex_register/config.example.json` is a template and may be committed.

## FAQ

### `EmailOtpValidate wrong_email_otp_code`

OpenAI rejected the submitted email OTP. Common causes are stale mailbox messages, ad emails containing six-digit numbers, or expired OTPs. Use another mailbox, clean old messages, or verify with manual OTP mode.

### Redirected to `accounts.google.com`

The mailbox was routed to Google OAuth instead of the normal email OTP flow. This tool does not automate Google account login. Use a mailbox that can proceed through email OTP.

### `CreateAccount HTTP 500 Request timeout`

This is usually caused by upstream instability, a slow proxy, request timeout, or high concurrency. Retry, change proxy, or lower concurrency.

### Cancel does not stop instantly

Tasks stop at the next cancellable boundary. If a network request is in progress, status updates may wait until that request returns or times out.

### Sub2API import fails

Verify `sub2apiUrl`, `sub2apiEmail`, `sub2apiPassword`, `sub2apiGroupName`, and proxy settings first. Then inspect the HTTP status and response summary in task logs.

### JSON output is missing

Verify `jsonOutDir`, `jsonOutFormat`, access-token availability, and write permission for the target directory.

## Build Verification

Run before committing:

```bash
npm run build
npx tsc --noEmit -p codex_register/tsconfig.json
git status --short --ignored
git ls-files | rg '(^|/)(data|json|auth|pool_tokens|config\.json|k12-basic-auth|\.env|.*\.pem|.*\.key)'
```

`npm run build` and `npx tsc` should pass. The sensitive-file check should produce no output.

## License

This project is licensed under the MIT License. See `LICENSE`.
