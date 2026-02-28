# OtterBot Docs

Everything you need to run, configure, and extend OtterBot — the free, open-source
multi-agent AI assistant that runs entirely in Docker.

[Quick Start](getting-started.md){ .md-button } [API Reference](api.md){ .md-button } [GitHub](https://github.com/TOoSmOotH/otterbot){ .md-button }

## Explore the Docs

- **[Getting Started](getting-started.md)** — Prerequisites, Docker quick start, development setup, and environment variables.
- **[Architecture](architecture.md)** — System overview, agent hierarchy, message bus, workspace isolation, and monorepo layout.
- **[Agent System](agents.md)** — Agent roles, built-in templates, lifecycle, tool capabilities, and customization.
- **[Features](features.md)** — Voice, 3D Live View, web search, browser automation, Kanban boards, messaging bridges, coding agents, memory, and more.
- **[API Reference](api.md)** — REST endpoints, Socket.IO events, request/response schemas, and data types.

## At a Glance

| Stat | Description |
|------|-------------|
| 14 | Built-in Agent Templates |
| 100+ | REST API Endpoints |
| 50+ | Real-time Socket Events |
| 11 | Messaging Integrations |
| 40+ | Built-in Tools |
| 18 | LLM Providers |
| 4 | Search Providers |

## Quick Start

Get OtterBot running in seconds with Docker:

```bash
# Pull & run
$ docker pull ghcr.io/toosmooth/otterbot:latest
$ docker run -d -p 62626:62626 --name otterbot \
  -e OTTERBOT_DB_KEY=change-me-to-something-secret \
  --shm-size 256m \
  ghcr.io/toosmooth/otterbot:latest

# Open in your browser
$ open http://localhost:62626
```

For detailed setup instructions including development mode and environment configuration,
see the [Getting Started](getting-started.md) guide.
