# Features Guide

A deep dive into OtterBot's capabilities -- from voice interaction and 3D visualization
to web search, browser automation, and project management.

## Voice (TTS & STT)

OtterBot supports both text-to-speech (reading responses aloud) and speech-to-text
(voice input). Both can run locally without any API keys.

### Text-to-Speech Providers

#### Kokoro

**Local**

High-quality local TTS using the Kokoro 82M ONNX model. Runs entirely on CPU inside
the container. Default voice: `af_heart`.

#### Edge TTS

**Free Cloud**

Microsoft's neural TTS voices. Free, no API key needed. Wide variety of voices and
languages available.

#### OpenAI-Compatible

**API Key**

Any provider with an OpenAI-compatible TTS endpoint (`/v1/audio/speech`).
Configure the base URL and optional API key.

### TTS Configuration

| Setting Key | Description | Default |
|---|---|---|
| `tts:enabled` | Enable or disable TTS | `false` |
| `tts:active_provider` | Active provider: `kokoro`, `edge-tts`, or `openai-compatible` | `kokoro` |
| `tts:voice` | Voice name | `af_heart` |
| `tts:speed` | Playback speed (0.5 -- 2.0) | `1` |
| `tts:openai-compatible:base_url` | Base URL for OpenAI-compatible provider | -- |
| `tts:openai-compatible:api_key` | API key (optional) | -- |

### Speech-to-Text Providers

#### Whisper Local

**Local**

OpenAI Whisper ONNX model running locally. Default model:
`onnx-community/whisper-base`. Configurable model ID.

#### OpenAI-Compatible

**API Key**

Any provider with an OpenAI-compatible transcription endpoint
(`/v1/audio/transcriptions`).

#### Browser Web Speech API

**Built-in**

Uses the browser's native Web Speech API for speech recognition. No setup or API key
required — works directly in supported browsers (Chrome, Edge).

### STT Configuration

| Setting Key | Description | Default |
|---|---|---|
| `stt:enabled` | Enable or disable STT | `false` |
| `stt:active_provider` | Active provider: `whisper-local` or `openai-compatible` | `whisper-local` |
| `stt:whisper:model_id` | HuggingFace model ID for local Whisper | `onnx-community/whisper-base` |
| `stt:openai-compatible:base_url` | Base URL for OpenAI-compatible provider | -- |
| `stt:openai-compatible:api_key` | API key (optional) | -- |

## 3D Live View

The Live View renders your agents as animated 3D characters in a virtual scene using
Three.js and React Three Fiber. Each agent can have its own character model with idle and
action animations.

### Model Packs

Character models are GLB files organized into **model packs**. OtterBot
auto-discovers packs from the `assets/workers/` directory:

```
assets/workers/
+-- MyCharacter/
|  +-- characters/
|  |  +-- character.glb        # 3D character model
|  +-- Animations/
|  |  +-- gltf/
|  |     +-- Rig_Medium/        # Medium rig animations
|  |     +-- Rig_Large/         # Large rig animations
|  +-- artwork.png              # Thumbnail
```

Each model pack includes idle and action animations. Agents assigned a model pack will
appear as that character in the Live View, with animations reflecting their current status.

### Environment Packs

Environment models (GLTF) are loaded from `assets/environments/` and define the
3D scene backdrop. Multiple environment assets can be placed in a scene.

### Scene Configurations

JSON files in `assets/scenes/` define camera position, lighting, and layout.
The Room Builder (below) lets you create and edit scene configurations visually.

## Web Search Providers

Agents with the `web_search` tool can search the web. OtterBot supports four
search providers:

#### DuckDuckGo

**Free -- No API Key**

Default provider. Searches via DuckDuckGo HTML interface. No configuration needed.

#### Brave Search

**API Key**

Brave's independent search index. Requires a Brave Search API key.
Config: `search:brave:api_key`.

#### Tavily

**API Key**

AI-optimized search API designed for LLM agents. Requires a Tavily API key.
Config: `search:tavily:api_key`.

#### SearXNG

**Self-Hosted**

Privacy-respecting metasearch engine. Connect to your own SearXNG instance.
Config: `search:searxng:base_url`.

Set the active provider with the `search:active_provider` setting. All providers
return results in a unified format with title, URL, and snippet.

## Browser Automation

Agents with the `web_browse` tool can navigate web pages, extract content, and
interact with websites using **Playwright**. A browser pool manages Chromium
instances efficiently.

