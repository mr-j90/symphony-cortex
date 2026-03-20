# 🧠 Symphony Cortex

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

Symphony Cortex is a single Postgres database with pgvector, fronted by an MCP server. That's it.

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│    Claude     │   │   ChatGPT    │   │    Cursor    │
│  (MCP native) │   │ (via plugin) │   │  (MCP native) │
└──────┬───────┘   └──────┬───────┘   └──────┬───────┘
       │                  │                  │
       └──────────┬───────┴──────────┬───────┘
                  │                  │
            ┌─────▼──────────────────▼─────┐
            │      Symphony Cortex MCP Server    │
            │   (search, store, classify)   │
            └─────────────┬────────────────┘
                          │
            ┌─────────────▼────────────────┐
            │   Postgres + pgvector         │
            │   (your knowledge, embedded)  │
            └──────────────────────────────┘
```

Every AI you connect sees the same context. Switch tools whenever you want. Your knowledge stays yours.

## What You Get

- **Semantic search** across everything you've captured — not keyword matching, meaning matching
- **Automatic classification** — entries are tagged and categorized on ingest
- **Cross-tool continuity** — context from a Claude conversation is available in Cursor five seconds later
- **Full ownership** — your data lives in a Postgres instance you control
- **Compound returns** — the system gets more valuable every day you use it, independent of any vendor

## Quick Start (≈45 minutes, no coding required)

### Prerequisites

- Docker and Docker Compose
- An embedding API key ([OpenAI](https://platform.openai.com), [OpenRouter](https://openrouter.ai), or local via Ollama)
- At least one MCP-compatible AI tool (Claude Desktop, Cursor, etc.)

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
MCP_SERVER_PORT=3100

# --- Provider keys (only set the one you're using) ---
OPENAI_API_KEY=sk-...              # if using openai
OPENROUTER_API_KEY=sk-or-...       # if using openrouter
OPENROUTER_MODEL=openai/text-embedding-3-small  # optional, defaults shown
```

### 2. Start the stack

```bash
docker compose up -d
```

This spins up:
- Postgres with pgvector extension
- The Symphony Cortex MCP server
- (Optional) Ollama for local embeddings

### 3. Connect your first AI tool

**Claude Desktop** — Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "symphony-cortex": {
      "command": "npx",
      "args": ["-y", "symphony-cortex-mcp"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/symphony_cortex"
      }
    }
  }
}
```

**Cursor** — Add the same MCP config in Cursor's settings under MCP Servers.

### 4. Verify it works

In Claude Desktop or your connected tool, try:

```
Store this in my Symphony Cortex: I prefer TypeScript for backend services
  and Vue/Nuxt for frontend work.
```

Then in a *different* tool:

```
Search my Symphony Cortex for my language preferences.
```

If the second tool returns your preference, you're live.

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
├── docker-compose.yml        # Full stack: Postgres, MCP server, (optional) Ollama
├── mcp-server/
│   ├── src/
│   │   ├── index.ts          # MCP server entry point
│   │   ├── tools/            # MCP tool definitions (store, search, classify)
│   │   ├── embeddings/       # Embedding provider abstraction
│   │   └── db/               # Postgres/pgvector queries
│   └── package.json
├── prompts/
│   ├── 01-memory-migration.md
│   ├── 02-spark.md
│   ├── 03-quick-capture.md
│   └── 04-weekly-review.md
├── sql/
│   └── init.sql              # Schema: entries, embeddings, tags
├── .env.example
└── README.md
```

### MCP Tools Exposed

| Tool | Description |
|---|---|
| `store_entry` | Save a knowledge entry with automatic embedding and classification |
| `search_entries` | Semantic search across all stored knowledge |
| `list_tags` | Browse existing categories and tags |
| `get_entry` | Retrieve a specific entry by ID |
| `delete_entry` | Remove an entry |
| `weekly_digest` | Generate a summary of recent captures |

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

- [x] Core MCP server with store/search/classify
- [x] Prompt kit (migration, spark, capture, review)
- [x] Docker Compose one-command setup
- [ ] Ollama embedding provider for fully local operation
- [ ] Web UI for browsing and managing entries
- [ ] Bulk import from Notion, Obsidian, Apple Notes
- [ ] Multi-user support with namespaced entries
- [ ] Automatic capture via Slack integration
- [ ] Scheduled weekly review delivery (email/Slack)

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

**Built by [Jordan](https://github.com/YOUR_USERNAME)** — part of the [Symphony](https://gosymphony.ai) ecosystem. Engineering manager, AI tooling builder, and believer that your context shouldn't be rented from a platform.
