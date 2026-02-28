# API Reference

Complete reference for OtterBot's REST API endpoints, Socket.IO events, and core data types.

## Authentication

OtterBot uses session-based authentication. After initial setup, the server issues a session
cookie. All subsequent requests (REST and Socket.IO) are authenticated via this cookie.

- Check auth status: `GET /api/auth/check`
- Logout: `POST /api/auth/logout`
- Socket.IO connections are authenticated on the `connection` event

## REST: Setup

### GET /api/setup/status

Returns whether the initial setup has been completed and which provider types are configured.

```json
// Response
{
  "setupComplete": true,
  "providerType": "anthropic"
}
```

### POST /api/setup/probe-models

Probe available models from an LLM provider. Used during the setup wizard to list models.

```json
// Request body
{
  "provider": "anthropic",
  "apiKey": "sk-ant-...",
  "baseUrl": "https://api.anthropic.com"  // optional
}
```

### POST /api/setup/tts-preview

Preview a TTS voice. Returns audio for a sample sentence.

```json
// Request body
{
  "voice": "af_heart",
  "ttsProvider": "kokoro"  // optional
}
```

## REST: Auth

### GET /api/auth/check

Check current authentication status.

### POST /api/auth/logout

Logout and destroy the current session.

## REST: Agents & Registry

### GET /api/agents

List all active agent instances.

```json
// Response
[
  {
    "id": "coo-main",
    "name": "COO",
    "role": "coo",
    "status": "idle",
    "registryId": "builtin-coo",
    "projectId": null
  }
]
```

### GET /api/registry

List all agent templates (built-in and custom) from the registry.

```json
// Response: RegistryEntry[]
[
  {
    "id": "builtin-coder",
    "name": "Coder",
    "role": "worker",
    "capabilities": ["code", "typescript", "python", "debugging"],
    "tools": ["file_read", "file_write", "shell_exec", "install_package"],
    "builtIn": true,
    ...
  }
]
```

### GET /api/registry/:id

Get a specific agent template by ID.

### POST /api/registry

Create a new custom agent template.

```json
// Request body: RegistryEntryCreate
{
  "name": "API Tester",
  "description": "Tests REST APIs...",
  "systemPrompt": "You are an API testing specialist...",
  "capabilities": ["testing", "api"],
  "defaultModel": "claude-sonnet-4-5-20250929",
  "defaultProvider": "anthropic",
  "tools": ["shell_exec", "file_read"]
}
```

### POST /api/registry/:id/clone

Clone an existing template (built-in or custom) as a new custom template.

### PATCH /api/registry/:id

Update a custom agent template. Built-in templates cannot be modified.

### DELETE /api/registry/:id

Delete a custom agent template. Built-in templates cannot be deleted.

## REST: Conversations & Messages

### GET /api/conversations

List conversations. Optionally filter by project.

**Query Parameters**

`projectId` (optional) -- Filter conversations by project ID

### GET /api/messages

Cursor-paginated message history.

**Query Parameters**

- `projectId` -- Project ID
- `agentId` -- Agent ID
- `limit` -- Number of messages (default 50)
- `before` -- Cursor for pagination (message ID)

## REST: Projects

### GET /api/projects

List all projects.

```json
// Response: Project[]
[
  {
    "id": "proj_abc",
    "name": "Build Login Page",
    "description": "Create a login page with OAuth...",
    "status": "active",
    "charter": "...",
    "charterStatus": "finalized",
    "createdAt": "2025-01-15T10:00:00Z"
  }
]
```

## REST: 3D Models & Scenes

### GET /api/model-packs

List available 3D character model packs for the Live View.

```json
// Response: ModelPack[]
[
  {
    "id": "otter-default",
    "name": "Otter Default",
    "characterUrl": "/assets/workers/otter/characters/otter.glb",
    "thumbnailUrl": "/assets/workers/otter/artwork.png",
    "animations": {
      "idle": "/assets/workers/otter/Animations/gltf/...",
      "action": "/assets/workers/otter/Animations/gltf/..."
    }
  }
]
```

### GET /api/environment-packs

List available 3D environment packs.

```json
// Response: EnvironmentPack[]
[
  {
    "id": "office",
    "name": "Office",
    "assets": [
      { "id": "desk", "name": "Desk", "modelUrl": "..." }
    ]
  }
]
```