- Navigate to URLs and extract page content as text
- Managed browser pool prevents resource exhaustion
- Runs headless Chromium inside the Docker container
- Browser binaries path configurable via `PLAYWRIGHT_BROWSERS_PATH`

!!! info

    **Researcher agents** have both `web_search` and
    `web_browse` -- search for results, then browse specific pages for
    detailed information.

## Project Management & Kanban

OtterBot organizes complex work into **Projects**, each with a Kanban board
for task tracking. Projects are created by the COO when you give it a multi-step goal.

### Project Lifecycle

- **Creation** -- COO creates a project with a name and description
- **Charter Gathering** -- COO writes a project charter (specification/requirements)
- **Charter Finalized** -- Charter is locked, Team Lead is spawned
- **Execution** -- Team Lead creates Kanban tasks and assigns workers
- **Completion** -- Project status set to `completed`, `failed`, or `cancelled`

### Kanban Board

Each project has a Kanban board with three columns:

| Column | Description |
|---|---|
| `backlog` | Tasks planned but not yet started |
| `in_progress` | Tasks currently being worked on by an agent |
| `done` | Completed tasks with optional completion reports |

Tasks can have labels, blocking dependencies, and agent assignments. Workers automatically
move tasks to `in_progress` when they start and `done` when they finish.
The frontend displays the Kanban board in real-time via Socket.IO events.

## Desktop Environment

OtterBot includes an optional full **XFCE desktop environment** running inside
the container, accessible via **noVNC** in your browser. This gives agents (and
you) access to a complete Linux desktop with GUI applications.

### What's Included

- **XFCE4** -- Lightweight desktop environment
- **File manager** -- Browse the workspace filesystem
- **Terminal** -- Full shell access
- **Web browser** -- Chromium for agents' browser automation
- **noVNC** -- Browser-based VNC client (no VNC client needed)

### Configuration

| Variable | Description | Default |
|---|---|---|
| `ENABLE_DESKTOP` | Enable or disable the virtual desktop | `true` |
| `DESKTOP_RESOLUTION` | Screen resolution (WxHxDepth) | `1280x720x24` |
| `VNC_PORT` | Internal VNC server port | `5900` |
| `SUDO_MODE` | Privilege level: `restricted` (safe) or `full` | `restricted` |

The desktop status can be checked via the `GET /api/desktop/status` endpoint.
noVNC files are served directly from the server at `/novnc/*`.

## Room Builder

The Room Builder is a visual 3D scene editor in the web UI. It lets you place environment
assets, position the camera, and configure the Live View scene without editing JSON files.

- Drag and drop environment assets from available packs
- Position, rotate, and scale objects in 3D space
- Configure lighting and camera angles
- Save scene configurations for the Live View

## Package Management

Agents with the `install_package` tool can install software packages inside
the container. This supports both:

- **apt packages** -- System-level packages (e.g., build tools, libraries)
- **npm packages** -- Node.js packages installed globally to `/otterbot/tools`

Installed packages can be listed via the `GET /api/packages` REST endpoint.
The `NPM_CONFIG_PREFIX` environment variable controls where npm global packages
are installed.

!!! tip

    **Ephemeral by default:** Packages installed inside the container are lost
    when the container stops. Mount a volume to persist installed tools across sessions.

## Admin Assistant

The Admin Assistant is a persistent personal productivity agent that handles everyday tasks
alongside the COO. It reports directly to the CEO (you) and manages:

- **Todos** -- Create, list, update, and complete personal todo items with priorities and due dates
- **Email** -- Read, search, send, reply, archive, and label Gmail messages
- **Calendar** -- Create, update, delete, and list Google Calendar events across multiple calendars

The Admin Assistant runs as its own agent role (`admin_assistant`) with a dedicated
system prompt optimized for productivity tasks. It can save memories to remember your
preferences across sessions.

## Gmail Integration

OtterBot integrates with Gmail via Google OAuth, giving the Admin Assistant full email
capabilities.

| Capability | Description |
|---|---|
| **Read messages** | Fetch and read email messages by ID or search query |
| **Send messages** | Compose and send new emails |
| **Reply** | Reply to existing email threads |
| **Archive** | Archive messages to remove them from the inbox |
| **Label** | Apply or remove Gmail labels for organization |
| **List labels** | Retrieve all available Gmail labels |

Gmail access requires configuring Google OAuth credentials in the Settings UI. All
email operations go through Google's official API.

## Google Calendar

