# Dirty Machine Test Environment

Test fixtures for [vibe-sec](https://github.com/kobzevvv/vibe-sec) — simulates a developer's messy/insecure setup to trigger ALL static security checks and produce a comprehensive report.

**All secrets in this repo are FAKE.** They use prefixes like `fake`, `test`, `FAKE` and are obviously synthetic. Values include `_TESTFIXTURE` markers to prevent false alerts from GitHub Push Protection and GitGuardian.

## Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/kobzevvv/vibe-sec-dirty-machine.git
cd vibe-sec-dirty-machine

# 2. Set up the dirty environment (installs fixtures to ~/)
./setup-macos.sh

# 3. Run the vibe-sec scanner (install vibe-sec first: npm i -g vibe-sec)
npx vibe-sec --static-only

# 4. View the report
open vibe-sec-log-report-*.html

# 5. Clean up (restores all backups)
./teardown.sh
```

Windows:
```powershell
.\setup-windows.ps1
npx vibe-sec --static-only
.\teardown.ps1
```

## What Each Fixture Triggers

| Fixture File | Scanner Check | What It Tests |
|---|---|---|
| `claude-settings.json` | `checkClaudeSettings` | `skipDangerousModePermissionPrompt: true`, MCP tokens in plaintext (`WEBFLOW_TOKEN`, `GITHUB_TOKEN`), MCP servers using `@latest` (unpinned versions) |
| `claude-history.jsonl` | `checkPromptInjectionSigns` | Prompt injection patterns: "ignore previous instructions", exfil commands (`curl \| bash`), `cat ~/.ssh/id_rsa`, `[system]` tags, `base64 -d` |
| `CLAUDE.md` | `checkClaudeMdHardening` | CLAUDE.md file with no prompt injection protection rules |
| `zsh_history` | `checkShellHistorySecrets` | Leaked secrets: `OPENAI_API_KEY=sk-proj-...`, GitHub PAT, AWS keys, Anthropic key, Stripe key, Slack token, GitLab PAT, Neon key |
| `git-repo-with-env/` | `checkGitSecurity` | Git repo with `.env` tracked (containing DB URL, Stripe key, OpenAI key, Supabase service role key, AWS keys) |
| `fake-sa-key.json` | `checkCliTokenFiles` | Google Service Account key JSON with `"type": "service_account"` and `"private_key"` |
| `fly-config.yml` | `checkCliTokenFiles` | Fly.io CLI config with `access_token` in plaintext |
| `paste-cache/` | `checkPasteAndSnapshots` | 11 paste files (>10 triggers the check), 6 contain secret patterns (`API_KEY=`, `sk-`, `AKIA`, `ghp_`, `SECRET_KEY=`, `PASSWORD=`) |
| `shell-snapshots/` | `checkPasteAndSnapshots` | Shell snapshot with `SECRET_KEY` and `DATABASE_URL` in environment |
| `clawdbot-config.json` | `checkClawdbot` | Telegram bot token and gateway auth token in plaintext config |
| `ssh-key-no-passphrase` | `checkSshKeysSecurity` | RSA private key WITHOUT `ENCRYPTED` header (unprotected), also triggers legacy RSA format warning |
| `ssh-key-with-passphrase` | `checkSshKeysSecurity` | SSH key WITH `ENCRYPTED` header (passes the check, used for comparison) |
| `git-credentials` | `checkGitCredentialStore` | Plaintext credentials: `https://user:password@github.com` format, 3 entries |
| `docker-compose.yml` | `checkDockerComposeExposure` | Ports exposed on 0.0.0.0: postgres (5432), redis (6379), app (3000), nginx (8080) |
| `terraform.tfstate` | `checkTerraformState` | Terraform state with `"password"`, `"aws_secret_access_key"`, `"secret"` in resource attributes |

## Checks That Cannot Be Fixtured

These checks examine live system state and cannot be triggered by static fixtures:

| Check | Why | Notes |
|---|---|---|
| `checkOpenPorts` | Runs `lsof -iTCP` to find listening processes | Only triggers if you have dev servers on 0.0.0.0 |
| `checkFirewall` | Reads macOS firewall state (`com.apple.alf`) | System-level setting |
| `checkNgrokTunnels` | Checks if ngrok is running via `pgrep` + API | Requires live ngrok process |
| `checkScreenLock` | Reads macOS screensaver settings | System-level setting |
| `checkHomebrewOutdated` | Runs `brew outdated --json` | Depends on installed packages |
| `checkOperationalSafety` | Checks Time Machine, root Claude, multiple instances | System-level state |
| `checkMcpToolSecurity` | Spawns MCP servers and reads tool descriptions | Requires actual MCP packages installed |

## Directory Structure

```
vibe-sec-dirty-machine/
  fixtures/                   # All fake insecure files
    claude-settings.json      # dangerousMode + MCP tokens + @latest
    claude-history.jsonl      # prompt injection indicators
    CLAUDE.md                 # no injection protection
    zsh_history               # leaked API keys
    git-repo-with-env/        # .env with secrets
    fake-sa-key.json          # Google service account key
    fly-config.yml            # Fly.io token
    paste-cache/              # 11 paste files (6 with secrets)
    shell-snapshots/          # env with secrets
    clawdbot-config.json      # Telegram + gateway tokens
    ssh-key-no-passphrase     # unencrypted RSA key
    ssh-key-with-passphrase   # encrypted SSH key (passes check)
    git-credentials           # plaintext git creds
    docker-compose.yml        # ports on 0.0.0.0
    terraform.tfstate         # state with secrets
  setup-macos.sh              # Sets up dirty env on macOS
  setup-windows.ps1           # Sets up dirty env on Windows
  teardown.sh                 # Cleanup (macOS/Linux)
  teardown.ps1                # Cleanup (Windows)
  README.md                   # This file
```

## How to Add New Test Cases

1. Create a new fixture file in `fixtures/`
2. Add installation logic to both `setup-macos.sh` and `setup-windows.ps1`
3. Add removal logic to both `teardown.sh` and `teardown.ps1`
4. Update the table in this README
5. Ensure all secrets are obviously fake (prefix with `fake`, `test`, `FAKE`)

## Safety Notes

- All secrets use `fake`, `test`, `FAKE` prefixes and are obviously synthetic
- Values include `_TESTFIXTURE` markers that prevent GitHub Push Protection and GitGuardian from flagging them
- The setup scripts back up any existing files before overwriting
- The teardown scripts restore backups and clean up all test artifacts
- The git repo fixture is created in `/tmp/` (or `$env:TEMP` on Windows)
- No real credentials are used anywhere in this test environment

## Related Projects

| Project | Platform | Description |
|---------|----------|-------------|
| [vibe-sec](https://github.com/kobzevvv/vibe-sec) | All | Security scanner CLI (main project) |
| [vibe-sec-app](https://github.com/kobzevvv/vibe-sec-app) | macOS | Menubar app |
| [vibe-sec-app-win](https://github.com/kobzevvv/vibe-sec-app-win) | Windows | System tray app |
