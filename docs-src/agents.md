# Agent System

OtterBot's multi-agent architecture uses a hierarchy of specialized AI agents, each with
distinct roles, tools, and capabilities.

## Agent Roles & Hierarchy

Every agent instance is created from a template in the **Agent Registry**. Each
template defines a role, which determines the agent's position in the hierarchy:

| Role | Count | Responsibility |
|------|-------|----------------|
| `coo` | Exactly 1 | Primary orchestrator. Receives CEO goals, creates projects, writes charters, delegates to Team Leads. Maintains conversation context per chat thread. |
| `team_lead` | 1 per project | Project manager. Breaks directives into Kanban tasks, spawns workers, monitors progress, reports results to the COO. |
| `admin_assistant` | Exactly 1 | Personal productivity agent. Manages todos, reads/sends email via Gmail, and manages Google Calendar events. Reports to the CEO alongside the COO. |
| `scheduler` | Exactly 1 | Background agent that runs recurring tasks on configurable intervals (e.g., daily summaries, periodic monitoring). Operates autonomously. |
| `worker` | 1 per task | Individual contributor. Executes a single task using its assigned tools. Reports completion back to its Team Lead. |
| `specialist_agent` | 1 per specialist | Extension agent spawned by a [Specialist](features.md#specialist-agents). Has access to an isolated knowledge store, custom tools, and data ingestion pipelines. Queries are routed through the COO. |

!!! info
    **The CEO is you.** The human user is always the CEO -- the top of the
    hierarchy. All work flows down from your goals and back up as results.

## Built-in Templates

OtterBot ships with 14 built-in agent templates. These cannot be modified or deleted, but
they can be [cloned and customized](#customizing-agents).

### Management Templates

#### COO

**Role:** COO

Chief Operating Officer. Receives goals from the CEO and delegates to Team Leads.

**Capabilities:** management, delegation, coordination

**Default Model:** claude-sonnet-4-5-20250929 (Anthropic)

#### Team Lead

**Role:** Team Lead

Manages a team of workers. Breaks directives into tasks and assigns them.

**Capabilities:** management, planning, coordination

**Default Model:** claude-sonnet-4-5-20250929 (Anthropic)

### Worker Templates

#### Coder

**Role:** Worker

Writes, edits, and debugs code across multiple languages.

**Capabilities:** code, typescript, python, debugging

**Tools:** file_read, file_write, shell_exec, install_package

#### Researcher

**Role:** Worker

Searches the web, reads content, and synthesizes findings.

**Capabilities:** research, analysis, summarization

**Tools:** web_search, web_browse, file_read

#### Reviewer

**Role:** Worker

Reviews code for quality, correctness, and best practices.

**Capabilities:** code-review, testing, quality

**Tools:** file_read

#### Writer

**Role:** Worker

Creates documentation, specifications, and written content.

**Capabilities:** writing, documentation, specs

**Tools:** file_read, file_write

#### Planner

**Role:** Worker

Designs architectures, plans implementations, and decomposes problems.

**Capabilities:** planning, architecture, decomposition

**Tools:** file_read, file_write

#### Security Reviewer

**Role:** Worker

Audits code for security vulnerabilities and compliance issues.

**Capabilities:** security, audit, vulnerability

**Tools:** file_read, shell_exec

#### Tester

**Role:** Worker

Writes and runs tests, identifies edge cases, and ensures quality.

**Capabilities:** testing, test-writing, qa, edge-cases

**Tools:** file_read, file_write, shell_exec

#### Browser Agent

**Role:** Worker

Navigates websites, scrapes content, fills forms, and interacts with web applications.

**Capabilities:** browser, web-scraping, form-filling, web-interaction

**Tools:** web_browse, file_read, file_write

#### OpenCode Coder

**Role:** Worker

Delegates coding tasks to the OpenCode CLI via a managed PTY session.

**Capabilities:** code, opencode, pty

**Tools:** codeagent_delegate, file_read

#### Claude Code Coder

**Role:** Worker

Delegates coding tasks to the Claude Code CLI via a managed PTY session.

**Capabilities:** code, claude-code, pty

**Tools:** codeagent_delegate, file_read

#### Codex Coder

**Role:** Worker

Delegates coding tasks to the OpenAI Codex CLI via a managed PTY session.

**Capabilities:** code, codex, pty

**Tools:** codeagent_delegate, file_read

#### Tool Builder

**Role:** Worker

Creates custom JavaScript tools at runtime that other agents can use.

**Capabilities:** tools, javascript, automation

**Tools:** tool_create, file_read, file_write, shell_exec

### Specialized Templates

#### Admin Assistant

**Role:** Admin Assistant

Personal productivity agent. Manages todos, reads/sends email, and manages calendar events.

**Capabilities:** todos, email, calendar, productivity

**Tools:** todo_manage, gmail_read, gmail_send, calendar_manage, memory_save

## Agent Lifecycle

Every agent instance transitions through a defined set of statuses:

```text
Idle -----> Thinking -----> Acting -----> Done
|                             |  |
|          (on error)         |  +--> Awaiting Input
+---------> Error <-----------+
```

| Status | Description |
|--------|-------------|
| `Idle` | Agent is created but not yet processing. Waiting for a message or task. |
| `Thinking` | Running LLM inference. Streaming tokens to the frontend via `agent:stream` or `coo:stream`. |
| `Acting` | Executing a tool call (file I/O, shell command, web browse, etc.). Tool invocations are broadcast via `agent:tool-call`. |
| `Awaiting Input` | Agent is waiting for user input (e.g., coding agent permission request or interactive prompt). |
| `Done` | Task completed successfully. Results reported to parent agent. |
| `Error` | An error occurred during thinking or acting. Error details logged. |

The **BaseAgent** class manages this lifecycle, including LLM streaming with
configurable timeouts (30s for first chunk, 120s between chunks) and a serialized message
queue to prevent concurrent processing.

## Tool Capabilities

Workers use tools to interact with the environment. Each tool is injected via a
**ToolContext** based on the agent's template configuration.

| Tool | Description | Used By |
|------|-------------|---------|
| `file_read` | Read file contents from the workspace | Coder, Researcher, Reviewer, Writer, Planner, Security Reviewer |
| `file_write` | Create or overwrite files in the workspace | Coder, Writer, Planner |
| `shell_exec` | Execute shell commands (respects `SUDO_MODE`) | Coder |
| `web_search` | Search the web using the configured provider | Researcher |
| `web_browse` | Navigate and extract content from web pages (Playwright) | Researcher |
| `install_package` | Install apt or npm packages in the container | Coder |
| `codeagent_delegate` | Delegate a coding task to an external CLI agent (OpenCode, Claude Code, or Codex) via PTY | OpenCode Coder, Claude Code Coder, Codex Coder |
| `tool_create` | Create a custom JavaScript tool at runtime, sandboxed via `isolated-vm` | Tool Builder |
| `todo_manage` | Create, list, update, and delete personal todo items | Admin Assistant |
| `gmail_read` | Read, search, and list Gmail messages and labels | Admin Assistant |
| `gmail_send` | Send, reply, archive, and label Gmail messages | Admin Assistant |
| `calendar_manage` | Create, update, delete, and list Google Calendar events | Admin Assistant |
| `memory_save` | Save an episodic memory for later semantic retrieval | Admin Assistant, COO |

!!! tip
    **Management agents** (COO and Team Lead) don't use tools directly. Instead,
    they delegate tasks to workers that have the right tools for the job.

## Customizing Agents

You can create custom agent templates through the Agent Registry. Custom templates let you
define specialized agents with tailored system prompts, capabilities, and tool sets.

### Registry Operations

- **List** -- View all templates (built-in + custom)
- **Create** -- Create a new custom template from scratch
- **Clone** -- Duplicate a built-in or custom template as a starting point
- **Update** -- Modify a custom template (built-in templates are immutable)
- **Delete** -- Remove a custom template (built-in templates cannot be deleted)
- **Search** -- Find templates by capability keyword

### RegistryEntry Fields

```json
{
  "id": "custom-api-tester",
  "name": "API Tester",
  "description": "Tests REST APIs...",
  "systemPrompt": "You are an API testing specialist...",
  "promptAddendum": null,
  "capabilities": ["testing", "api", "http"],
  "defaultModel": "claude-sonnet-4-5-20250929",
  "defaultProvider": "anthropic",
  "tools": ["shell_exec", "file_read", "web_browse"],
  "builtIn": false,
  "role": "worker",
  "modelPackId": null,
  "gearConfig": null,
  "clonedFromId": "builtin-coder"
}
```

The `gearConfig` field lets you assign a 3D model pack to an agent for the
[Live View](features.md#3d-live-view) visualization.

## Extended Thinking

OtterBot supports extended thinking (reasoning) for models that support it (e.g., Claude
Sonnet). When enabled, agents emit `coo:thinking` / `agent:thinking`
events with reasoning tokens before the final response.

- Thinking tokens are streamed in real-time to the frontend
- A `coo:thinking-end` / `agent:thinking-end` event signals the end of the reasoning block
- The final response follows as normal `coo:stream` / `agent:stream` tokens

This allows the UI to show the agent's reasoning process separately from its final output,
giving transparency into how decisions are made.
