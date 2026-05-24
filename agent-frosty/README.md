# Agent Frosty ❄️

**Expert intelligent agent system** — ingest domain knowledge (papers, diagrams, formulas, audio, video) and produce grounded, citation-backed technical reports.

> You do not train a foundation model to become an expert in your domain.
> You *construct* expertise using structured knowledge, graph-augmented retrieval, and a specialist committee of self-improving agents.

---

## Vision

Agent Frosty turns raw domain materials into expert-level technical reports without fine-tuning. It uses **Hermes-Agent** for runtime and self-improvement, **AgentDB** for high-performance knowledge storage, and **Paperclip** for governance. A committee of specialist agents (Diagram Analyst, Research Analyst, Reasoning Agent, Report Writer) collaborates under a parent Controller agent to produce grounded, citable outputs.

---

## Architecture

```
PAPERCLIP (governance, budgets, org chart, dashboard)
    ↑
HERMES-AGENT (runtime, built-in learning loop, subagent spawning, cron, multi-platform delivery)
    ↑
AGENTDB (domain ontology + documents + diagrams + formulas — vector + hybrid search)
    ↑
FOUNDATION MODEL (any provider: OpenAI, Anthropic, Google, local via Nous Portal or OpenRouter)
```

---

## Key Features

| Capability | How It Works |
|---|---|
| **Constructed expertise** | No fine-tuning. Expertise emerges from prompt architecture + graph constraints + tool-augmented reasoning |
| **Self-improving** | Hermes-Agent's built-in learning loop creates skills from experience, improves them during use, persists knowledge across sessions |
| **Multimodal ingestion** | Papers (PDF), diagrams (vision LLM → symbolic graph), formulas (math OCR → LaTeX), audio/video (Whisper → transcript) |
| **Provenance-first** | Every claim in every report is traceable to a source node in the knowledge graph |
| **Specialist agent committee** | Diagram Analyst, Research Analyst, Reasoning Agent, Report Writer — each implemented as a modular Hermes skill |
| **Conflict detection** | Contradictory claims are flagged at ingest time, surfaced in reports as uncertainty |
| **Multi-platform delivery** | Reports delivered via CLI, Telegram, Discord, Slack, WhatsApp, Signal, or Email — all through Hermes gateway |
| **Scheduled reports** | Built-in cron via Hermes for recurring analyses |
| **Governance** | Paperclip provides org charts, budgets, mission alignment, and full audit trails |
| **Model freedom** | Use any LLM provider — switch with a single command via Hermes |

---

## Technology Stack

