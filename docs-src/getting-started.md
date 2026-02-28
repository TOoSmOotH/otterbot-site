# Getting Started

Get OtterBot up and running in minutes. Docker is the recommended approach — one command and you're live.

## Prerequisites

OtterBot has minimal requirements depending on how you run it:

### Docker (recommended)

- **Docker** 20.10+ (or Docker Desktop)
- At least **2 GB RAM** available for the container
- An **LLM API key** (Anthropic, OpenAI, or any OpenAI-compatible provider)

### From Source

- **Node.js** 20+ and **pnpm** 9+
- **Git**
- An **LLM API key**
- Optional: **Playwright** browsers (for web automation features)

## Quick Start with Docker

### Step 1 — Pull the image

```bash
$ docker pull ghcr.io/toosmooth/otterbot:latest
```

### Step 2 — Run the container

```bash
$ docker run -d -p 62626:62626 \
  --name otterbot \
  -e OTTERBOT_DB_KEY=change-me-to-something-secret \
  --shm-size 256m \
  ghcr.io/toosmooth/otterbot:latest
```

**Want persistent data?** Add `-v ~/otterbot:/otterbot` to keep installed packages, projects, and settings between container restarts. Without it, OtterBot runs fully ephemeral — everything resets when the container is removed. See [Volume & Persistent Data](#volume-persistent-data) for details.

### Step 3 — Open the UI

Navigate to `http://localhost:62626` in your browser. The first-run setup wizard will guide you through configuring your LLM provider and API key.

!!! warning "Important"
    Set `OTTERBOT_DB_KEY` to a unique secret. This encrypts your database (which stores API keys and settings). Losing this key means losing access to stored data.

!!! info "Accessing remotely?"
    If you're connecting from a different origin (e.g. behind a reverse proxy or on a different host), you must set the `OTTERBOT_ALLOWED_ORIGIN` environment variable so CORS works properly. Pass a comma-separated list of allowed origins: `-e OTTERBOT_ALLOWED_ORIGIN=https://otterbot.example.com`

### Full Docker command with all options

```bash
$ docker run -d \
  --name otterbot \
  -p 62626:62626 \
  --shm-size 256m \
  --restart unless-stopped \
  -v ~/otterbot:/otterbot            # Persistent storage
  -e OTTERBOT_DB_KEY=my-secret-key   # Required: DB encryption key
  -e OTTERBOT_UID=$(id -u)           # Match host user
  -e OTTERBOT_GID=$(id -g)           # Match host group
  -e ENABLE_DESKTOP=true             # Virtual XFCE desktop
  -e SUDO_MODE=restricted            # Sudo policy
  -e OTTERBOT_ALLOWED_ORIGIN=https://otterbot.example.com # CORS (if remote)
  ghcr.io/toosmooth/otterbot:latest
```

### Docker Compose

The repository includes a `docker-compose.yml` for easier management. Create a `.env` file alongside it:

```bash
# .env
OTTERBOT_DB_KEY=change-me-to-something-secret
OTTERBOT_UID=1000
OTTERBOT_GID=1000
OTTERBOT_DATA_DIR=./docker/otterbot
```

```bash
$ docker compose up -d
```

## Volume & Persistent Data

OtterBot uses a single bind mount at `/otterbot` inside the container. All persistent data lives here:

```
~/otterbot/                  # or your OTTERBOT_DATA_DIR
+-- config/
|  +-- packages.json        # Auto-restored apt/npm packages
|  +-- bootstrap.sh         # Custom startup script (optional)
+-- data/
|  +-- otterbot.db          # Encrypted SQLite database
+-- home/
|  +-- .ssh/                # SSH keys (auto-chmod 700/600)
|  +-- .gitconfig           # Git configuration
|  +-- .npmrc               # npm configuration
|  +-- .venv/               # Python venv (auto-created)
|  +-- go/                  # Go workspace (GOPATH)
+-- projects/               # Agent workspaces (per project)
+-- tools/                  # npm global installs (persist)
+-- logs/                   # Application logs
```

!!! tip "Persistent home"
    Place your `.ssh` keys, `.gitconfig`, and `.npmrc` in the `home/` directory on the host. The entrypoint automatically fixes SSH permissions on startup.

### Bootstrap Script

To run custom setup on every container start, create a `config/bootstrap.sh` file in your data directory. It runs as root before the application starts:

```bash
#!/bin/sh
# ~/otterbot/config/bootstrap.sh
apt-get update && apt-get install -y --no-install-recommends python3
npm install -g @anthropic-ai/claude-code
```

### Package Manifest

Agents can install packages at runtime. These are recorded in `config/packages.json` and automatically restored on container restart:

```json
{
  "repos": [],
  "apt": [{ "name": "python3" }],
  "npm": [{ "name": "typescript", "version": "latest" }]
}
```

## Development Setup

To run OtterBot from source for development or contribution:

```bash
# Clone the repository
$ git clone https://github.com/TOoSmOotH/otterbot.git
$ cd otterbot

# Install dependencies (monorepo via pnpm workspaces)
$ pnpm install

# Configure environment
$ cp .env.example .env
# Edit .env and set OTTERBOT_DB_KEY to a unique secret

# Apply the database schema
$ pnpm db:push

# Start the dev server (server + web concurrently)
$ pnpm dev
```

The development server starts on `http://localhost:62626` (server) and `http://localhost:5173` (Vite frontend with hot-reload).

!!! info "Monorepo structure"
    OtterBot uses pnpm workspaces with three packages: `@otterbot/server`, `@otterbot/web`, and `@otterbot/shared`. See [Architecture](architecture.md) for details.

You can also develop inside Docker with hot-reload:

```bash
$ pnpm docker:dev
```

## First-Run Setup Wizard

When you first open OtterBot, the setup wizard walks you through the essential configuration:

### 1. LLM Provider

Choose your AI provider and enter your API key. Supported providers:

- **Anthropic** — Claude models (recommended)
- **OpenAI** — GPT-4o, o3, and more
- **Google Gemini** — Gemini 2.5, 2.0 models
- **OpenRouter** — 200+ models from multiple providers
- **Ollama** — Local models (no API key needed)
- **DeepSeek** — DeepSeek Chat and Reasoner
- **xAI** — Grok models
- **Mistral** — Mistral Large, Small, Codestral
- **Perplexity** — Sonar search-grounded models
- **AWS Bedrock** — Managed AWS AI service
- **GitHub Copilot** — GitHub-hosted models
- **Hugging Face** — Open-source models
- **NVIDIA** — NVIDIA-hosted models
- **MiniMax** — MiniMax models
- **Z.AI** — GLM models
- **Deepgram** — Deepgram models
- **LM Studio** — Local models (no API key needed)
- **OpenAI-Compatible** — Any provider with an OpenAI-compatible API

The wizard probes available models from your provider so you can select which model to use for the COO agent and worker agents.

### 2. Voice Setup (optional)

Configure text-to-speech and speech-to-text if desired. You can preview voices during setup. See [Features — Voice](features.md#voice-tts-stt) for provider details.

### 3. Search Provider (optional)

Set up web search for your agents. DuckDuckGo works out of the box with no API key. See [Features — Web Search](features.md#web-search-providers) for all providers.

## Environment Variables

OtterBot is configured through environment variables passed to the container at runtime. LLM API keys, search provider keys, and model preferences are managed through the Settings UI and stored in the encrypted database — not in environment variables.

### Required

| Variable | Description | Default |
|----------|-------------|---------|
| `OTTERBOT_DB_KEY` | Encryption key for the SQLite database. All API keys and settings are stored encrypted with this key. **Required.** | — |

### Docker Runtime

| Variable | Description | Default |
|----------|-------------|---------|
| `OTTERBOT_UID` | User ID for the container process. Set to your host UID (`id -u`) to avoid permission issues with bind-mounted volumes. | `1000` |
| `OTTERBOT_GID` | Group ID for the container process. Set to your host GID (`id -g`). | `1000` |
| `OTTERBOT_DATA_DIR` | Host path for the data volume (used in `docker-compose.yml`). | `./docker/otterbot` |

!!! info "Why set UID/GID?"
    The entrypoint script updates the container's `otterbot` user to match your host UID/GID at startup. This ensures files created inside the container are owned by your host user, avoiding permission headaches with bind mounts.

### Server

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | HTTP server port | `62626` |
| `HOST` | Bind address | `0.0.0.0` |
| `DATABASE_URL` | SQLite database file path | `file:/otterbot/data/otterbot.db` |
| `WORKSPACE_ROOT` | Root directory for all OtterBot data | `/otterbot` |
| `OTTERBOT_ALLOWED_ORIGIN` | Comma-separated list of origins allowed to access the API (CORS). Example: `http://localhost:62626,https://otterbot.example.com` | Same-origin only |

### Desktop Environment

| Variable | Description | Default |
|----------|-------------|---------|
| `ENABLE_DESKTOP` | Start the virtual XFCE desktop with noVNC viewer. When enabled, Playwright runs in headed mode so you can watch agents browse. | `true` |
| `DESKTOP_RESOLUTION` | Desktop resolution (WxHxDepth) | `1280x720x24` |
| `VNC_PORT` | Internal VNC server port (bound to localhost only) | `5900` |
| `SUDO_MODE` | Sudo policy for the container user. `restricted` limits sudo to `apt-get`, `npm`, `tee`, `gpg`, and `install`. `full` grants unrestricted sudo. | `restricted` |

### Toolchain & Paths

| Variable | Description | Default |
|----------|-------------|---------|
| `NPM_CONFIG_PREFIX` | npm global install prefix. Installs persist across restarts on the bind mount. | `/otterbot/tools` |
| `GOPATH` | Go workspace directory | `/otterbot/home/go` |
| `VIRTUAL_ENV` | Python venv location (auto-created on first run) | `/otterbot/home/.venv` |
| `PLAYWRIGHT_BROWSERS_PATH` | Playwright Chromium location | `/opt/playwright` |
| `HOME` | Container home directory (persistent on bind mount) | `/otterbot/home` |

### Pre-installed Languages

The Docker image includes these languages and tools out of the box:

- **Node.js 22** with pnpm
- **Python 3** with pip and venv
- **Go 1.24**
- **Rust** (via rustup)
- **Java** (OpenJDK headless)
- **Ruby**
- **Build tools:** build-essential, git, curl, sqlite3, ffmpeg, GitHub CLI (gh)
- **Coding agent CLIs:** Claude Code, Codex, OpenCode (optional — configure in Settings)

## Hosting for Clients

If you're hosting an OtterBot instance for someone else — a client, teammate, or friend — you can set a temporary bootstrap passphrase so they can log in and complete their own setup (change passphrase, configure LLM provider, profile, voice, search, etc.).

### Bootstrap Passphrase

Set the `OTTER_PASSPHRASE` environment variable to a temporary passphrase before starting the container. This passphrase is marked as temporary — the client will be required to change it on first login.

For Docker secrets, use `OTTER_PASSPHRASE_FILE` instead, pointing to a file containing the passphrase.

```bash
$ docker run -d -p 62626:62626 \
  --name otterbot \
  -v ~/otterbot:/otterbot \
  -e OTTERBOT_DB_KEY=my-secret-key \
  -e OTTER_PASSPHRASE=temp-pass-for-client \
  --shm-size 256m \
  ghcr.io/toosmooth/otterbot:latest
```

### Client Setup Flow

### Step 1 — Log in

The client opens `https://your-host:62626` in their browser and enters the bootstrap passphrase you provided.

### Step 2 — Set a permanent passphrase

Because the bootstrap passphrase is temporary, the client is immediately prompted to choose their own permanent passphrase.

### Step 3 — Complete the setup wizard

The client walks through the standard setup wizard — configuring their LLM provider and API key, profile, voice, and search preferences.

### CORS / Remote Access

If your OtterBot instance is behind a reverse proxy or accessed from a different origin, set `OTTERBOT_ALLOWED_ORIGIN` to the origin(s) that need access:

```bash
-e OTTERBOT_ALLOWED_ORIGIN=https://otterbot.example.com
```

!!! info "HTTPS & Microphone"
    OtterBot automatically generates a self-signed TLS certificate so the UI is served over HTTPS. This is required for browser microphone access (`getUserMedia`) when connecting from a remote host.

### Hosting Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OTTER_PASSPHRASE` | Bootstrap passphrase for client login. Marked as temporary — the client must change it on first use. | — |
| `OTTER_PASSPHRASE_FILE` | Path to a file containing the bootstrap passphrase (for Docker secrets). | — |
| `OTTERBOT_ALLOWED_ORIGIN` | Comma-separated origins allowed to access the API (CORS). Required when the client connects from a different origin than the server. | Same-origin only |

!!! warning "Security"
    The bootstrap passphrase is intended as a one-time handoff. Ensure your client changes it immediately. For production deployments, place OtterBot behind a reverse proxy with a proper TLS certificate rather than relying on the self-signed cert.
