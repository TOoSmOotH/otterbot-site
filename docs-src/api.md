# API Reference

The OtterBot server exposes a REST API and a Socket.IO interface, both served
from the daemon port (default `3001`).

## REST Endpoints

### Setup

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/setup-state` | Current onboarding state |
| `POST` | `/api/setup-state/complete` | Mark onboarding complete |

### Agents

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/agents` | List agent summaries |
| `POST` | `/api/agents` | Create an agent |
| `GET` | `/api/agents/:id` | Get an agent profile |
| `PATCH` | `/api/agents/:id` | Update an agent |
| `DELETE` | `/api/agents/:id` | Delete an agent (the COO cannot be deleted) |

### Memory

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/agents/:id/memories` | List an agent's memories |
| `POST` | `/api/agents/:id/memories` | Add a memory |
| `DELETE` | `/api/agents/:id/memories/:memId` | Delete a memory |
| `GET` | `/api/memories` | List memories across all agents |
| `GET` | `/api/user-profile` | Get the learned user profile |

### Skills

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/agents/:id/skills` | List an agent's skills |
| `POST` | `/api/agents/:id/skills` | Create a skill for an agent |
| `POST` | `/api/agents/:id/skills/install` | Install a catalog skill |
| `DELETE` | `/api/agents/:id/skills/:skillId` | Remove a skill |
| `GET` | `/api/skills` | List all skills |
| `GET` | `/api/skill-catalog` | The built-in skill catalog |
| `POST` | `/api/skills/import` | Import a skill from a URL or raw markdown |
| `GET` | `/api/skills/:id/export` | Export one skill |
| `GET` | `/api/skills/export-all` | Export all skills |

### Credentials

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/agents/:id/credentials` | Store encrypted credentials for an agent |

### Scheduled Tasks

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/agents/:id/scheduled-tasks` | List an agent's scheduled tasks |
| `POST` | `/api/agents/:id/scheduled-tasks` | Create a scheduled task |
| `DELETE` | `/api/scheduled-tasks/:taskId` | Delete a scheduled task |

### Bus & Activity

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/bus/messages` | Recent agent-to-agent bus messages (optional `?limit=`) |
| `GET` | `/api/subagent-tasks` | Subagent task records |

### Models & Settings

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/providers` | Available model providers |
| `POST` | `/api/provider-models` | List models served by a provider |
| `POST` | `/api/test-model` | Verify a model and credential combination |
| `GET` | `/api/settings/global` | Read global settings |
| `PUT` | `/api/settings/global` | Update global settings |

### Assets

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/model-packs` | 3D character / model packs |
| `GET` | `/api/scenes` | Scene configurations |
| `GET` | `/api/environment-packs` | 3D environment packs |

### OpenAI Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/auth/openai/status` | ChatGPT OAuth status |
| `POST` | `/api/auth/openai/login` | Begin ChatGPT OAuth sign-in |
| `POST` | `/api/auth/openai/signout` | Sign out of ChatGPT OAuth |

## Socket.IO Events

Chat happens over Socket.IO. Chat events carry an `agentId`.

### Client → Server

| Event | Description |
|-------|-------------|
| `chat:join` | Join an agent's conversation |
| `chat:message` | Send a message to an agent |
| `chat:close` | Close the chat session |

### Server → Client

| Event | Description |
|-------|-------------|
| `chat:joined` | Confirms the conversation was joined |
| `chat:stream` | A streaming response chunk |
| `chat:done` | The response is complete |
| `agent:status` | An agent's status changed |
| `bus:message` | A new agent-to-agent bus message |