| Layer | Technology |
|---|---|
| Agent Runtime & Learning Loop | [Hermes-Agent](https://github.com/nousresearch/hermes-agent) (NousResearch) |
| Governance & Dashboard | [Paperclip](https://github.com/paperclipai/paperclip) |
| Knowledge Storage | AgentDB (multi-database, HNSW, hybrid search, quantization) |
| LLM Abstraction (optional) | [Pi](https://github.com/earendil-works/pi) |
| Pipeline Workflows (optional) | n8n (via MCP) |
| Development Tool | [OpenCode](https://github.com/anomalyco/opencode) |
| Vision Models | GPT-4V, Claude Sonnet (diagram → symbolic graph) |
| Math OCR | Nougat, Pix2Struct |
| Audio Transcription | Whisper |
| Skill Format | [agentskills.io](https://agentskills.io) open standard |

---

## Quick Start

### 1. Install Hermes-Agent

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

### 2. Configure a Model Provider

```bash
hermes setup          # Run the setup wizard
hermes model          # Choose your LLM provider (OpenRouter, Nous Portal, OpenAI, etc.)
```

### 3. Install AgentDB

```bash
npx agentdb@latest init .agentdb/ontology.db --dimension 768
npx agentdb@latest init .agentdb/documents.db --dimension 1536
npx agentdb@latest init .agentdb/diagrams.db --dimension 768
npx agentdb@latest init .agentdb/formulas.db --dimension 768
npx agentdb@latest mcp   # Start MCP server for Hermes integration
```

### 4. Load the Agent Frosty Context

```bash
cp docs/context/agent-frosty.md ~/.hermes/context/agent-frosty.md
```

### 5. Start Chatting

```bash
hermes
```

Ask a domain question. The parent agent will call AgentDB via MCP, retrieve relevant knowledge, and produce a grounded answer.

---

## Agent Skills

Each specialist agent is a Hermes skill stored in `skills/agent-frosty/`:

| Skill | Role |
|---|---|
| `diagram-analyst` | Vision → component graph → ontology linking |
| `research-analyst` | Paper evaluation, claim extraction, contradiction detection |
| `reasoning-agent` | Graph traversal, formula application, constraint checking |
| `report-writer` | Structured report generation with citations |

Invoke a skill directly:

```bash
hermes /run agent-frosty/diagram-analyst with diagram.png
hermes /run agent-frosty/report-writer format=compliance
```

Or let the parent Controller agent orchestrate the full workflow automatically based on your request.

---

## Project Structure

```
agent-frosty/
├── README.md                  # This file
├── docs/
│   ├── agent-frosty.md        # Original concept and architecture review
│   ├── implementation-plan.md # v1 plan (n8n-heavy architecture)
│   ├── implementation-plan-v2.md # v2 plan (Hermes/Paperclip hybrid stack)
│   ├── prd.md                 # v1 product requirements (n8n-heavy)
│   ├── prd-v2.md              # v2 product requirements (current)
│   └── discussions.md         # Architecture decisions and trade-off analysis
├── skills/
│   └── agent-frosty/          # Hermes skills (agentskills.io format)
│       ├── diagram-analyst/
│       ├── research-analyst/
│       ├── reasoning-agent/
│       └── report-writer/
├── .agentdb/                  # AgentDB databases (auto-created)
│   ├── ontology.db
│   ├── documents.db
│   ├── diagrams.db
│   └── formulas.db
└── config/
    └── hermes-mcp.json        # Hermes MCP tool configuration for AgentDB
```

---

## MVP Roadmap

| Release | Timeline | Scope |
|---|---|---|
| **MVP** | Weeks 1-2 | Hermes-Agent + AgentDB + text ingestion + single Analyst skill + 20 test queries |
| **Release 2** | Weeks 3-4 | Multimodal ingestion (diagrams, formulas) + all specialist skills + multi-platform delivery |
| **Release 3** | Week 5+ | Paperclip governance + evaluation pipeline + production deployment + optional LoRA fine-tuning |

See [prd-v2.md](docs/prd-v2.md) for full release plan and [implementation-plan-v2.md](docs/implementation-plan-v2.md) for detailed tasks.

---

## Key Design Decisions

| Decision | Rationale |
|---|---|
| **Hermes-Agent over custom runtime** | Built-in learning loop, subagent spawning, trajectory tracking, FTS5 memory, cron, multi-platform delivery — months of custom development avoided |
| **AgentDB over Graphiti + Pinecone** | Single stack for vector search, hybrid search, HNSW, quantization — 150x faster than alternatives |
| **Paperclip for governance** | Org charts, budgets, mission alignment, and audit logs are features Paperclip already provides |
| **No fine-tuning first** | Expertise constructed via prompt architecture + graph constraints + tool-augmented reasoning. Fine-tuning only for Report Writer style later |
| **Hermes skills over custom agents** | Modular, reusable, self-improving units following the agentskills.io open standard |
| **n8n as MCP tool, not orchestrator** | n8n is useful for visual pipeline stages but unsuited for dynamic agent coordination |

Full discussion in [discussions.md](docs/discussions.md).

---

## Contributing

This project follows the **SPARC methodology** (Specification → Architecture → Refinement → Review → Completion) using OpenCode's build and plan agents.

1. Review the current docs and [discussions.md](docs/discussions.md) for active decisions
2. Check [prd-v2.md](docs/prd-v2.md) for prioritized epics and stories
3. Check [implementation-plan-v2.md](docs/implementation-plan-v2.md) for detailed tasks

---

## License

MIT

---

## Acknowledgments

- [Hermes-Agent](https://github.com/nousresearch/hermes-agent) by Nous Research — agent runtime with built-in learning
- [Paperclip](https://github.com/paperclipai/paperclip) — agent orchestration and governance
- [Pi](https://github.com/earendil-works/pi) — agent toolkit and LLM abstraction
- [OpenCode](https://github.com/anomalyco/opencode) — open source coding agent used to build this project
- [AgentDB](https://agentdb.ruv.io) — high-performance vector database
- [n8n](https://n8n.io) — workflow automation (MCP tool)
- [agentskills.io](https://agentskills.io) — open skill standard
