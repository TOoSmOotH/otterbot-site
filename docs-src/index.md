# OtterBot Docs

Everything you need to run, configure, and extend OtterBot — the free,
open-source, local-first multi-agent AI workspace.

[Quick Start](getting-started.md){ .md-button } [API Reference](api.md){ .md-button } [GitHub](https://github.com/TOoSmOotH/otterbot){ .md-button }

## What Is OtterBot?

OtterBot is built around **agent profiles**. Instead of one assistant with a
handful of modes, you run a roster of independent agents — each with its own
personality, model, memory, skills, credentials, and scheduled tasks. A **COO**
agent coordinates the team, agents talk to each other over a message bus you
can watch live, and trusted agents can spawn subagents for focused work.

Everything runs on your own machine, and every database is encrypted at rest.

## Explore the Docs

- **[Getting Started](getting-started.md)** — Install the CLI, run the daemon, and configure your first agent.
- **[Architecture](architecture.md)** — Agent profiles, the orchestrator, runtimes, the message bus, and storage.
- **[Agent System](agents.md)** — Agent roles, the COO, subagents, and Agent Studio.
- **[Features](features.md)** — Roster, chat, memory, skills, the scheduler, integrations, and the 3D scene.
- **[API Reference](api.md)** — REST endpoints and Socket.IO events.

## At a Glance

| Aspect | Detail |
|--------|--------|
| Local-first | Runs on your own machine — no cloud required |
| Multi-agent | A roster of independent agent profiles |
| Encrypted | Every SQLite database is encrypted at rest |
| Model providers | Anthropic, OpenAI, LM Studio, Ollama |
| Per-agent | Personality, model, memory, skills, schedule, credentials |
| Observable | A live message bus and a 3D agent scene |

## Quick Start

Install the CLI from npm, start the daemon, and open the web UI:

```bash
npm install -g otterbot
otterbot start
otterbot web
```

The daemon serves the web UI on `http://localhost:3001`. On first run, a setup
wizard configures your first agent. See the [Getting Started](getting-started.md)
guide for CLI commands, running from source, and configuration.
