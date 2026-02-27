# Architecture

How OtterBot is built: the agent hierarchy, message bus, workspace isolation, and the
technology that ties it all together.

## System Overview

OtterBot is a multi-agent AI system that runs inside a single Docker container. A human
user (the "CEO") gives high-level goals to an AI orchestrator (the "COO"), which delegates
work through a hierarchy of team leads and workers — each with their own tools and
capabilities.

```text
                    OtterBot Architecture

  +---------------------------------------------------------+
  |  Browser (React + Three.js)                              |
  |  Chat UI · 3D Live View · Agent Graph · Kanban Board   |
  +---------------------------+-----------------------------+
                              |
                     Socket.IO + REST
                              |
  +---------------------------+-----------------------------+
  |  Server (Fastify + Socket.IO)                            |
  |                                                         |
  |  CEO → COO → Team Leads → Workers                  |
  |                                                         |
  |  Message Bus · Agent Registry · Tool Context           |
  |  TTS/STT · Search · Browser Pool · Package Mgr       |
  |  Memory & Soul · Module System · Scheduler            |
  +---------------------------+-----------------------------+
                              |
  +---------------------------+-----------------------------+
  |  Storage & Services                                     |
  |  SQLite (encrypted) · Workspace FS · noVNC Desktop     |
  |  Messaging Bridges · Coding Agent PTY Sessions         |
  +---------------------------------------------------------+
```

## Agent Hierarchy

OtterBot uses a corporate-inspired command chain where each level has distinct
responsibilities:

```text
CEO (Human User)
 |
 |  Sends high-level goals via chat
 |
 v
COO (Single persistent AI agent)
 |
 +--> Admin Assistant (Persistent — todos, email, calendar)
 |
 +--> Scheduler (Background — recurring tasks)
 |
 |  Creates projects, writes charters,
 |  delegates to Team Leads
 |
 v
Team Leads (One per project)
 |
 |  Break directives into tasks,
 |  assign Kanban items, spawn Workers
 |
 v
Workers (Per-task agents)

   Execute individual tasks using tools:
   file I/O, shell, browser, web search, etc.
```

- **CEO (Human)** — You. Send goals through the chat interface.
- **COO** — The primary AI agent. Maintains conversation context, creates projects, writes project charters, and delegates work to Team Leads.
- **Admin Assistant** — Persistent personal productivity agent. Manages todos, reads/sends email via Gmail, and manages Google Calendar events. Reports to the CEO alongside the COO.
- **Scheduler** — Background agent that executes recurring tasks on configurable intervals (e.g., daily standup summaries, periodic monitoring).
- **Team Lead** — Manages a single project. Breaks directives into Kanban tasks, spawns workers, and monitors progress.
- **Worker** — Executes a single task. Has access to tools (file read/write, shell, browser, web search, package installs) based on its template.

