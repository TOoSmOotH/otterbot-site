# Getting Started

OtterBot ships to npm as a single self-contained package — the server, the web
UI, and the CLI in one install.

## Prerequisites

- **Node.js 20+** (Node 22 recommended)
- A **model provider** — one of:
    - **Anthropic** or **OpenAI** — a cloud API key (or a ChatGPT subscription for OpenAI)
    - **LM Studio** or **Ollama** — a local OpenAI-compatible endpoint, no key required

## Install

```bash
npm install -g otterbot
```

This installs the `otterbot` command (also available as `otter`).

## Run the Daemon

`otterbot` runs as a background **daemon** that serves the web UI and chat on a
single port. Control it with the CLI:

```bash
otterbot start      # start the daemon in the background
otterbot status     # check whether it is running
otterbot web        # open the web UI in a browser
otterbot chat       # interactive terminal chat (the default with no command)
otterbot logs -f    # follow the daemon log
otterbot restart    # restart the daemon
otterbot stop       # stop the daemon
```

| Command | Description |
|---------|-------------|
| `otterbot start` | Start the daemon in the background |
| `otterbot stop` | Stop the daemon |
| `otterbot restart` | Restart the daemon |
| `otterbot status` | Report whether the daemon is running |
| `otterbot logs [-f]` | Print the daemon log; `-f` follows it |
| `otterbot chat` | Open an interactive terminal chat (the default with no command) |
| `otterbot web` | Open the web UI in a browser |

## First-Run Setup

On first run, the daemon generates an `OTTERBOT_DB_KEY` and an onboarding
wizard in the web UI walks you through configuring your first agent — the
**COO**:

1. **Provider** — choose Anthropic, OpenAI, LM Studio, or Ollama.
2. **Model** — pick a chat model; OtterBot probes the provider for available models.
3. **Credentials** — enter an API key (cloud providers) or an endpoint URL (local providers).
4. **Personality** — give the COO its persona.

You can skip setup and configure agents later in Agent Studio.

!!! warning "Back up your encryption key"
    `OTTERBOT_DB_KEY` encrypts every database, including all stored
    credentials. If you lose it, the databases cannot be opened.

## Where Data Lives

State lives under `~/.otterbot/`:

```text
~/.otterbot/
├── otterbot.pid     # daemon process id
├── otterbot.log     # daemon log
├── .env             # contains the generated OTTERBOT_DB_KEY
└── data/
    ├── control.db   # shared control plane (registry, bus, secrets, settings)
    └── profiles/    # one directory per agent
```

Set `OTTERBOT_HOME` to relocate this directory, or `PORT` / `HOST` to change
where the daemon listens (default `3001` / `0.0.0.0`).

## Running From Source

To develop or contribute, run OtterBot from source. Requires **Node.js 20+** and
**pnpm 9+**.

```bash
git clone https://github.com/TOoSmOotH/otterbot.git
cd otterbot
pnpm install
pnpm dev
```

`pnpm dev` starts both servers:

- Backend API and Socket.IO: `http://localhost:3001`
- Vite web app (hot reload): `http://localhost:5173`

On first launch, `pnpm dev` creates `.env` from `.env.example` and generates a
local `OTTERBOT_DB_KEY`.

For a single-port, production-style run, build first, then start — the backend
serves the built frontend from the same port as the API:

```bash
pnpm build
pnpm start
```

## Configuration

OtterBot reads configuration from `.env`. For development, `pnpm dev` creates it
from `.env.example` automatically; to reset it manually, run
`cp .env.example .env`.

Most credentials — API keys, endpoints, tokens — are stored **per agent** in the
encrypted database and managed through Agent Studio, not in `.env`. The one
secret expected in `.env` is `OTTERBOT_DB_KEY`.

| Variable | Default | Description |
|----------|---------|-------------|
| `OTTERBOT_DB_KEY` | unset | Encrypts `control.db` and every per-agent database. If unset, local dev databases are unencrypted. |
| `PORT` | `3001` | Backend HTTP and Socket.IO port |
| `HOST` | `0.0.0.0` | Backend bind host |
| `DATA_DIR` | `./data` | Runtime data directory |
| `LOG_LEVEL` | `info` | Logger level |
| `LMSTUDIO_BASE_URL` | `http://localhost:1234/v1` | Default OpenAI-compatible local endpoint |
| `LMSTUDIO_MODEL` | `local-model` | Initial COO chat model fallback |
| `ENABLE_EMBEDDINGS` | `false` | Enables embedding initialization for semantic memory |
| `AGENT_TRANSPORT` | `local` | Agent bus transport: `local` or `discord` |
| `DISCORD_BOT_TOKEN` | unset | Required only when `AGENT_TRANSPORT=discord` |
| `DISCORD_CHANNEL_ID` | unset | Required only when `AGENT_TRANSPORT=discord` |

## Next Steps

- Learn how OtterBot is put together in [Architecture](architecture.md).
- Understand agent roles and Agent Studio in [Agent System](agents.md).
- Browse what you can do in [Features](features.md).
