# K12 Space Automation

[中文](README.md) | English

K12 Space Automation is a local K12 workspace automation console for mailbox pool management, email OTP flows, K12 workspace join/switch tasks, Sub2API imports, access-token checks/repairs, and account JSON export.

This repository contains source code, documentation, configuration templates, and lock files only. It does not include real runtime configuration, tokens, cookies, mailbox refresh tokens, account JSON files, or task data by default. Do not commit real credentials or local runtime data, regardless of repository visibility.

## Features

- Mailbox pool management: import, select, delete, status marking, and retry.
- OTP handling: mailbox URL, manual OTP, SMSBower Gmail, and Emailnator Gmail.
- K12 flow: login, join or switch K12 workspace, and read K12-context access tokens.
- Sub2API: OAuth import, noRT import, account liveness check, and access-token repair.
- JSON output: SUB2API and CPA account JSON formats.
- Data migration: import/export local configuration, mailbox pool, tasks, and token data packages.
- Task management: batch start, cancel, retry, clear failed tasks, pagination, status, and logs.

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

1. Open the web console.
2. Fill Settings and confirm proxy, workspace, Sub2API, and OTP settings.
3. Import a mailbox pool or enable dynamic Gmail OTP.
4. Configure task count, concurrency, K12 workspace flow, Sub2API/noRT, and JSON output.
5. Start tasks and inspect status, logs, access-token summaries, and output paths.
6. Retry failed tasks after reading logs, or lower concurrency before rerunning.
7. Use data import/export for local state migration. Do not use Git for runtime data.

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