### GET /api/scenes

List available scene configurations (camera, lighting, layout).

### PUT /api/scenes/:id

Save or update a scene configuration.

## REST: Settings

### GET /api/settings

Get all settings as key-value pairs.

### LLM Providers

### GET /api/settings/providers

Get configured LLM provider settings (provider type, models).

### PUT /api/settings/provider/:id

Update an LLM provider's configuration (API key, base URL, etc.).

### PUT /api/settings/defaults

Update default models for each agent tier (COO, Team Lead, Worker).

### POST /api/settings/provider/:id/test

Test connectivity to an LLM provider.

### GET /api/settings/models/:id

Fetch available models from a specific provider.

### Search Settings

### GET /api/settings/search

Get search provider configuration.

### PUT /api/settings/search/active

Set the active search provider.

### PUT /api/settings/search/provider/:id

Update a search provider's configuration (API key, base URL).

### POST /api/settings/search/provider/:id/test

Test connectivity to a search provider.

### TTS Settings

### GET /api/settings/tts

Get text-to-speech settings (provider, voice, speed).

### PUT /api/settings/tts/enabled

Enable or disable TTS.

### PUT /api/settings/tts/active

Set the active TTS provider.

### PUT /api/settings/tts/voice

Set the TTS voice.

### PUT /api/settings/tts/speed

Set the TTS playback speed.

### PUT /api/settings/tts/provider/:id

Update a TTS provider's configuration.

### POST /api/settings/tts/provider/:id/test

Test a TTS provider.

### POST /api/settings/tts/preview

Preview a TTS voice with a sample sentence.

### STT Settings

### GET /api/settings/stt

Get speech-to-text settings (provider, model).

### PUT /api/settings/stt/enabled

Enable or disable STT.

### PUT /api/settings/stt/active

Set the active STT provider.

### PUT /api/settings/stt/language

Set the STT language.

### PUT /api/settings/stt/model

Set the Whisper model ID for local STT.

### PUT /api/settings/stt/provider/:id

Update an STT provider's configuration.

### POST /api/settings/stt/provider/:id/test

Test an STT provider.

### Other Settings

### POST /api/stt/transcribe

Transcribe audio to text. Accepts multipart form data with an audio file.

**Body (multipart/form-data)**

`file` -- Audio file (WebM format)

### GET /api/settings/opencode

Get OpenCode integration settings.

### POST /api/settings/opencode/test

Test the OpenCode integration configuration.

## REST: Desktop, Packages, Profile

### GET /api/desktop/status

Get the virtual desktop environment status (running, resolution, ports).

### GET /novnc/*

Serve noVNC web client files. Path traversal protection is enforced.

### GET /api/packages

List installed apt and npm packages.

### POST /api/packages

Install a new package (apt or npm).

### DELETE /api/packages

Uninstall a package.

### GET /api/profile

Get the current user's profile information.

### PUT /api/profile/model-pack

Update the user's 3D model pack for the Live View.

## REST: Todos

### GET /api/todos

List all todo items.

### POST /api/todos

Create a new todo item.

```json
// Request body
{
  "title": "Review PR #42",
  "description": "Check for security issues",
  "priority": "high",
  "dueDate": "2025-03-01T00:00:00Z"
}
```

### PUT /api/todos/:id

Update a todo item (title, description, priority, status, due date).

### DELETE /api/todos/:id

Delete a todo item.

## REST: Custom Tools

### GET /api/tools

List all custom tools.

### POST /api/tools

Create a new custom tool.

```json
// Request body
{
  "name": "weather_lookup",
  "description": "Get current weather for a city",
  "parameters": { /* JSON Schema */ },
  "code": "const resp = await fetch(...);"
}
```

### PATCH /api/tools/:id

Update an existing custom tool.

### DELETE /api/tools/:id

Delete a custom tool.

### POST /api/tools/ai-generate

AI-generate a custom tool from a natural language description.

### GET /api/tools/available

List all available tools (built-in + custom) that can be assigned to agents.

### GET /api/tools/examples

Get example custom tool definitions for reference.

## REST: Skills

### GET /api/skills

List all skills (markdown prompt fragments).

### POST /api/skills

Create a new skill.

### PUT /api/skills/:id

Update a skill's content or metadata.

### DELETE /api/skills/:id

Delete a skill.

