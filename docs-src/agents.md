# Agent System

OtterBot is a roster of independent agents. Each agent is a self-contained
**profile** — its own personality, model, memory, skills, credentials, and
schedule. This page covers agent roles, the COO, subagents, and Agent Studio.

## Agent Roles

Every agent has one of three roles:

| Role | Description |
|------|-------------|
| `coo` | The default coordinating agent. Created during onboarding, pinned at the top of the roster, and cannot be deleted. |
| `agent` | A top-level agent you create and tune yourself. |
| `subagent` | A temporary worker spawned by another agent for a focused task. |

There are no fixed "agent templates" — every agent beyond the COO is one you
create and configure.

## The COO

The COO (Chief Operating Officer) is the agent you talk to by default. It
answers you directly and coordinates the rest of the roster: when a request
suits another agent's specialty, the COO delegates it over the message bus and
relays the result.

The COO is configured by the first-run onboarding wizard and cannot be deleted
through the API.

## Creating Agents

Add agents from the roster's **New Agent** action, or via the API. Each new
agent gets its own profile directory, persona, model configuration, and
isolated database. Give different agents different chat and embedding models,
different skills, and different credentials.

Each agent is an isolated directory under `data/profiles/<id>/`:

```text
profile.json   # identity, role, model refs, transport, avatar, limits
SOUL.md        # persona / system-prompt identity block
agent.db       # conversations, messages, memories, skills, vectors
skills/        # per-agent markdown skills
```

## Subagents

Agents allowed to spawn subagents can create temporary workers for focused
research or delegated work. A subagent:

- inherits its parent's model and credentials by default,
- is tracked as a **subagent task** with a goal, status, and result summary,
- is temporary — it exists for the task and is cleaned up afterward.

Subagent tasks appear in the **Activity** view alongside live bus messages. Each
agent has a configurable subagent limit.

## Agent Studio

Agent Studio is the per-agent editor. It is organized into tabs:

| Tab | What you configure |
|-----|--------------------|
| Identity | Display name, role, email, avatar / model pack, transport |
| Persona | The agent's `SOUL.md` — its system-prompt personality |
| Model | Chat and embedding model references, and per-agent endpoints |
| Skills | View, create, import, install, and export markdown skills |
| Schedule | Cron prompts that run against the agent |
| Memory | Add, review, and delete the agent's long-term memories |
| Credentials | API keys, endpoints, and tokens, stored encrypted per agent |

See [Features](features.md) for skills, memory, scheduling, and the rest of the
UI in more detail.