The Admin Assistant can manage your Google Calendar, creating and organizing events
through natural language requests.

- **List calendars** -- See all available calendars on your account
- **List events** -- Query events by date range or calendar
- **Create events** -- Schedule new events with title, time, location, and description
- **Update events** -- Modify existing event details
- **Delete events** -- Remove events from your calendar

Calendar access shares the same Google OAuth credentials as Gmail.

## Custom Tools

OtterBot lets you create **custom JavaScript tools** that agents can use at
runtime. Tools are sandboxed using `isolated-vm` for security.

- **Create tools manually** -- Write a tool with a name, description, JSON schema for parameters, and JavaScript implementation
- **AI-generate tools** -- Describe what you need and OtterBot will generate the tool code for you
- **Sandboxed execution** -- Tools run in an isolated V8 context with controlled access to `fetch` and environment variables
- **Tool examples** -- Browse built-in example tools for inspiration

Custom tools appear in the `/api/tools/available` endpoint and can be assigned
to any agent template.

## Skills System

Skills are reusable **markdown prompt fragments** that can be attached to agents
to give them specialized knowledge or behavior patterns.

- **Create skills** -- Write markdown content that gets injected into agent system prompts
- **Clone skills** -- Duplicate existing skills as a starting point
- **Import/export** -- Share skills between OtterBot instances as JSON
- **Scan** -- Auto-discover skills from a directory structure

## Memory & Soul

OtterBot's memory system gives agents persistent knowledge that carries across conversations.

### Episodic Memories

Agents can save important observations, decisions, and learnings as episodic memories. These
are stored with vector embeddings and retrieved via **semantic search** when
relevant to the current conversation.

- Memories are automatically extracted from significant conversations
- A **memory compactor** consolidates similar memories to prevent bloat
- Memories can be listed, saved, and managed via the API and Socket.IO

### Soul Documents

Soul documents define an agent's identity, values, and long-term behavioral guidelines.
They provide a stable foundation that persists regardless of conversation context.

- Create and edit soul documents through the API
- A **soul advisor** injects relevant identity context into agent prompts
- Useful for defining company culture, communication style, or domain expertise

## Coding Agents

OtterBot can delegate coding tasks to external coding agent CLIs, running them in managed
**PTY (pseudo-terminal) sessions**.

#### OpenCode

**Open Source**

Open-source coding assistant. Configure the CLI path in Settings.

#### Claude Code

**API Key**

Anthropic's CLI coding agent. Requires Claude Code CLI installed and an Anthropic API key.

#### Codex

**API Key**

OpenAI's CLI coding agent. Requires Codex CLI installed and an OpenAI API key.

- **PTY sessions** -- Each coding agent runs in its own terminal, streamed to the UI in real-time
- **Permission handling** -- File write and shell command permissions are relayed to the user for approval
- **File diffs** -- Changes made by the coding agent are captured and displayed
- **Session management** -- Start, monitor, and stop coding agent sessions via Socket.IO

## Messaging Integrations

OtterBot can bridge conversations to eleven external messaging platforms, letting you chat
with your agents from anywhere.

#### Discord

**Bot Token**

Full Discord bot integration. Configure a bot token and channel in Settings.

#### Slack

**App Token**

Slack workspace app via Bolt framework. Requires app and bot tokens. Supports threaded conversations.

#### Matrix

**Self-Hosted**

Decentralized chat protocol with end-to-end encryption (E2EE) support. Connect to any Matrix homeserver.

#### IRC

**Open Protocol**

Classic IRC networks. Configure server, channel, and nickname.

#### Microsoft Teams

**Webhook**

Teams channel integration via incoming/outgoing webhooks.

#### Telegram

**Bot Token**

Telegram bot integration. Create a bot via BotFather and configure the token in Settings.

#### WhatsApp

**Bridge**

WhatsApp messaging bridge integration.

#### Signal

**Bridge**

Signal messenger bridge for secure messaging.

#### Mattermost

**Webhook/Bot**

Mattermost team chat integration.

#### Nextcloud Talk

**Integration**

Nextcloud Talk chat integration.

#### Tlon

**Integration**

Tlon (Urbit) communication platform integration.

Each bridge relays messages bidirectionally between the external platform and the COO.
Configure credentials and channels through the Settings UI.

## Scheduled Tasks

OtterBot supports **custom recurring tasks** that run automatically on
configurable intervals. Use these for:

- Daily standup summaries
- Periodic monitoring or health checks
- Automated report generation
- Regular data collection tasks