### POST /api/skills/:id/clone

Clone an existing skill as a new skill.

### POST /api/skills/import

Import skills from a JSON payload.

### POST /api/skills/scan

Scan a directory for skill files and auto-import them.

### GET /api/skills/export

Export all skills as a JSON file.

## REST: Usage Analytics

### GET /api/usage/summary

Get a summary of total token usage and estimated costs.

### GET /api/usage/recent

List recent usage entries with timestamps.

### GET /api/usage/by-model

Get token usage broken down by LLM model.

### GET /api/usage/by-agent

Get token usage broken down by agent instance.

## REST: GitHub

### GET /api/settings/github

Get GitHub integration settings.

### PUT /api/settings/github

Update GitHub integration settings.

### POST /api/settings/github/ssh/generate

Generate a new SSH key pair for Git authentication.

### POST /api/settings/github/ssh/import

Import an existing SSH private key.

### POST /api/settings/github/ssh/test

Test SSH connectivity to GitHub.

### GET /api/settings/github/ssh/public-key

Get the public key for the current SSH key pair.

### DELETE /api/settings/github/ssh

Delete the current SSH key pair.

## REST: Messaging

Each messaging bridge has GET/PUT settings endpoints and a test endpoint.

### Discord

### GET /api/settings/discord

Get Discord bridge settings (bot token, channel ID, enabled status).

### PUT /api/settings/discord

Update Discord bridge settings.

### POST /api/settings/discord/test

Test the Discord bot connection.

### Slack

### GET /api/settings/slack

Get Slack bridge settings.

### PUT /api/settings/slack

Update Slack bridge settings.

### POST /api/settings/slack/test

Test the Slack app connection.

### Matrix

### GET /api/settings/matrix

Get Matrix bridge settings.

### PUT /api/settings/matrix

Update Matrix bridge settings.

### POST /api/settings/matrix/test

Test the Matrix homeserver connection.

### IRC

### GET /api/settings/irc

Get IRC bridge settings.

### PUT /api/settings/irc

Update IRC bridge settings.

### POST /api/settings/irc/test

Test the IRC server connection.

### Microsoft Teams

### GET /api/settings/teams

Get Teams bridge settings.

### PUT /api/settings/teams

Update Teams bridge settings.

### POST /api/settings/teams/pairing

Initiate Teams webhook pairing.

### Telegram

### GET /api/settings/telegram

Get Telegram bridge settings (bot token, enabled status).

### PUT /api/settings/telegram

Update Telegram bridge settings.

### POST /api/settings/telegram/test

Test the Telegram bot connection.

### WhatsApp

### GET /api/settings/whatsapp

Get WhatsApp bridge settings.

### PUT /api/settings/whatsapp

Update WhatsApp bridge settings.

### POST /api/settings/whatsapp/test

Test the WhatsApp bridge connection.

### Signal

### GET /api/settings/signal

Get Signal bridge settings.

### PUT /api/settings/signal

Update Signal bridge settings.

### POST /api/settings/signal/test

Test the Signal bridge connection.

### Mattermost

### GET /api/settings/mattermost

Get Mattermost bridge settings.

### PUT /api/settings/mattermost

Update Mattermost bridge settings.

### POST /api/settings/mattermost/test

Test the Mattermost connection.

### Nextcloud Talk

### GET /api/settings/nextcloud-talk

Get Nextcloud Talk bridge settings.

### PUT /api/settings/nextcloud-talk

Update Nextcloud Talk bridge settings.

### POST /api/settings/nextcloud-talk/test

Test the Nextcloud Talk connection.

### Tlon

### GET /api/settings/tlon

Get Tlon bridge settings.

### PUT /api/settings/tlon

Update Tlon bridge settings.

### POST /api/settings/tlon/test

Test the Tlon connection.

## REST: Coding Agents

### OpenCode

### GET /api/settings/opencode

Get OpenCode integration settings.

### PUT /api/settings/opencode

Update OpenCode integration settings.

### POST /api/settings/opencode/test

Test the OpenCode CLI availability.

### Claude Code

### GET /api/settings/claude-code

Get Claude Code integration settings.

### PUT /api/settings/claude-code

Update Claude Code integration settings.

### POST /api/settings/claude-code/test

Test the Claude Code CLI availability.

### Codex

### GET /api/settings/codex

Get Codex integration settings.

