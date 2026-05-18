# Architecture

How OtterBot is built: agent profiles, the orchestrator, per-agent runtimes,
the message bus, and the storage that ties it together.

## System Overview

OtterBot runs **one runtime per agent profile**. The browser talks to a Fastify
server over REST and Socket.IO; the server owns the agent runtimes, the message
bus, the scheduler, and encrypted storage.

```text
  ┌──────────────────────────────────────────────────────────┐
  │  Browser (React + Zustand + Three.js)                     │
  │  Agent roster · Chat · Agent Studio · Activity · 3D       │
  └───────────────────────────┬──────────────────────────────┘
                       Socket.IO + REST
  ┌───────────────────────────┴──────────────────────────────┐
  │  Server (Fastify + Socket.IO)                             │
  │                                                           │
  │  Orchestrator ── AgentRuntime (one per profile)           │
  │       │                                                   │
  │       ├── Message Bus   ── transports: local / discord    │
  │       ├── Scheduler     ── cron prompts                   │
  │       └── Secrets Store ── encrypted per-agent creds      │
  └───────────────────────────┬──────────────────────────────┘
  ┌───────────────────────────┴──────────────────────────────┐
  │  Storage (encrypted SQLite)                               │
  │  control.db  ·  data/profiles/<id>/agent.db               │
  └───────────────────────────────────────────────────────────┘
```

## Agent Profiles

Each agent is an isolated directory under `data/profiles/<id>/`:

```text
profile.json   # identity, role, model refs, transport, avatar, limits
SOUL.md        # persona / system-prompt identity block
agent.db       # conversations, messages, memories, skills, vectors
skills/        # per-agent markdown skills
```

Every agent owns:

| Part | Description |
|------|-------------|
| Identity | Display name, role, email, avatar/model pack, transport |
| Persona | The `SOUL.md` prompt that defines the agent's behavior |
| Models | Separate chat and embedding model references |
| Credentials | API keys and local endpoints scoped to that agent |
| Skills | Markdown instruction packs loaded into that agent's prompt |
| Memory | Long-term facts and observations searchable by that agent |
| Schedule | Cron prompts that run against that agent |
| Subagents | Optional delegated workers with inherited model and secrets |

## Backend Components

| Module | Responsibility |
|--------|----------------|
| `profiles/profile-store.ts` | Creates, loads, saves, and deletes profile directories |
| `orchestrator/orchestrator.ts` | Owns agent lifecycles, runtimes, the bus, the scheduler, secrets, and subagent spawning |
| `runtime/agent-runtime.ts` | Runs one conversational loop for one agent |
| `runtime/agent-context.ts` | Bundles a profile with its DB, memory, skills, secrets, and embeddings |
| `bus/bus.ts` | Persists and routes agent-to-agent messages |
| `scheduler/scheduler.ts` | Runs cron prompts against agents |
| `secrets/secrets-store.ts` | Stores per-agent credentials in the encrypted control DB |
| `providers/registry.ts` | Resolves model references into Vercel AI SDK models |
| `skills/*` | Loads, scans, imports, installs, and exports markdown skills |
| `memory/*` | Summarizes, extracts, stores, and searches agent memories |

## The Message Bus

All agent-to-agent communication flows through a central message bus. When an
agent sends a request, response, broadcast, spawn notice, report, status
update, tool event, or error, the bus:

1. Persists it to `control.db`.
2. Delivers it to the target runtime, or broadcasts it.
3. Emits it to the frontend over Socket.IO.

The **Activity** view in the UI is a live observer of this bus. It also shows
subagent task records — goals, status, and result summaries.

### Transports

The bus is pluggable. The transport is chosen with `AGENT_TRANSPORT`:

- **`local`** (default) — in-process delivery between runtimes.
- **`discord`** — agents communicate over a Discord channel.

## Storage

OtterBot uses encrypted SQLite, split into two layers:

- **`control.db`** — the shared control plane: the agent registry, the
  agent-to-agent bus message log, subagent task records, scheduled tasks,
  encrypted agent secrets, and app settings.
- **`agent.db`** — one per agent, holding that agent's conversations, messages,
  memories, skills, and vector embeddings.

Per-agent databases are intentionally isolated: different agents may use
different embedding models, and `sqlite-vec`'s vector table has a fixed
embedding dimension.

Every database is encrypted with `OTTERBOT_DB_KEY`. Credentials are never on
disk in plaintext — they live in the encrypted `agent_secrets` table.

## Tech Stack

### Backend

| Technology | Purpose |
|------------|---------|
| Fastify | HTTP server for the REST API |
| Socket.IO | Real-time bidirectional communication |
| Vercel AI SDK | Unified model interface (Anthropic, OpenAI, LM Studio, Ollama) |
| Drizzle ORM | Type-safe database access |
| SQLite + `sqlite-vec` | Encrypted storage with vector search |
| `croner` | Cron scheduling |
| `discord.js`, `@slack/web-api`, `nodemailer` | Integrations |

### Frontend

| Technology | Purpose |
|------------|---------|
| React 19 | UI framework |
| Vite | Build tool and dev server |
| Zustand | State management |
| Three.js | 3D agent scene |
| Tailwind CSS | Styling |
| Socket.IO client | Real-time event handling |

## Package Structure

OtterBot is a pnpm-workspace monorepo:

```text
packages/
  shared/   TypeScript contracts for agents, messages, memory, skills, views
  server/   Fastify API, Socket.IO, orchestration, profile storage, SQLite
  web/      React/Vite UI, Zustand stores, Three.js agent scene
  cli/      Ink terminal client and daemon launcher
assets/
  workers/       3D character / model packs
  environments/  3D environment packs
  scenes/        scene configurations
```
