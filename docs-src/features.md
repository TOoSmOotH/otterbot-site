# Features

What you can do with OtterBot today.

## Agent Roster

A left-rail roster lists every agent — the COO pinned first, all other agents
below — with live status dots, model labels, and a **New Agent** action. Select
an agent to chat with it or open its studio.

## Per-Agent Chat

Every agent has its own persistent conversation space with streaming responses.
Conversations and memory persist across sessions, so an agent remembers what
you told it last time.

## Agent Studio

A full per-agent editor with tabs for identity, persona, model, skills,
schedule, memory, and credentials. See
[Agent System → Agent Studio](agents.md#agent-studio).

## Memory

Each agent has its own database for conversations, memories, and vector
embeddings. Memory search is hybrid — full-text (FTS5) plus vector similarity
(`sqlite-vec`).

- Agents search memory during a turn and save new memories through tools.
- When a session ends, the agent summarizes the conversation, extracts facts,
  rebuilds its model of you, and can author a new skill.
- You can add, review, and delete memories yourself from Agent Studio.

## Skills

Skills are markdown instruction packs scoped to a single agent. They are merged
into that agent's prompt when relevant. You can:

- author a skill manually,
- import one from raw markdown or a URL,
- install one from the built-in catalog,
- export skills to share them.

Catalog skills are fetched from GitHub and scanned before being stored. Skill
files live in each agent's `skills/` directory.

## Scheduled Tasks

Give an agent cron prompts that run on a schedule — a daily summary, a periodic
check-in. Scheduled tasks are managed per agent in Agent Studio's Schedule tab
and fire through the orchestrator's scheduler.

## Message Bus & Activity View

Agents coordinate over a central message bus. The **Activity** view is a live
observer of that bus: every request, response, broadcast, and status update,
plus the list of subagent tasks with their goals and results.

## Subagents

Trusted agents can spawn temporary subagents for focused work. See
[Agent System → Subagents](agents.md#subagents).

## 3D Agent Scene

The **3D** view renders the whole roster as a shared scene — each agent a
character with a status ring — using bundled character packs.

## Model Providers

Each agent picks its own chat and embedding models. OtterBot uses the Vercel AI
SDK and supports:

| Provider | Notes |
|----------|-------|
| Anthropic | Cloud chat models; API key stored per agent |
| OpenAI | Cloud chat and embedding models; API key or ChatGPT OAuth |
| LM Studio | Local OpenAI-compatible endpoint (default `http://localhost:1234/v1`) |
| Ollama | Local OpenAI-compatible endpoint (default `http://localhost:11434/v1`) |

Mix and match — a fast local model for one agent, a frontier cloud model for
another.

## Transports

The agent-to-agent bus runs over a pluggable transport, chosen with
`AGENT_TRANSPORT`:

- **`local`** — in-process delivery (default).
- **`discord`** — agents communicate over a Discord channel.

## Integrations

Agents can connect to external services through per-agent credentials:

- **Discord** — channel transport and bot connector.
- **Slack** — workspace connector via Slack's Web API.
- **Email** — send mail over SMTP.
- **GitHub** — repository access via the GitHub API.

## Credentials & Encryption

Per-agent API keys, endpoints, and tokens are stored in the encrypted control
database and managed in Agent Studio's Credentials tab. The only secret kept in
`.env` is `OTTERBOT_DB_KEY`, which encrypts every database.

## Onboarding Wizard

On first run, a setup wizard configures your first agent — the COO — choosing a
provider, model, credentials, and personality. See
[Getting Started → First-Run Setup](getting-started.md#first-run-setup).