### PUT /api/settings/codex

Update Codex integration settings.

### POST /api/settings/codex/test

Test the Codex CLI availability.

## REST: Google (Calendar & Gmail)

### Calendar

### GET /api/calendar/calendars

List all calendars on the connected Google account.

### GET /api/calendar/events

List events, optionally filtered by calendar ID and date range.

### POST /api/calendar/events

Create a new calendar event.

### PUT /api/calendar/events/:id

Update an existing calendar event.

### DELETE /api/calendar/events/:id

Delete a calendar event.

### Gmail

### GET /api/gmail/labels

List all Gmail labels.

### GET /api/gmail/messages

List Gmail messages, optionally filtered by label or search query.

### GET /api/gmail/messages/:id

Get a specific Gmail message by ID.

### POST /api/gmail/send

Send a new email.

### POST /api/gmail/messages/:id/archive

Archive a Gmail message (remove from inbox).

## REST: MCP Servers

### GET /api/settings/mcp-servers

List all configured MCP servers.

```json
// Response
[
  {
    "id": "mcp_abc",
    "name": "My MCP Server",
    "transport": "stdio",
    "command": "/usr/local/bin/mcp-server",
    "args": ["--port", "3000"],
    "status": "running",
    "tools": ["tool_a", "tool_b"]
  }
]
```

### POST /api/settings/mcp-servers

Create a new MCP server configuration.

```json
// Request body
{
  "name": "My MCP Server",
  "transport": "stdio",
  "command": "/usr/local/bin/mcp-server",
  "args": ["--port", "3000"],
  "env": {}
}
```

### GET /api/settings/mcp-servers/:id

Get a specific MCP server's configuration and status.

### PUT /api/settings/mcp-servers/:id

Update an MCP server's configuration.

### DELETE /api/settings/mcp-servers/:id

Delete an MCP server configuration.

### POST /api/settings/mcp-servers/:id/start

Start an MCP server process.

### POST /api/settings/mcp-servers/:id/stop

Stop a running MCP server process.

### POST /api/settings/mcp-servers/:id/restart

Restart an MCP server process.

### POST /api/settings/mcp-servers/:id/test

Test connectivity to an MCP server.

### POST /api/settings/mcp-servers/:id/discover

Discover available tools from a connected MCP server.

### PUT /api/settings/mcp-servers/:id/tools

Update the list of allowed tools from an MCP server.

## REST: Backup & Restore

### GET /api/settings/backup

Download a full database backup.

### POST /api/settings/restore

Restore the database from an uploaded backup file.

## REST: Modules

### GET /api/modules

List all installed modules.

### POST /api/modules/install

Install a module from a GitHub repository URL.

### POST /api/modules/:id/toggle

Enable or disable an installed module.

### DELETE /api/modules/:id

Uninstall a module.

### POST /api/modules/webhooks/:moduleId/*

Webhook handler for module-registered endpoints (e.g., GitHub webhooks).

## REST: World Layout

### GET /api/world

Get the current world layout (all zones).

### POST /api/world

Create or update a zone in the world layout.

### DELETE /api/world/:zoneId

Delete a zone from the world layout.

## REST: Scheduled Tasks

### GET /api/scheduled-tasks

List all custom scheduled tasks.

### POST /api/scheduled-tasks

Create a new scheduled task.

### PUT /api/scheduled-tasks/:id

Update a scheduled task (interval, enabled status, prompt).

### DELETE /api/scheduled-tasks/:id

Delete a scheduled task.

## REST: Files

### GET /api/projects/:id/files

List files in a project's workspace directory.

### GET /api/projects/:id/files/read

Read the contents of a file in a project's workspace.

**Query Parameters**

`path` -- Relative file path within the project workspace

### GET /api/projects/:id/files/git

Get Git status information for a project's workspace.

## Socket.IO: Client --> Server

Events emitted by the client (browser) to the server. Most use an acknowledgment callback to return data.

### Chat

### ceo:message

Client --> Server

Send a message from the CEO (human) to the COO.

```json
// Payload
{ "content": "Build a REST API for...", "conversationId"?: "conv_1", "projectId"?: "proj_1" }

// Ack callback
{ "messageId": "msg_abc", "conversationId": "conv_1" }
```

### ceo:new-chat

Client --> Server

Start a new conversation, resetting the COO's context.

