# Forma

[![GitHub](https://img.shields.io/badge/github-forma--tools%2Fforma-blue?logo=github)](https://github.com/forma-tools/forma)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![Tools](https://img.shields.io/badge/tools-199-orange.svg)](#available-tools)

> A unified CLI ecosystem for agentic development.

Forma is a protocol and ecosystem that standardises CLI tools for consistent behaviour across agentic workflows. Instead of wrestling with different output formats, flag conventions, and error handling patterns for every API, Forma tools follow a single protocol that makes them predictable and composable.

The core insight: AI assistants and power users need tools that behave consistently. When every CLI has its own quirks, automation breaks. Forma solves this by defining a protocol that covers command structure, JSON output shapes, exit codes, authentication patterns, and stream separation. Any tool that follows the protocol becomes instantly usable by both humans and AI.

The ecosystem includes 78 Forma CLIs for popular SaaS APIs, 72 vendor CLIs, 48 system tools, and an MCP server - 199 capabilities across 17 collections, tracked in a central registry. An always-on daemon with a persistent Claude session orchestrates the ecosystem, accumulates knowledge, and is accessible from anywhere via remote control.

## Why Forma?

### The Problem

Every SaaS API has its own CLI (or none at all). Each one:
- Has different flags (`-j` vs `--json` vs `--format json`)
- Outputs different shapes (arrays, objects, wrapped, raw)
- Uses different exit codes (or just 0/1)
- Mixes progress bars with data in stdout

This chaos breaks agentic workflows where AI assistants need to **discover**, **invoke**, and **chain** tools reliably.

### The Solution

Forma is a **protocol** + **ecosystem** + **daemon** that makes CLI tools:

| Property | Without Forma | With Forma |
|----------|---------------|------------|
| **Discovery** | Read docs, guess flags | `forma list`, then `<tool> --help` |
| **Output** | Random JSON shapes | Always `{data: [...], meta: {...}}` |
| **Chaining** | Hope it works | `tool1 --json \| jq '.data' \| tool2` |
| **Errors** | Parse error messages | Exit code 3 = not found, 2 = auth required |
| **Auth** | Different for each | Always `<tool> auth login/status/logout` |
| **Orchestration** | Manual | Always-on daemon with schedules, monitors, and memory |

### Why Not MCP?

MCP is powerful, but it has a scaling problem: **every registered tool consumes context tokens**. With 2-3 MCP servers, you can burn 20% of your context window before you've started working. Try running 10+ tools and you're out of room.

| Aspect | MCP | Forma |
|--------|-----|-------|
| **Context cost** | All tools loaded upfront | Zero until invoked |
| **10+ tools** | Context explosion | No problem |
| **Tool chaining** | Through the model | Direct stdout piping |
| **Discovery** | Model reads tool schemas | `forma list` + `--help` |

MCP is great for 2-3 frequently-used integrations. Forma is for ecosystems of 10+ tools where context efficiency matters.

### Who Is This For?

- **AI Assistants** (Claude Code, Codex, Gemini) - Discover and use tools without MCP overhead
- **Power Users** - Chain tools with jq, build automation scripts
- **Developers** - Ship CLIs that work in agentic workflows
- **Entrepreneurs** - Always-on daemon manages tools across workspaces (agencies, clients, personal)

## Architecture

```
+-----------------------------------------------------------------------------------+
|                        FORMA ECOSYSTEM (199 capabilities)                         |
+-----------------------------------------------------------------------------------+
|                                                                                   |
|  78 Forma CLIs        71 Vendor CLIs        48 System Tools       1 MCP          |
|  +---------+--+--+     +-----+-----+--+     +-----+-----+--+     +-----+        |
|  |xero|harv|..|  |     |gws|gh|wrng|  |     |jq|ffm|pan|  |     |cf-mcp|       |
|  +----+----+--+--+     +---+--+----+--+     +--+---+---+--+     +------+        |
|       |                      |                    |                    |          |
|       +----------+-----------+--------------------+--------------------+          |
|                  |                                                                |
|         +--------v--------+          +---------------------------+               |
|         | FORMA PROTOCOL  |          |     FORMA DAEMON (OS)     |               |
|         |                 |          |                           |               |
|         |  - --json       |          |  Kernel: process monitor  |               |
|         |  - --help       |          |  User space: per-workspace|               |
|         |  - exit codes   |          |    - SOUL.md (identity)   |               |
|         |  - auth pattern |          |    - memory/ (knowledge)  |               |
|         |  - {data, meta} |          |    - journal/ (activity)  |               |
|         |  - --no-cache   |          |    - persistent session   |               |
|         +--------+--------+          |    - schedules + monitors |               |
|                  |                   +-------------+-------------+               |
|    +-------------+-------------+                   |                              |
|    v                           v                   v                              |
|  +------------------+       +------------------+  +------------------+           |
|  |   AI Assistants  |       |   Power Users    |  |  Remote Control  |           |
|  | (Claude/Gemini)  |       |   (Shell/jq)     |  | (Phone/Browser)  |           |
|  +------------------+       +------------------+  +------------------+           |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

## Quick Start

```bash
# What tools are available?
forma list

# What can Xero do?
xero --help

# List draft invoices as JSON
xero invoices list --status DRAFT --json

# Pipe between tools
harvest clients list --json | jq -r '.data[].name' | head -5

# Check auth status
xero auth status --json
```

## The Daemon

An always-on orchestrator that monitors, schedules, and learns across the ecosystem. Follows an OS architecture: shared kernel for process monitoring, isolated user-space daemons per workspace.

### What It Does

- **Schedules** - Cron-triggered tasks (daily audits, weekly verification, monthly billing)
- **Monitors** - Condition-triggered reactions (uncommitted changes, credential expiry, registry drift)
- **Memory** - Accumulates knowledge as flat .md files. Patterns, facts, warnings, preferences. Never decays.
- **Journal** - Daily activity log with monthly archival
- **Persistent Session** - The daemon IS a Claude session, not just a spawner. Accumulates context across ticks.
- **Remote Control** - Drop in from your phone, tablet, or any browser via claude.ai/code

### Workspace Isolation

Each workspace gets its own daemon with its own identity, memory, and context. Like separate user accounts on the same OS.

```bash
forma daemon --workspace default install       # Personal daemon
forma daemon --workspace evolution7 install     # Agency daemon
forma daemon services                           # See all daemons
forma daemon health                             # Cross-workspace view
```

### Talk To It

```bash
forma daemon chat                  # Interactive session with remote control
forma daemon ask "morning briefing"  # Quick context-aware question
forma daemon status                # What's happening
forma daemon journal               # What happened today
forma daemon memory list           # What it's learned
```

### Agent Pipeline

Five specialised agent roles for autonomous tool development:

```
Scout --> Architect(s) --> Verifier(s) --> Inspector(s)
              ^                |               |
              +-- fix (max 3) -+---------------+

Warden (independent, on-demand registry custodian)
```

## Registry and Discovery

The Forma registry is the central mechanism for tool discovery. 199 tools across 17 collections.

### How It Works

1. **Registration**: Each tool declares itself in its `pyproject.toml` with a `[tool.forma]` section
2. **Discovery**: Run `forma list` to see all registered tools
3. **Exploration**: Run `<tool> --help` for any tool's commands and options
4. **Execution**: All tools follow the same patterns, so knowledge transfers

### Collections

| Collection | Description |
|------------|-------------|
| ai | LLM, search, speech synthesis |
| analytics | Performance, tracking, reporting |
| cms | Content management, ecommerce |
| comms | Messaging, email, voice, scheduling |
| crawl | Web scraping, crawling, change detection |
| crypto | Cryptocurrency data |
| data | Archival, research, bookmarks |
| devtools | Developer CLI tools - code search, analysis, version control |
| domains | Domain registration, DNS, WHOIS |
| finance | Accounting, payments, exchange rates |
| geo | Maps, places, spatial data, travel |
| infra | Cloud, deployment, secrets, CDN |
| marketing | SEO, PPC, content marketing |
| media | Audio, video, design tools |
| monitoring | Observability, alerting, incident management |
| osint | Open-source intelligence, reconnaissance |
| productivity | Business ops - PM, invoicing, CRM, scheduling |

## The Protocol

All Forma tools follow the [Forma Protocol](docs/protocol/00-index.md):

### Command Structure

```
<tool> <resource> <action> [OPTIONS] [ARGS]
```

### Standard Flags

| Flag | Purpose |
|------|---------|
| `--json` | Machine-readable output to stdout |
| `--help` | Show usage with examples |
| `--version` | Show version |
| `--limit N` | Limit results |
| `--quiet` | Suppress non-essential output |
| `--fields` | Comma-separated fields to include in JSON output |
| `--dry-run` | Preview mutation without executing |

### JSON Output Shape

**Lists:**
```json
{
  "data": [{"id": "1", "name": "Item 1"}, ...],
  "meta": {"count": 10, "timestamp": "2025-01-28T12:00:00Z"}
}
```

**Single items:**
```json
{
  "data": {"id": "1", "name": "Item 1", ...},
  "meta": {"timestamp": "2025-01-28T12:00:00Z"}
}
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | Auth required |
| 3 | Not found |
| 4 | Validation |
| 5 | Permission denied |
| 6 | Conflict |
| 7 | Rate limited |

### Stream Separation

```bash
# stdout = data only (JSON when --json)
# stderr = human output (progress, errors, tables)
xero invoices list --json 2>/dev/null | jq '.data[0]'
```

## Credential Management

Forma tools use `forma_auth` for unified credential storage:

- **OS Keyring** as primary storage (macOS Keychain, Windows Credential Manager)
- **`.env` file** fallback when keyring is unavailable
- **Environment variables** override for CI/testing

```bash
xero auth login          # Store credentials
xero auth status --json  # Check where credentials come from
xero auth logout         # Remove credentials
```

## Creating a New Tool

```bash
# Copy the template
copier copy templates/cli-tool ./my-tool

# Install in development mode
cd my-tool && uv pip install -e .

# Register with Forma
forma sync
```

### Minimum Requirements

1. **pyproject.toml** with `[tool.forma]` section
2. **CLI entry point** following the protocol (`{data, meta}` envelope, exit codes, `--json`)
3. **Auth commands** if auth required (`auth login/status/logout`)
4. **AGENTS.md** for AI assistant context

See the [Forma Protocol](docs/protocol/00-index.md) for the complete specification (27 sections).

## For AI Assistants

See [AGENTS.md](AGENTS.md) for discovery patterns, command structure, JSON output shapes, exit codes, and chaining examples.

## License

MIT
