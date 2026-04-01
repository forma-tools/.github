# Forma

**A unified CLI ecosystem for agentic development.**

Forma is a protocol and ecosystem that standardises CLI tools for consistent behaviour across agentic workflows. Every tool follows the same command structure, JSON output shape, exit codes, and authentication pattern - making them predictable, composable, and instantly usable by both humans and AI.

## The Protocol

```bash
<tool> <resource> <action> [--json] [--limit N] [--fields a,b,c]
```

- **Consistent output** - always `{data: [...], meta: {...}}`
- **Semantic exit codes** - 0 success, 2 auth required, 3 not found, 7 rate limited
- **Stream separation** - data to stdout, human output to stderr
- **Unified auth** - `<tool> auth login/status/logout` everywhere

## The Ecosystem

**106 capabilities** across four tiers:

| Tier | Count | Examples |
|------|-------|---------|
| **Forma CLIs** | 49 | `xero` `harvest` `ahrefs` `grok` `sentry` `slack` `figma` `elevenlabs` |
| **System Tools** | 49 | `ffmpeg` `pandoc` `jq` `rg` `duckdb` `imagemagick` |
| **Vendor CLIs** | 7 | `supabase` `vercel` `resend` `ksm` `clickhouse` |
| **MCP Servers** | 1 | `cloudflare-mcp` |

## Quick Start

```bash
# Discover tools
forma list

# Explore a tool
xero --help

# Get data as JSON
xero invoices list --status DRAFT --json

# Chain tools
harvest clients list --json | jq -r '.data[].name'
```

## Why Not MCP?

MCP loads every tool into context upfront. With 10+ tools, that's a context explosion. Forma tools cost **zero tokens until invoked** - discover with `forma list`, use on demand, chain via stdout.

## Links

- **[Forma Protocol](https://github.com/forma-tools/forma)** - specification, registry, and documentation
- **[AGENTS.md](https://github.com/forma-tools/forma/blob/main/AGENTS.md)** - AI assistant reference guide

---

*Built by [@0xDarkMatter](https://github.com/0xDarkMatter)*