```json
// Ack callback
{ "ok": true }
```

### Conversations

### ceo:list-conversations

Client --> Server

List conversations, optionally filtered by project.

```json
// Payload (optional)
{ "projectId"?: "proj_1" }

// Ack: Conversation[]
```

### ceo:load-conversation

Client --> Server

Load a specific conversation's messages.

```json
// Payload
{ "conversationId": "conv_1" }

// Ack
{ "messages": BusMessage[] }
```

### Registry & Agents

### registry:list

Client --> Server

Get all agent templates from the registry.

```json
// Ack: RegistryEntry[]
```

### agent:inspect

Client --> Server

Get detailed information about a specific agent instance.

```json
// Payload
{ "agentId": "worker-abc" }

// Ack: Agent | null
```

### agent:activity

Client --> Server

Get an agent's message history and activity log.

```json
// Payload
{ "agentId": "worker-abc" }

// Ack
{ "messages": BusMessage[], "activity": AgentActivityRecord[] }
```

### Projects

### project:list

Client --> Server

List all projects.

```json
// Ack: Project[]
```

### project:get

Client --> Server

Get a single project by ID.

```json
// Payload
{ "projectId": "proj_1" }

// Ack: Project | null
```

### project:enter

Client --> Server

Enter a project context. Returns the project, its conversations, and Kanban tasks.

```json
// Payload
{ "projectId": "proj_1" }

// Ack
{ "project": Project, "conversations": Conversation[], "tasks": KanbanTask[] }
```

### project:conversations

Client --> Server

List conversations for a specific project.

```json
// Payload
{ "projectId": "proj_1" }

// Ack: Conversation[]
```

### project:delete

Client --> Server

Delete a project and cascade-delete all related data (tasks, conversations, agents, messages).

```json
// Payload
{ "projectId": "proj_1" }

// Ack
{ "ok": true, "error"?: "..." }
```

### project:create-manual

Client --> Server

Manually create a project (bypasses COO orchestration).

### Agents & Coding

### agent:stop

Client --> Server

Force-stop a running agent instance.

### codeagent:respond

Client --> Server

Send a response to a coding agent's prompt (user input during PTY session).

### codeagent:permission-respond

Client --> Server

Approve or deny a coding agent's permission request (file write, shell command).

### Terminal

### terminal:input

Client --> Server

Send keyboard input to a terminal PTY session.

### terminal:resize

Client --> Server

Resize a terminal PTY session (rows, cols).

### Memory & Soul

### memory:list

Client --> Server

List episodic memories, optionally filtered by search query.

### memory:save

Client --> Server

Save a new episodic memory.

### soul:list

Client --> Server

List all soul documents.

### soul:save

Client --> Server

Create or update a soul document.

## Socket.IO: Server --> Client

Events broadcast from the server to connected clients.

### Message Bus

### bus:message

Server --> Client

Broadcast of every message on the internal agent bus. Used by the frontend to show agent communication in real-time.

### COO Responses

### coo:response

Server --> Client

Complete COO response message (after streaming ends).

### coo:stream

Server --> Client

Token-by-token streaming from the COO agent. Emitted as each token is generated.

### coo:thinking

Server --> Client

Extended thinking (reasoning) tokens from the COO.

### coo:thinking-end

Server --> Client

Signals the end of a COO thinking block.

### coo:audio

Server --> Client

TTS audio synthesis result for the COO's response.

### Conversations

### conversation:created

Server --> Client

A new conversation was created.

### Projects

### project:created

Server --> Client

A new project was created by the COO.

### project:updated

Server --> Client

A project was updated (status change, charter update, etc.).

### project:deleted

Server --> Client

A project was deleted.

### Kanban

### kanban:task-created

Server --> Client

A new Kanban task was created.

### kanban:task-updated

Server --> Client

A Kanban task was updated (column change, assignment, completion report).

### kanban:task-deleted

Server --> Client

A Kanban task was deleted.

### Agents

### agent:spawned

Server --> Client

A new agent instance was created (Team Lead or Worker).

### agent:status

Server --> Client

An agent's status changed (idle, thinking, acting, done, error).

### agent:destroyed

Server --> Client

An agent instance was destroyed.

### agent:stream

Server --> Client

Token-by-token streaming from a worker agent.

### agent:thinking

Server --> Client

