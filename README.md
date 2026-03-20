# 🧠 Symphony Cortex - WIP

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/f4a21db0-bf4d-4cf8-ab57-7455b9dd2451" />

**One database. One protocol. Every AI tool you use.**

Symphony Cortex is a self-hosted knowledge system that gives you a single, portable memory layer across *all* your AI tools — Claude, ChatGPT, Cursor, Copilot, whatever ships next month. It uses [MCP (Model Context Protocol)](https://modelcontextprotocol.io) to expose your accumulated context through one open standard.

Type a thought, and seconds later it's embedded, classified, and queryable by meaning from any tool that speaks MCP.

No SaaS middlemen. No per-tool silos. ~$0.10–$0.30/month to run.

---

## The Problem

Every AI tool you use builds its own memory about you — but none of them talk to each other.

- **Claude** knows your coding style and project history
- **ChatGPT** knows your writing preferences and business context
- **Cursor** knows your codebase patterns

Switch tools, and you start from zero. Stay on one, and you're locked in by context you can't export. Every month you invest in a platform's memory makes it harder to leave.

Your note-taking apps weren't built for this either. Notion, Obsidian, and Apple Notes were designed for the human web — documents you read and organize yourself. The **agent web** needs something different: structured, embeddable, semantically searchable knowledge that machines can query on your behalf.

## The Solution

Symphony Cortex is a containerized API — a Postgres database with pgvector and an MCP-compatible server, deployed as a single Docker stack. You run it once, point your tools at it, and forget about it. Your AI tools are the ones calling Cortex, not you.

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│    Claude     │   │   ChatGPT    │   │    Cursor    │
│              │   │              │   │              │
└──────┬───────┘   └──────┬───────┘   └──────┬───────┘
       │                  │                  │
       │          MCP / REST API             │
       └──────────┬───────┴──────────┬───────┘
                  │                  │
       ┌──────────▼──────────────────▼──────────┐
       │         🐳 Docker Container            │
       │  ┌───────────────────────────────────┐ │
       │  │   Symphony Cortex API Server      │ │
       │  │   (MCP + REST • store • search)   │ │
       │  └───────────────┬───────────────────┘ │
       │                  │                     │
       │  ┌───────────────▼───────────────────┐ │
       │  │   Postgres + pgvector             │ │
       │  │   (your knowledge, embedded)      │ │
       │  └───────────────────────────────────┘ │
       └────────────────────────────────────────┘
```

You deploy the container. Your AI tools connect to it. Every tool sees the same knowledge. Switch tools whenever you want — your context stays yours.

## What You Get

- **Semantic search** across everything you've captured — not keyword matching, meaning matching
- **Automatic classification** — entries are tagged and categorized on ingest
- **Cross-tool continuity** — context from a Claude conversation is available in Cursor five seconds later
- **Full ownership** — your data lives in a Postgres instance you control
- **Compound returns** — the system gets more valuable every day you use it, independent of any vendor

## Quick Start (≈15 minutes)

### Prerequisites

- Docker and Docker Compose
- An embedding API key ([OpenAI](https://platform.openai.com), [OpenRouter](https://openrouter.ai), or local via Ollama)

### 1. Clone and configure

```bash
git clone https://github.com/YOUR_USERNAME/symphony-cortex.git
cd symphony-cortex
cp .env.example .env
```

Edit `.env` with your preferred settings:

```env
POSTGRES_PASSWORD=your_secure_password
EMBEDDING_PROVIDER=openai          # "openai", "openrouter", or "local" (ollama)
CORTEX_PORT=3100                   # port the API listens on

# --- Provider keys (only set the one you're using) ---
OPENAI_API_KEY=sk-...              # if using openai
OPENROUTER_API_KEY=sk-or-...       # if using openrouter
OPENROUTER_MODEL=openai/text-embedding-3-small  # optional, defaults shown
```

### 2. Deploy

```bash
docker compose up -d
```

That's it. Cortex is now running at `http://localhost:3100`. Two containers, one command:

- **symphony-cortex-api** — the MCP + REST server
- **symphony-cortex-db** — Postgres with pgvector

### 3. Connect your AI tools

Cortex exposes both an **MCP endpoint** (for MCP-native tools) and a **REST API** (for everything else).

**MCP tools (Claude Desktop, Cursor, Claude Code, etc.):**

Add to your MCP config:

```json
{
  "mcpServers": {
    "symphony-cortex": {
      "url": "http://localhost:3100/mcp"
    }
  }
}
```

**REST API (ChatGPT plugins, custom agents, scripts, etc.):**

```bash
# Store a memory
curl -X POST http://localhost:3100/api/entries \
  -H "Content-Type: application/json" \
  -d '{"content": "I prefer TypeScript for backends and Vue/Nuxt for frontend work."}'

# Search by meaning
curl http://localhost:3100/api/search?q=language+preferences
```

### 4. Verify it works

Store something from one tool:

```
Store this in Cortex: I prefer TypeScript for backend services
  and Vue/Nuxt for frontend work.
```

Search from a *different* tool:

```
Search Cortex for my language preferences.
```

If the second tool returns your preference, you're live.

### 5. Deploy remotely (optional)

To run Cortex on a home server, VPS, or always-on machine so all your devices can reach it:

```bash
# On your server
docker compose up -d

# From your tools, point to the server's IP/hostname
# MCP: "url": "http://your-server:3100/mcp"
# REST: http://your-server:3100/api/...
```

For production, put it behind a reverse proxy (Caddy, nginx) with HTTPS and add an API key for auth. See the [deployment guide](./docs/deployment.md) for details.

---

## Prompt Kit

Symphony Cortex ships with four prompts designed to make the system compound from day one. These aren't throwaway templates — they're the operational layer that turns a database into a second brain.

### 1. Memory Migration

Frontload your Symphony Cortex with context your AI tools already know about you. This prompt walks your AI through extracting its existing knowledge about you and storing it in Symphony Cortex — preferences, project context, decisions, patterns.

> **When to use:** Immediately after setup. Run once per AI tool you're migrating from.

### 2. Cortex Spark

Generates your personalized "First 20 Captures" list — the twenty things worth storing that will have the highest impact on your day-to-day AI interactions. Based on your role, tools, and workflow.

> **When to use:** Right after migration. Gives you a running start.

### 3. Quick Capture Templates

Pre-structured capture formats optimized for clean metadata extraction. Covers decisions, learnings, preferences, project context, and meeting notes. Designed so the MCP server can classify entries automatically without you thinking about taxonomy.

> **When to use:** Daily. These become your default input format.

### 4. Weekly Review

A synthesis prompt that queries your captures from the past seven days, surfaces patterns, flags forgotten threads, and suggests connections you missed. Think of it as a retrospective for your knowledge, not just your tasks.

> **When to use:** Weekly. 10 minutes. The compounding happens here.

All prompts are in the [`/prompts`](./prompts) directory.

---

## Architecture

```
symphony-cortex/
├── docker-compose.yml        # Full stack: API server + Postgres
├── Dockerfile                # Multi-stage build for the API server
├── api/
│   ├── src/
│   │   ├── index.ts          # Server entry point (MCP + REST)
│   │   ├── mcp/              # MCP protocol handler and tool definitions
│   │   ├── rest/             # REST API routes (/api/entries, /api/search)
│   │   ├── embeddings/       # Embedding provider abstraction (OpenAI, OpenRouter, Ollama)
│   │   └── db/               # Postgres/pgvector queries
│   └── package.json
├── prompts/
│   ├── 01-memory-migration.md
│   ├── 02-spark.md
│   ├── 03-quick-capture.md
│   └── 04-weekly-review.md
├── sql/
│   └── init.sql              # Schema: entries, embeddings, tags
├── docs/
│   └── deployment.md         # Production deployment guide
├── .env.example
└── README.md
```

### API Surface

Cortex exposes the same capabilities through two interfaces:

**MCP Tools** (for AI tools that speak MCP natively):

| Tool | Description |
|---|---|
| `store_entry` | Save a knowledge entry with automatic embedding and classification |
| `search_entries` | Semantic search across all stored knowledge |
| `list_tags` | Browse existing categories and tags |
| `get_entry` | Retrieve a specific entry by ID |
| `delete_entry` | Remove an entry |
| `weekly_digest` | Generate a summary of recent captures |

**REST Endpoints** (for everything else):

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/entries` | Store a new entry |
| `GET` | `/api/search?q=...` | Semantic search |
| `GET` | `/api/entries/:id` | Get a specific entry |
| `DELETE` | `/api/entries/:id` | Delete an entry |
| `GET` | `/api/tags` | List all tags |
| `GET` | `/api/digest?days=7` | Weekly digest |

### Data Model

Each entry stores:

- **Content** — the raw text of the knowledge
- **Embedding** — vector representation for semantic search (via pgvector)
- **Tags** — auto-generated classification labels
- **Category** — high-level type (preference, decision, learning, context, note)
- **Source** — which tool created the entry
- **Timestamps** — created and last accessed

---

## Embedding Providers

Symphony Cortex treats embeddings as a swappable layer. Pick whichever fits your priorities:

| Provider | Config Value | What You Get |
|---|---|---|
| **OpenAI** | `openai` | Battle-tested `text-embedding-3-small`. Lowest cost per token. Requires OpenAI API key. |
| **OpenRouter** | `openrouter` | Access 50+ embedding models through one API key — OpenAI, Cohere, Mistral, and more. Swap models without changing providers. Great if you want flexibility or prefer a single billing account. |
| **Ollama (local)** | `local` | Fully offline. No API keys, no network calls, no cost. Runs models like `nomic-embed-text` or `mxbai-embed-large` on your own hardware. Best for privacy-first setups. |

Switching providers is a one-line change in `.env`. Existing entries keep their embeddings — new entries use the new provider going forward. A re-embed utility is on the roadmap for bulk migration between providers.

---

| Component | Monthly Cost |
|---|---|
| Postgres (self-hosted via Docker) | $0.00 |
| OpenAI Embeddings (ada-002, ~1k entries/month) | $0.01–$0.05 |
| OpenRouter Embeddings (model-dependent, ~1k entries/month) | $0.01–$0.10 |
| VPS to host it (optional, smallest tier) | $0.00–$5.00 |
| **Local-only setup (Ollama embeddings)** | **$0.00** |

If you run it on a machine you already have (home server, Mac Mini, always-on desktop), the marginal cost is effectively zero with local embeddings.

---

## Roadmap

- [x] Containerized API server (MCP + REST)
- [x] Docker Compose one-command deployment
- [x] OpenAI and OpenRouter embedding support
- [x] Prompt kit (migration, spark, capture, review)
- [ ] Ollama embedding provider for fully local/offline operation
- [ ] API key authentication for remote deployments
- [ ] Web UI for browsing and managing entries
- [ ] Bulk import from Notion, Obsidian, Apple Notes
- [ ] Multi-user support with namespaced entries
- [ ] Automatic capture via Slack/webhook integration
- [ ] Scheduled weekly review delivery (email/Slack)
- [ ] Re-embed utility for migrating between embedding providers
- [ ] Pre-built Docker images on Docker Hub / GHCR

---

## Contributing

Contributions welcome. If you're building something on top of Symphony Cortex or have ideas for new MCP tools, open an issue or PR.

Areas where help is especially useful:

- Additional embedding providers (Cohere, Voyage, Mistral, etc.)
- Import scripts for other knowledge tools
- MCP tool refinements and new tool ideas
- Documentation and setup guides for specific AI tools

## License

MIT — use it however you want, keep your knowledge yours.

---
**Built by [Jordan](https://github.com/mrj-90)** — part of the [Symphony](https://gosymphony.ai) ecosystem. Engineering manager, AI tooling builder, and believer that your context shouldn't be rented from a platform.