Scheduled tasks are managed through the API with configurable intervals and can be
enabled or disabled individually.

## Usage Analytics

OtterBot tracks token usage and costs across all LLM interactions, giving you visibility
into resource consumption.

| View | Description |
|---|---|
| **Summary** | Total tokens and estimated costs across all models |
| **Recent** | Recent usage entries with timestamps |
| **By Model** | Token usage broken down by LLM model |
| **By Agent** | Token usage broken down by agent instance |

## Backup & Restore

OtterBot supports full **database backup and restore** through the Settings UI
and API.

- **Backup** -- Download a complete backup of the encrypted database
- **Restore** -- Upload a previous backup to restore all data (settings, conversations, agents, projects)

## GitHub Integration

OtterBot includes built-in GitHub integration for managing repositories and monitoring activity.

- **SSH key management** -- Generate, import, test, and delete SSH keys for Git authentication
- **Issue monitoring** -- Track GitHub issues and discussions
- **Auto-project creation** -- Automatically create OtterBot projects from GitHub issues
- **GitHub CLI** -- The `gh` CLI is pre-installed for agents to use

## Specialist Agents

Specialist Agents are OtterBot's primary extension point for connecting to external data
sources. Each specialist is an autonomous agent with its own isolated knowledge store,
data ingestion pipeline, configuration, and custom tools.

!!! info
    Specialists were formerly called "modules." The internal implementation still uses module
    naming for backward compatibility — `defineSpecialist()` and `defineModule()` are aliases.

### Capabilities

| Capability | Description |
|---|---|
| **Knowledge Store** | Isolated SQLite database with hybrid FTS5 + vector search per specialist |
| **Data Ingestion** | Automated polling on configurable intervals and/or webhook listeners |
| **Custom Tools** | Specialist-specific tools available to the specialist's agent |
| **AI Agent** | Optional reasoning layer — an LLM agent that can query and synthesize from the knowledge store |
| **Config Schema** | Typed settings (string, number, boolean, secret, select) managed through the Settings UI |
| **Migrations** | Versioned database schema migrations for custom tables |

### Knowledge Store

Each specialist gets its own SQLite database with **hybrid search** combining:

- **FTS5 full-text search** for keyword matching (BM25 ranking)
- **Vector embeddings** for semantic similarity (cosine similarity)
- **Reciprocal rank fusion** to merge both result sets

```typescript
// Available via ctx.knowledge in all handlers
ctx.knowledge.upsert("doc-1", "Document content", { url: "..." });
const results = await ctx.knowledge.search("query", 10);
const doc = ctx.knowledge.get("doc-1");
const count = ctx.knowledge.count();
```

### Triggers & Data Pipeline

Specialists ingest data through two trigger types:

#### Poll Trigger

Runs on a configurable interval. Items returned from the `onPoll` handler are automatically
upserted into the knowledge store.

```typescript
triggers: [
  { type: "poll", intervalMs: 300_000, minIntervalMs: 60_000 }  // 5 min poll, 1 min minimum
]
```

A `onFullSync` handler can also be defined for comprehensive reindexing of all data.

#### Webhook Trigger

Registers a webhook endpoint at `POST /api/modules/:moduleId/webhook`. Supports GitHub
signature verification (`X-Hub-Signature-256`) and secret-based auth.

```typescript
triggers: [
  { type: "webhook", path: "/webhook" }
]
```

### Custom Tools

Specialists can expose tools to their agent. Every specialist agent also automatically
receives a `knowledge_search` tool for hybrid search over its knowledge store.

```typescript
tools: [
  {
    name: "search_discussions",
    description: "Search discussions with filters",
    parameters: {
      query: { type: "string", description: "Search text" },
      category: { type: "string", description: "Filter by category" },
    },
    async execute(args, ctx) {
      // Can use raw DB access: ctx.knowledge.db.prepare(sql)
      return JSON.stringify(results);
    },
  },
]
```

### Agent Configuration

Specialists can declare an AI agent that reasons over the indexed knowledge:

```typescript
agent: {
  defaultName: "Discussions Agent",
  defaultPrompt: "You are a GitHub Discussions specialist...",
  defaultModel: "claude-sonnet-4-5-20250929",  // optional
  defaultProvider: "anthropic",                  // optional
}
```

Agent behavior is controlled by a **posting mode**:

| Mode | Behavior |
|---|---|
| `respond` | Always respond to queries (default) |
| `lurk` | Index only — respond only to direct queries from COO/CEO |
| `new_chats` | Respond to new conversations, then lurk |
| `permission` | Ask COO for permission before responding |

### Installation

Specialists can be installed from three sources:

| Source | Description |
|---|---|
| **Git** | Cloned from a GitHub repository, built with `pnpm install && pnpm build` |
| **npm** | Installed from an npm registry |
| **Local** | Symlinked from a local path (for development) |

Install via the REST API, the COO's `module_install` tool, or the Settings UI. Multiple
instances of the same specialist type can be installed with different configurations.

### Management

- **Enable/disable** -- Toggle specialists without uninstalling
- **Configure** -- Update settings through the UI (config schema fields appear automatically)
- **Reload** -- Unload and reload a specialist to pick up changes
- **Query** -- The COO can query any specialist's knowledge via the `module_query` tool

### Defining a Specialist

Create a package with an entry point that exports a specialist definition:

```typescript
import { defineSpecialist } from "@otterbot/shared";

export default defineSpecialist({
  manifest: {
    id: "my-specialist",
    name: "My Specialist",
    version: "1.0.0",
    description: "Monitors an external data source",
  },
  configSchema: { /* typed settings */ },
  agent: { /* optional AI agent config */ },
  tools: [ /* custom tools */ ],
  triggers: [ /* poll and/or webhook triggers */ ],
  migrations: [ /* database schema versions */ ],
  onPoll: async (ctx) => { /* fetch and return items */ },
  onWebhook: async (req, ctx) => { /* handle webhook events */ },
  onQuery: async (query, ctx) => { /* custom search logic */ },
  onLoad: async (ctx) => { /* startup logic */ },
  onUnload: async (ctx) => { /* cleanup logic */ },
});
```

### Example: GitHub Discussions

The built-in `github-discussions` specialist demonstrates the full system:

- **Polls** GitHub's GraphQL API every 5 minutes for new discussions
- **Indexes** discussions and comments into its knowledge store with structured metadata
- **Handles webhooks** from GitHub for real-time updates on discussion/comment events
- **Exposes** a `search_discussions` tool for structured filtering by category, author, and answered status
- **Provides** an AI agent that synthesizes answers with discussion numbers and URLs

## MCP Integration

OtterBot supports the **Model Context Protocol (MCP)**, letting you connect external
MCP servers that provide additional tools for your agents.

### Server Management

- **Add MCP servers** — Configure stdio or SSE-based MCP servers with command, args, and environment variables
- **Start/stop servers** — Manage MCP server lifecycles at runtime
- **Tool discovery** — Automatically discover available tools from connected MCP servers
- **Tool filtering** — Select which discovered tools to make available to agents
- **Security** — Command validation, URL validation, and secret masking built in

### Transport Types

| Transport | Description |
|---|---|
| `stdio` | Runs the MCP server as a local child process (command + args) |
| `sse` | Connects to a remote MCP server via Server-Sent Events (HTTPS required) |

MCP servers are configured through the Settings UI or REST API. Discovered tools appear
alongside built-in tools and can be assigned to agent templates.

## Code Review Pipeline

OtterBot includes an automated **code review pipeline** that orchestrates multi-stage
review and implementation workflows for pull requests.

### Pipeline Stages

| Stage | Description |
|---|---|
| `Triage` | LLM-based classification of the issue or PR |
| `Implementation` | Spawns coding agents to implement changes |
| `Testing` | Validates changes and runs checks |
| `Integration` | Final review with optional kickback to earlier stages |

The pipeline integrates with the Merge Queue for end-to-end automation: issues flow
through triage and implementation, PRs are opened automatically, and validated changes
are queued for merge.

## Merge Queue

The **merge queue** automates the process of merging approved pull requests safely
and sequentially.

### Merge Flow

- **Queued** — PR is approved and added to the merge queue
- **Rebasing** — Automatically rebased onto the target branch
- **Re-review** — Optional pipeline check after rebase to verify changes still pass
- **Merging** — PR is merged after all checks pass

Features include automatic conflict detection, sequential processing to prevent
race conditions, position-based queue reordering, and real-time status updates via
Socket.IO events.

## World Layout

The World Layout system manages **zones** for 3D scene organization in the
Live View.

- **Create zones** -- Define named areas in the 3D world with position and size
- **Manage zones** -- Update or delete zones via the API
- **Agent placement** -- Agents can be positioned within zones for spatial organization
