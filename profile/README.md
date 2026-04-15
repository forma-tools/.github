# Forma

> A unified CLI ecosystem for agentic development.

Forma is a protocol, ecosystem, and agent runtime that standardises CLI tools for consistent behaviour across agentic workflows. 240 tools, 44 AI models, 67 skills, one coherent system.

## What It Does

**For users**: One `forma setup` and you have an AI agent with 240 tools, a background daemon, and connections to Telegram/Slack/Discord. Ask it to check your invoices, monitor your services, or build you a daily news digest.

**For developers**: Build CLI tools that work in agentic workflows. Follow the protocol, get discovery, chaining, auth, and AI compatibility for free.

**For AI assistants**: Discover and invoke tools without MCP overhead. Zero context tokens until a tool is actually called.

## Quick Start

```bash
# Install
curl -fsSL https://raw.githubusercontent.com/forma-tools/forma/main/install.sh | bash

# Interactive setup (collections, model, personality, workspace)
forma setup

# Chat with your agent
forma daemon chat

# Or from Telegram
# (configured during setup)
```

## Architecture

```
User (CLI / Telegram / Slack / Discord)
    │
    ▼
┌──────────────────────────────────────────────────────┐
│  Forma Agent Engine                                  │
│                                                      │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │ 44 Models    │  │ 8 Built-in   │  │ 67 Skills  │ │
│  │ 8 Providers  │  │ Tools        │  │ On-demand  │ │
│  │ Zero deps    │  │ + CLI Tools  │  │ Procedures │ │
│  └──────┬──────┘  └──────┬───────┘  └─────┬──────┘ │
│         │                │                │         │
│         ▼                ▼                ▼         │
│  ┌──────────────────────────────────────────────┐   │
│  │  Agent Loop                                   │   │
│  │  System prompt (personality + context + memory)│   │
│  │  Tool calling → execute → feed back → loop    │   │
│  │  Approval gate (3-tier safety model)          │   │
│  └──────────────────────────────────────────────┘   │
│         │                                           │
│         ▼                                           │
│  ┌──────────────────────────────────────────────┐   │
│  │  240 CLI Tools (subprocess isolation)          │   │
│  │  forma/ (105 Forma CLIs)                      │   │
│  │  vendor/ (88 vendor CLIs)                     │   │
│  └──────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘
```

## Agent Engine

The agent is model-agnostic with zero external dependencies. Direct HTTP to each provider - no LiteLLM, no middleware.

### 44 Models, 8 Providers

| Provider | Models | Route |
|----------|--------|-------|
| **OpenAI** | GPT-5.4, GPT-4.1, o3, o4-mini | Direct API |
| **Anthropic** | Claude Opus 4.6, Sonnet, Haiku | Direct API |
| **Gemini** | 3.1 Pro, 3 Flash, 2.5 Pro/Flash | Direct API |
| **GLM** | GLM-5.1, GLM-5, GLM-4.7 | Direct (Z.AI Coding endpoint) |
| **OpenRouter** | DeepSeek R1, Llama 4, Mistral, +300 | Proxy |
| **Ollama** | Llama, Qwen, Gemma (local) | Local |
| **Kimi** | Kimi K2, Moonshot | Direct API |
| **Qwen** | Qwen Max, Plus, Turbo | Direct API |

Switch models: `forma model set gemini-3-flash`

### 8 Built-in Tools

| Tool | Purpose |
|------|---------|
| `forma_read_file` | Read files with line numbers |
| `forma_write_file` | Create/overwrite files |
| `forma_edit_file` | Find-and-replace edits |
| `forma_bash` | Execute shell commands |
| `forma_glob` | Find files by pattern |
| `forma_grep` | Search file contents |
| `forma_install_tool` | Install tools from registry |
| `forma_load_skill` | Load multi-step procedures |

Plus all installed CLI tools (weather, xero, linear, etc.)

### 67 Skills

Pre-built procedures for common tasks. The agent loads them on demand:

git-ops, python-ops, debug-ops, security-ops, docker-ops, react-ops, typescript-ops, testing-ops, perf-ops, auth-ops, ci-cd-ops, postgres-ops, refactor-ops, scaffold, and 53 more.

### Safety Model

Three-tier approval gate:
- **Low risk** (read, search): always allowed
- **Medium risk** (write, edit): session-approved
- **High risk** (bash, install): pattern-matched, dangerous commands blocked

Gateway users (Telegram/Slack) cannot run bash or write files.

## Credentials

All API keys stored in `~/.forma/keys.env` (chmod 600). No keyring dependency, no env var pollution.

```bash
forma auth set openai        # Store a key
forma auth list              # Show all configured providers
forma auth test anthropic    # Verify a key works
```

## The Daemon

Always-on background agent with schedules, monitors, memory, and journal.

```bash
forma daemon start --detach  # Start background agent
forma daemon chat            # Interactive session
forma daemon ask "morning briefing"
forma daemon status
```

Connect from Telegram, Slack, Discord, WhatsApp, or Signal.

## Setup

```bash
forma setup
```

9-step interactive wizard:
1. Choose collections (240 tools across 24 domains)
2. Download tools from GitHub
3. Install into isolated per-tool venvs
4. Choose AI model (44 options, featured picker)
5. Connect tool accounts (OAuth/API keys)
6. Set up workspace context
7. Choose agent personality (13 styles)
8. Explore use case suggestions
9. Start the background daemon

## The Protocol

All Forma CLIs follow the [Forma Protocol](docs/protocol/00-index.md):

```bash
<tool> <resource> <action> [OPTIONS]
```

- `--json` for machine-readable output
- `--help` for discovery
- Consistent exit codes (0=success, 2=auth required, 3=not found)
- `{data: [...], meta: {...}}` JSON envelope
- `<tool> auth login/status/logout` for credentials

## YAML Tool Plugins

Add tools without building a CLI:

```yaml
# ~/.forma/plugins/my-tool/forma-tool.yaml
name: my-tool
tools:
  - name: my_tool_search
    description: Search for things
    parameters:
      query: { type: string, required: true }
    command: "my-tool search {query} --json"
```

## For AI Assistants

See [AGENTS.md](AGENTS.md).

## License

MIT
