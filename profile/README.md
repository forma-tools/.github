# Forma

**A unified CLI ecosystem for agentic development.**

Forma standardises CLI tools so they all behave the same way. Same command structure. Same JSON output. Same exit codes. Same auth flow. One protocol across 130+ tools - instantly usable by both humans and AI agents.

## The Problem

Every SaaS API has its own CLI with its own quirks. Different flags, different output shapes, different error handling. This breaks automation. AI agents can't reliably chain tools when each one speaks a different language.

## The Solution

```bash
# Every Forma tool works the same way
<tool> <resource> <action> [--json] [--limit N] [--fields a,b,c]

# Predictable JSON output
{ "data": [...], "meta": { "count": 20 } }

# Semantic exit codes
# 0 = success, 2 = auth required, 3 = not found, 6 = rate limited

# Unified auth
<tool> auth login / status / logout
```

## The Ecosystem

**130 capabilities** organised into collections:

| Collection | Tools | What You Can Do |
|------------|-------|-----------------|
| **Productivity** | Xero, Harvest, Asana, Notion, HubSpot, Calendly, Shortcut | Invoicing, time tracking, project management, CRM, scheduling |
| **Marketing** | Ahrefs, SEMrush, Google Ads, Google Search Console | SEO audits, keyword research, PPC campaigns, search analytics |
| **Analytics** | GA4, GTM, GTmetrix, Looker | Web analytics, tag management, performance testing, reporting |
| **CMS** | WordPress, Shopify, Contentful, Sanity, Strapi, Storyblok, Payload, EmDash | Content management, ecommerce, headless CMS |
| **Infrastructure** | AWS Pricing, Supabase, Vercel, Netlify, Cloudflare, Doppler, 1Password | Cloud costing, deployment, secrets, CDN, edge computing |
| **Domains** | Namecheap, Porkbun, DNSimple, Name.com, dprobe | Domain registration, DNS management, WHOIS, reconnaissance |
| **Communications** | Slack, Gmail (Gwark), Twilio, Resend, Dialpad, Calendly | Messaging, email, SMS, voice, scheduling |
| **AI** | Grok, Perplexity, ElevenLabs | LLM chat, search-grounded AI, text-to-speech |
| **Data** | Firecrawl, CommonCrawl, Raindrop, Trove, Wikimedia | Web scraping, archival research, bookmarks |
| **Finance** | Xero, Harvest, Stripe, Currency | Accounting, payments, exchange rates |
| **Geo** | Mapbox, Mapillary, Nominatim, OpenStreetMap, Google Places | Geocoding, street imagery, spatial queries |
| **Media** | Figma, ElevenLabs, Crunch, FFmpeg | Design, audio, video optimisation |

**53 Forma CLIs** built to the protocol, **28 vendor CLIs** registered from official providers, **48 system tools** catalogued, **1 MCP server**.

## How It Works

```bash
# Discover what's available
forma list
forma list --collection productivity

# Explore any tool
xero --help
xero invoices list --help

# Get data as JSON - pipe to jq, feed to scripts
xero invoices list --status DRAFT --json | jq '.data[].total'

# Chain tools together
harvest clients list --json | jq -r '.data[].name'
CLIENT_ID=$(xero contacts search --name "Acme" --json | jq -r '.data[0].id')
xero invoices list --contact "$CLIENT_ID" --json

# Define infrastructure in YAML and cost it
aws-pricing configs cost production.yaml --json

# Generate client-ready quotes
aws-pricing configs quote production.yaml --format xlsx
```

## Why Not MCP?

MCP is powerful for 2-3 integrations. But every registered tool consumes context tokens. With 10+ tools, you burn your context window before you've started working.

| | MCP | Forma |
|---|---|---|
| **Context cost** | All tools loaded upfront | Zero until invoked |
| **10+ tools** | Context explosion | No problem |
| **Tool chaining** | Through the model | Direct stdout piping |
| **Discovery** | Model reads schemas | `forma list` + `--help` |
| **Offline** | Needs server running | Just CLI binaries |

Forma tools cost **zero tokens until invoked**. Discover with `forma list`, invoke on demand, chain via stdout. MCP is great for real-time integrations. Forma is for ecosystems of tools at scale.

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     WORKSPACES                                │
│  Org-specific: credentials, playbooks, context documents     │
├──────────────────────────────────────────────────────────────┤
│                     FORMA CORE                                │
│  Protocol · Registry · SDK · Agent Pipeline                  │
├──────────────────────────────────────────────────────────────┤
│                     CLI TOOLS                                 │
│  53 Forma CLIs · 28 Vendor CLIs · 48 System Tools            │
└──────────────────────────────────────────────────────────────┘
```

**Workspaces** group tools by organisation with shared credentials and automation playbooks. **Forma Core** provides the protocol specification, unified registry, and an async SDK for building multi-step workflows. **CLI Tools** are standalone binaries that follow the protocol.

## The Protocol

26 specification sections covering every aspect of CLI behaviour:

- **Command architecture** - `<tool> <resource> <action>` pattern
- **Output specification** - `{data, meta}` envelope, stream separation
- **Exit codes** - semantic codes 0-7 for scripting
- **Authentication** - `auth login/status/logout` with keyring storage
- **Agent safety** - input validation, prompt injection defence
- **Caching** - TTL-based with `--no-cache` and `--refresh`
- **README standard** - consistent badges, structure, examples
- **Supply chain security** - lockfiles, 7-day quarantine, vulnerability scanning

## SDK

The Forma SDK (v0.2) provides async primitives for building automation:

- **Async by default** - concurrent step execution via DAG dependencies
- **State persistence** - crash recovery, resume from last successful step
- **Retry policies** - exponential backoff with rate limit awareness
- **Structured results** - `ToolResult` with exit code semantics, no exception-driven flow
- **Input validation** - Pydantic models validated before execution

```python
from forma_sdk import Playbook, step, RETRY_RATE_LIMIT

class ClientOnboarding(Playbook):
    @step("Create billing contact", retry=RETRY_RATE_LIMIT)
    async def create_contact(self, client_name, email, **kw):
        result = await self.tools.xero("contacts", "create",
            name=client_name, email=email)
        result.raise_for_status()
        return {"contact_id": result.data["data"]["ContactID"]}
```

## Agent Pipeline

Five autonomous agent roles for building and maintaining tools at scale:

| Role | Purpose |
|------|---------|
| **Scout** | Evaluate build-or-buy decisions for new tools |
| **Architect** | Design and scaffold CLI tools |
| **Verifier** | Static protocol compliance checking |
| **Inspector** | Install, write tests, validate runtime |
| **Warden** | Registry custodian, ecosystem integrity |

Agents run headlessly in parallel, enabling batch tool creation with automated quality assurance.

---

*Built by [@0xDarkMatter](https://github.com/0xDarkMatter)*