Extended thinking tokens from a worker agent.

### agent:thinking-end

Server --> Client

End of an agent's thinking block.

### agent:tool-call

Server --> Client

An agent invoked a tool (file_read, shell_exec, web_search, etc.).

### Admin Assistant

### admin-assistant:stream

Server --> Client

Token-by-token streaming from the Admin Assistant.

### admin-assistant:response

Server --> Client

Complete Admin Assistant response.

### Coding Agents

### codeagent:session-start

Server --> Client

A coding agent PTY session has started.

### codeagent:session-end

Server --> Client

A coding agent PTY session has ended.

### codeagent:event

Server --> Client

A coding agent lifecycle event (tool call, file change, etc.).

### codeagent:message

Server --> Client

A message from the coding agent (output text).

### codeagent:permission-request

Server --> Client

A coding agent is requesting permission for a file write or shell command.

### Terminal

### terminal:output

Server --> Client

Terminal PTY output data.

### World & Todos

### world:updated

Server --> Client

The world layout (zones) has been updated.

### todo:created

Server --> Client

A new todo item was created.

### todo:updated

Server --> Client

A todo item was updated.

### todo:deleted

Server --> Client

A todo item was deleted.

### reminder:triggered

Server --> Client

A scheduled reminder was triggered.

### Messaging Status

### messaging:status

Server --> Client

Status update for a messaging bridge (connected, disconnected, error).

## Data Types

### Agent

```json
{
  "id": string,
  "name": string,
  "role": "coo" | "team_lead" | "worker" | "admin_assistant" | "scheduler" | "module_agent",
  "status": "idle" | "thinking" | "acting" | "awaiting_input" | "done" | "error",
  "registryId": string,
  "projectId": string | null
}
```

### RegistryEntry

```json
{
  "id": string,
  "name": string,
  "description": string,
  "systemPrompt": string,
  "promptAddendum": string | null,
  "capabilities": string[],
  "defaultModel": string,
  "defaultProvider": string,
  "tools": string[],
  "builtIn": boolean,
  "role": "coo" | "team_lead" | "worker" | "admin_assistant" | "scheduler" | "module_agent",
  "modelPackId": string | null,
  "gearConfig": GearConfig | null,
  "clonedFromId": string | null,
  "createdAt": string
}
```

### Project

```json
{
  "id": string,
  "name": string,
  "description": string,
  "status": "active" | "completed" | "failed" | "cancelled",
  "charter": string | null,
  "charterStatus": "gathering" | "finalized",
  "createdAt": string
}
```

### KanbanTask

```json
{
  "id": string,
  "projectId": string,
  "title": string,
  "description": string,
  "column": "backlog" | "in_progress" | "done",
  "position": number,
  "assigneeAgentId": string | null,
  "createdBy": string | null,
  "completionReport": string | null,
  "labels": string[],
  "blockedBy": string[],
  "createdAt": string,
  "updatedAt": string
}
```

### Conversation

```json
{
  "id": string,
  "title": string | null,
  "projectId": string | null,
  "createdAt": string
}
```

### BusMessage

```json
{
  "id": string,
  "fromAgentId": string | null,
  "toAgentId": string | null,
  "content": string,
  "role": "user" | "assistant",
  "conversationId": string,
  "projectId": string | null,
  "timestamp": string
}
```

### Todo

```json
{
  "id": string,
  "title": string,
  "description": string | null,
  "status": "pending" | "in_progress" | "done",
  "priority": "low" | "medium" | "high",
  "dueDate": string | null,
  "createdAt": string,
  "updatedAt": string
}
```

### CustomTool

```json
{
  "id": string,
  "name": string,
  "description": string,
  "parameters": object,  // JSON Schema
  "code": string,
  "enabled": boolean,
  "createdAt": string,
  "updatedAt": string
}
```

### Skill

```json
{
  "id": string,
  "name": string,
  "description": string,
  "content": string,  // Markdown prompt fragment
  "tags": string[],
  "createdAt": string,
  "updatedAt": string
}
```

### Memory

```json
{
  "id": string,
  "content": string,
  "agentId": string | null,
  "embedding": number[],  // Vector embedding
  "createdAt": string
}
```

### SoulDocument

```json
{
  "id": string,
  "title": string,
  "content": string,
  "createdAt": string,
  "updatedAt": string
}
```
