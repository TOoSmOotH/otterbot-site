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

Get OtterBot running with a single command:

=== "Linux / macOS"

    ```bash
    curl -fsSL https://otterbot.ai/install.sh | sh
    ```

=== "Windows (PowerShell)"

    ```powershell
    irm https://otterbot.ai/install.ps1 | iex
    ```

The installer checks for Docker, generates config files, pulls the image, and starts the container. Open **https://localhost:62626** when it finishes.

For manual Docker setup, development mode, and environment configuration,
see the [Getting Started](getting-started.md) guide.