!!! info
    **Agent lifecycle:** All agents follow the status cycle
    `Idle` → `Thinking` → `Acting` → `Done`
    (or `Error`). See [Agent System → Lifecycle](agents.md#agent-lifecycle).

## Message Bus

All agent-to-agent communication flows through a central message bus. Every message is
persisted to the database and broadcast to connected clients via Socket.IO.

### Message Flow

- The CEO sends a message via `ceo:message` Socket.IO event
- The COO receives it, processes with an LLM, and responds via `coo:response` / `coo:stream`
- If the COO creates a project, it spawns a Team Lead and sends directives via the bus
- Team Leads spawn Workers and assign tasks via bus messages
- Workers report results back up the chain
- All messages are broadcast as `bus:message` events to the frontend

### BusMessage Structure

Every message on the bus contains:

```json
{
  "id": "msg_abc123",
  "fromAgentId": "coo-main",
  "toAgentId": "tl-project-1",
  "content": "Implement the login page...",
  "role": "assistant",
  "conversationId": "conv_xyz",
  "projectId": "proj_1",
  "timestamp": "2025-01-15T10:30:00Z"
}
```

## Workspace Isolation

Each agent operates within an isolated workspace directory under the
`WORKSPACE_ROOT`. This prevents agents from accidentally interfering with each
other's files.

- The workspace root defaults to `./data`
- Project files are scoped to their project directory
- The database is stored at `WORKSPACE_ROOT/otterbot.db` and encrypted with `OTTERBOT_DB_KEY`
- AI model caches (TTS/STT) are stored under `WORKSPACE_ROOT/data/models`
- The optional virtual desktop runs in its own X11 session

## Memory System

OtterBot includes an episodic memory and soul document system that gives agents persistent
knowledge across conversations.

### Episodic Memories

- Agents can save key observations, decisions, and learnings as **memories**
- Memories are stored with vector embeddings for **semantic search**
- Relevant memories are automatically surfaced during conversations
- A **memory compactor** periodically consolidates related memories to prevent bloat

### Soul Documents

- **Soul documents** define an agent's identity, values, and behavioral guidelines
- A **soul advisor** layer injects relevant identity context into agent prompts
- Soul documents can be created, edited, and managed through the API

## Module System

OtterBot supports an extensible **module system** that lets you install third-party
functionality without modifying core code.

- Modules are installed from GitHub repositories and run in isolation
- Each module can register webhook endpoints, scheduled tasks, or custom behaviors
- Modules can be enabled, disabled, or uninstalled via the API
- Example: the `github-discussions` module monitors GitHub Discussion activity and posts summaries to the COO

## Messaging Bridges

OtterBot can bridge conversations to external messaging platforms, allowing you to interact
with your agents from anywhere.

- **Discord** — Full bot integration via `discord.js`
- **Slack** — Workspace app via `@slack/bolt`
- **Matrix** — Decentralized chat via `matrix-js-sdk`
- **IRC** — Classic IRC networks via `irc-framework`
- **Microsoft Teams** — Teams channel integration via webhook

Each bridge runs independently and relays messages bidirectionally between the external
platform and the COO. Configuration and credentials are managed through the Settings UI.

## Coding Agent Integration

OtterBot can delegate coding tasks to external coding agent CLIs that run in
**PTY (pseudo-terminal) sessions** managed by `node-pty`.

- **OpenCode** — Open-source coding assistant CLI
- **Claude Code** — Anthropic's CLI coding agent
- **Codex** — OpenAI's CLI coding agent

Each coding agent runs in its own terminal session. OtterBot streams output back to the UI
in real-time, handles permission requests (e.g., file write approvals), and captures file
diffs when the session completes. This lets you leverage specialized coding agents while
keeping everything orchestrated through OtterBot's hierarchy.

## Tech Stack

### Backend

| Technology | Purpose |
| --- | --- |
| `Fastify` | HTTP server for REST API |
| `Socket.IO` | Real-time bidirectional communication |
| `Vercel AI SDK` | Unified LLM interface (Anthropic, OpenAI, compatible providers) |
| `Drizzle ORM` | Type-safe database access |
| `better-sqlite3` | SQLite with encryption support |
| `Playwright` | Browser automation for agents |
| `kokoro-js` | Local text-to-speech (Kokoro model) |
| `@huggingface/transformers` | Local speech-to-text (Whisper) |
| `discord.js` | Discord messaging bridge |
| `@slack/bolt` | Slack messaging bridge |
| `matrix-js-sdk` | Matrix messaging bridge |
| `irc-framework` | IRC messaging bridge |
| `node-pty` | PTY sessions for coding agent CLIs |
| `isolated-vm` | Sandboxed execution for custom tools |

### Frontend

| Technology | Purpose |
| --- | --- |
| `React 19` | UI framework |
| `Vite` | Build tool and dev server |
| `Three.js / R3F` | 3D agent visualization (Live View) |
| `@xyflow/react` | Agent graph visualization |
| `Socket.IO Client` | Real-time event handling |

### Infrastructure

| Technology | Purpose |
| --- | --- |
| `Docker` | Container runtime |
| `XFCE + noVNC` | Virtual desktop environment |
| `TypeScript` | Language across all packages |
| `pnpm workspaces` | Monorepo management |

## Package Structure

OtterBot is organized as a monorepo with three pnpm workspace packages:

### @otterbot/server

`packages/server/`

Main backend: Fastify REST API, Socket.IO handlers, agent orchestration (COO, Team Lead,
Worker), database, TTS/STT, search, browser pool, tool execution.

### @otterbot/web

`packages/web/`

React 19 + Vite frontend: chat interface, 3D Live View (Three.js), agent graph (XYFlow),
Kanban board, settings panels, room builder.

### @otterbot/shared

`packages/shared/`

Shared TypeScript type definitions: Agent, AgentRole, AgentStatus, RegistryEntry,
Project, KanbanTask, BusMessage, Socket.IO event interfaces.

```text
otterbot/
+-- package.json          # Root workspace config
+-- packages/
|  +-- server/            # @otterbot/server
|  |  +-- src/
|  |  |  +-- index.ts     # Fastify app + routes
|  |  |  +-- agents/      # COO, Team Lead, Worker, Admin Assistant
|  |  |  +-- socket/      # Socket.IO event handlers
|  |  |  +-- db/          # Drizzle schema, migrations, seed
|  |  |  +-- tools/       # Agent tools (file, shell, browse, search)
|  |  |  +-- tts/         # Text-to-speech providers
|  |  |  +-- stt/         # Speech-to-text providers
|  |  |  +-- registry/    # Agent template registry
|  |  |  +-- models3d/    # 3D model/environment pack discovery
|  |  |  +-- memory/      # Episodic memory & soul documents
|  |  |  +-- coding-agents/ # OpenCode, Claude Code, Codex PTY
|  |  |  +-- messaging/   # Discord, Slack, Teams bridges
|  |  |  +-- irc/         # IRC & Matrix bridges
|  |  |  +-- modules/     # External module system
|  |  |  +-- skills/      # Skill/prompt fragment registry
|  |  |  +-- scheduling/  # Scheduler & custom tasks
|  +-- web/               # @otterbot/web
|  |  +-- src/
|  |  |  +-- components/  # React components
|  |  |  +-- stores/      # State management
|  +-- shared/            # @otterbot/shared
|     +-- src/types/     # TypeScript interfaces & enums
+-- assets/
   +-- workers/           # 3D character models (GLB)
   +-- environments/      # 3D environment models (GLTF)
   +-- scenes/            # Scene configuration JSON
```
