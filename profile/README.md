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

## The Protocol

Forma is a **specification** before it's a toolset. Every CLI in the ecosystem implements the same 27-section contract, and that contract is what lets 240+ tools behave as a single coherent system instead of 240 one-off scripts.

```bash
<tool> <resource> <action> [OPTIONS]
```

One command grammar. One `{data, meta}` output envelope. stdout for data, stderr for humans — strictly separated. Semantic exit codes (`0` success · `2` auth · `3` not found · `4` validation · `5` conflict · `6` rate limit · `7` server). Deterministic `auth login/status/logout/use` across every tool. Self-describing at every level via `--help` and `describe`. Declared safety tier on every mutating operation. Caching with `--no-cache` and `--refresh` escape hatches. Lockfiles and 7-day dependency quarantine. SemVer discipline with an explicit stability contract.

### The 27 sections

| # | Section | What it covers |
|---|---------|----------------|
| 1 | **Philosophy** | Core principles — stdout sacred, stderr human, fail fast, help exhaustive |
| 2 | **Command Architecture** | `<tool> <resource> <action>` shape, noun-before-verb, consistent across the ecosystem |
| 3 | **Flags & Options** | Standard flags (`--json`, `--limit`, `--fields`), naming rules, type handling |
| 4 | **Output Specification** | The `{data, meta}` envelope, UTF-8, deterministic shape, stream separation |
| 5 | **Exit Codes** | Semantic 0–7: success, auth required, not found, validation, conflict, rate limit, server error |
| 6 | **Error Handling** | Error object shape, `retry-after` surfacing, timeout behaviour, partial-success rules |
| 7 | **Agent Safety** | Risk tiers, confirmation gates, destructive-op declarations, prompt-injection defence |
| 8 | **Help System** | `--help` at every level (tool/resource/action), example-first, machine-parseable headers |
| 9 | **Introspection** | `describe` subcommand emits capability manifests for agent discovery |
| 10 | **Skill Specification** | Embedding multi-step procedures inside tools as loadable skills |
| 11 | **Authentication** | `auth login/status/logout/use`, keyring primary, `.env` fallback, profile switching |
| 12 | **Data Conventions** | ISO-8601 dates, string IDs, currency objects, enum casing, null handling |
| 13 | **Filtering & Pagination** | `--limit`, `--from`, `--to`, `--status`, cursor-based pagination in `meta` |
| 14 | **Batch & Parallel** | `--batch` file input, `--workers` concurrency, raw passthrough for bulk ops |
| 15 | **Caching** | TTL-based defaults, `--no-cache`, `--refresh`, `meta.cache` visibility |
| 16 | **Project Structure** | Directory layout, `pyproject.toml` conventions, package naming |
| 17 | **Implementation Reference** | Shared boilerplate — error handlers, renderers, HTTP clients, credential wiring |
| 18 | **Testing Protocol** | Required test categories, manual verification checklist, compliance gates |
| 19 | **Anti-Patterns** | What breaks the contract — chatty stdout, colors in pipes, silent failures |
| 20 | **Compliance Checklist** | Minimum-viable and complete compliance gates for registry entry |
| 21 | **Versioning** | SemVer strict, 0.x stability contract, status lifecycle (alpha → stable → maintenance) |
| 22 | **Releases & Changelog** | Keep-a-Changelog format, tag conventions, release workflow |
| 23 | **Self-Update** | `<tool> update` mechanism, version check, rollback on failure |
| 24 | **Supply Chain Security** | Lockfiles required, 7-day dependency quarantine, vulnerability scanning |
| 25 | **README & GitHub Conventions** | README structure, badges, repo topics, default settings |
| 26 | **Filesystem Conventions** | Root discovery, workspaces, output paths, presets, cache locations |
| 27 | **Registry Specification** | Unified registry schema, collections, entry format, capability declarations |

### Why this matters

Without a protocol, 240 tools is 240 learning curves. With one, it's a single grammar plus a directory of resources — every hour spent learning Forma amortises across every tool you'll ever use in the ecosystem.

For AI agents, the protocol is the critical substrate. Autonomous tool chaining is impossible when tools disagree about how to report errors, paginate, or authenticate. Forma's contract is what lets an agent pipe `tool A → tool B → tool C` with full confidence about shapes, exit codes, and failure modes — no schema inference, no error-message parsing, no per-tool adaptation layer.

**Full specification**: [27 sections](https://github.com/forma-tools/forma/tree/main/docs/protocol) with implementation reference, testing requirements, and compliance checklists.

---

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
