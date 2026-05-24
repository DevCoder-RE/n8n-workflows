# Agent Frosty — Implementation Plan v2

**Version**: 2.0  
**Previous**: [implementation-plan.md](implementation-plan.md)  
**Key Change**: Architecture revised from n8n-heavy to hybrid agentic stack (Hermes-Agent + Paperclip + Pi + AgentDB + OpenCode). n8n demoted to MCP-connected pipeline tool.  
**Discussion**: See [discussions.md](discussions.md) for full rationale.

---

## Core Insight

> You do not train a foundation model to become an expert in your domain.
> You *construct* expertise using structured knowledge, multimodal grounding, and agent orchestration.

This plan replaces Graphiti with **AgentDB** (150x faster vector search, HNSW indexing, hybrid search) and replaces the custom-built ReasoningBank + Swarm Orchestration with **Hermes-Agent** (built-in learning loop, subagent spawning, trajectory tracking, multi-platform delivery). **Paperclip** provides governance/budgets/dashboards. **n8n** is demoted to one of many MCP-connected pipeline tools.

---

## Four-Layer Architecture

```
PAPERCLIP (governance, budgets, org chart, dashboard)
    ↑
HERMES-AGENT (runtime, learning loop, subagents, cron, delivery)
    ↑
AGENTDB MEMORY LAYER (ontology + documents + diagrams + formulas)
    ↑
FOUNDATION MODEL (reasoning + multimodal via any provider)
```

---

## Phase 1: Knowledge Base — AgentDB Multi-Database

### Database Structure

```
.agentdb/
├── ontology.db       # Domain entities & relationships (hybrid search)
├── documents.db      # Papers, reports, transcripts (vector + metadata)
├── diagrams.db       # Diagram semantics (symbolic graphs as vectors)
├── formulas.db       # LaTeX/math embeddings
└── trajectories.db   # Hermes-Agent learning patterns (symlink/alias from ~/.hermes)
```

### Domain Ontology Schema

**Entities:**
- `Component` — physical or logical system parts
- `Formula` — mathematical expressions
- `Concept` — abstract domain ideas
- `Constraint` — limits, standards, tolerances
- `Method` — procedures, algorithms
- `FailureMode` — known failure patterns
- `Metric` — measurable quantities
- `Diagram` — visual artifacts
- `Paper` — research sources
- `Standard` — regulatory/compliance references

**Relationships:**
- `depends_on` — A depends on B
- `implements` — A implements B
- `constrains` — A constrains B
- `derived_from` — A derived from source B
- `validated_by` — A validated by evidence B
- `contradicts` — A contradicts claim B
- `illustrated_by` — A shown in diagram B
- `measured_by` — A measured by metric B

Each node includes: `version`, `validity_range`, `confidence_score`, `source_refs` for full provenance tracking.

---

## Phase 2: Ingestion Pipeline

| Input Type | Processing | Storage Target |
|---|---|---|
| Research papers | PDF → chunk → embed with metadata | `documents.db` |
| Architectural diagrams | Vision model (GPT-4V/Claude Sonnet) → JSON component graph | `diagrams.db` |
| Formulas | Math OCR (Nougat/Pix2Struct) → LaTeX → semantic meaning | `formulas.db` |
| Reports | NLP chunking + claim extraction | `documents.db` |
| Audio / Video | Whisper transcription → speaker roles → claims | `documents.db` |
| Images | Vision captioning + object graph | `diagrams.db` |

### Conflict Detection

When contradictory claims are detected during ingestion (`max_load = 50kN` vs `45kN`):
1. Create a `ConflictNode` in `trajectories.db`
2. Link both claims with full provenance (date, source, context)
3. Flag for human review OR let the ReasoningAgent surface uncertainty during report generation

This turns the system from passive database into **active knowledge validator**.

### AgentDB Configuration

```javascript
const ontologyDB = await createAgentDBAdapter({ dbPath: '.agentdb/ontology.db', quantizationType: 'scalar' });
const documentsDB = await createAgentDBAdapter({ dbPath: '.agentdb/documents.db', quantizationType: 'binary' });
const diagramsDB = await createAgentDBAdapter({ dbPath: '.agentdb/diagrams.db', quantizationType: 'scalar' });
const formulasDB = await createAgentDBAdapter({ dbPath: '.agentdb/formulas.db', quantizationType: 'scalar' });
// Hermes manages its own trajectory store at ~/.hermes/ — AgentDB trajectories.db is optional add-on
```

- **Binary quantization** (32x) for large document volumes
- **Scalar quantization** (4x) for high-precision ontology and formula queries
- **HNSW indexing** for sub-100µs search across all databases
- **MCP server** exposes AgentDB to Hermes-Agent as a tool

---

## Phase 3: Agent Architecture — Hermes-Agent Subagent Swarm

### Topology: Hermes Parent Agent → Spawned Subagents

The old Controller + Swarm Orchestration pattern is replaced by **Hermes-Agent's native subagent spawning**. The parent Hermes agent acts as the Controller, spawning isolated subagents for specialist tasks.

```
                         ┌────────────────────────────────────────┐
                         │      Hermes Parent Agent (Controller)   │
                         │  - Intent parsing                       │
                         │  - Workflow planning                    │
                         │  - Subagent lifecycle management        │
                         │  - Result aggregation                   │
                         │  - Learning loop integration            │
                         └────────────────┬───────────────────────┘
                                          │
                         spawns subagents via `hermes spawn` or MCP
                                          │
              ┌───────────────────────────┼───────────────────────────┐
              ▼                           ▼                           ▼
┌─────────────────────────┐  ┌─────────────────────────┐  ┌─────────────────────────┐
│   Diagram Analyst       │  │   Research Analyst      │  │   Reasoning Agent        │
│   (Hermes skill)        │  │   (Hermes skill)        │  │   (Hermes skill)         │
│                         │  │                         │  │                         │
│  vision → component     │  │  paper eval → claim     │  │  graph traversal         │
│  graph → ontology       │  │  extraction → citation  │  │  constraint checks       │
│  linking → diagrams.db  │  │  detection → docs.db   │  │  formula application     │
└───────────┬─────────────┘  └───────────┬─────────────┘  └───────────┬─────────────┘
            │                            │                            │
            └────────────────────────────┼────────────────────────────┘
                                         ▼
                          ┌───────────────────────────┐
                          │    Report Writer           │
                          │    (Hermes skill)          │
                          │                            │
                          │  formal output, citations, │
                          │  tone, multi-platform      │
                          └───────────────────────────┘
```

### Agent Descriptions (as Hermes Skills)

Each specialist agent is implemented as a **Hermes skill** — procedural memory that the parent agent loads when needed. Skills are stored in `~/.hermes/skills/agent-frosty/` and follow the [agentskills.io](https://agentskills.io) format.

**1. Ingestion Agent** (Hermes skill + Python/JS tool)
- Extracts structured knowledge from raw assets
- Updates AgentDB databases via MCP
- Flags conflicts during ingestion

**2. Diagram Interpretation Agent** (Hermes skill + vision LLM tool)
- Takes diagram images → produces component graph
- Links extracted components to ontology nodes
- Stores diagram semantics (load paths, connections), not pixels

**3. Research Analyst Agent** (Hermes skill + AgentDB MCP tool)
- Evaluates papers against existing knowledge
- Extracts claims with confidence scoring
- Detects contradictions with existing graph

**4. Reasoning Agent** (Hermes skill + AgentDB MCP tool)
- Traverses ontology graph via hybrid search
- Applies formulas from `formulas.db`
- Checks constraints and flags violations

**5. Report Authoring Agent** (Hermes skill + delivery tool)
- Produces formal outputs (technical, regulatory, research)
- Cites graph nodes with provenance
- Uses domain-specific tone and format

**6. Controller (Parent Hermes Agent)**
- Interprets user intent from ambiguous requests
- Plans workflow (which skills or subagents to invoke, what order, loops)
- Spawns subagents for parallel work
- Aggregates intermediate results
- Handles iterative refinement via Hermes' native loop

### Orchestration Flow

```
User Input (diagram + request)
   ↓
Hermes Parent Agent (intent → plan)
   ↓
Spawns Diagram Analyst subagent → component graph → diagrams.db query
   ↓
Spawns Research Analyst subagent → relevant papers/claims → documents.db query
   ↓
Spawns Reasoning Agent subagent → ontology traversal + constraint checks → formulas.db
   ↓
Parent Agent aggregates evidence bundle
   ↓
Spawns Report Writer subagent → structured output with citations
   ↓
Delivers via Hermes multi-platform gateway (CLI, Telegram, Slack, email, etc.)
```

### Subagent Invocation (Hermes-native)

```bash
# Parent Hermes agent spawns a specialist subagent
hermes spawn --skill agent-frosty/diagram-analyst --input diagram.png
hermes spawn --skill agent-frosty/research-analyst --paper paper.pdf
hermes spawn --skill agent-frosty/reasoning-agent --context evidence-bundle.json
hermes spawn --skill agent-frosty/report-writer --output-format technical-report
```

Or from within a conversation via the parent agent's tool calling:
```
/run agent-frosty/diagram-analyst with diagram.png
/run agent-frosty/report-writer format=compliance
```

### MCP-Tool Integration

AgentDB, n8n (for pipeline stages), and Pi (LLM abstraction) are exposed to Hermes-Agent via MCP:

```
Hermes Parent Agent
  ├─ MCP Tool: AgentDB (hybrid search, graph traversal)
  ├─ MCP Tool: n8n (diagram processing pipeline, document transformation)
  ├─ MCP Tool: Pi (alternative LLM routing — optional)
  └─ Tool: Reports (multi-platform delivery via Hermes gateway)
```

### Hybrid Retrieval Strategy

When a user asks "Analyze this diagram and produce a compliance report":
1. Parent agent receives intent → plans workflow
2. Spawns Diagram Analyst → parsed components
3. Graph traversal via AgentDB MCP tool: what standards apply? what formulas govern behavior?
4. Vector chunks retrieved: similar diagrams, prior reports
5. Evidence bundle assembled with MMR for diversity
6. Passes to Reasoning Agent → checks constraints → flags violations
7. Report Writer synthesizes final output
8. Delivered via Hermes gateway (CLI, Telegram, Slack, email, etc.)

---

## Phase 4: Expert Reasoning (No Fine-Tuning)

### Prompt Architecture

Every agent operates under a domain expert system prompt, enforced via Hermes' personality and system prompt system:

```
You are a domain expert in [FIELD].
You must:
- Ground all claims in provided sources
- Use formulas exactly as defined
- Reference diagrams explicitly
- Flag uncertainty or missing data
- Never hallucinate unstated assumptions
```

AgentDB output becomes the "textbook" the model reasons from.

### Learning Loop (Hermes-Native, replaces ReasoningBank)

Hermes-Agent's built-in learning loop replaces the custom ReasoningBank:

| Custom feature | Hermes replacement |
|---|---|
| Trajectory Tracking | Built-in: `~/.hermes/sessions/` with FTS5 search |
| Verdict Judgment | Built-in: Hermes judges task success autonomously |
| Memory Distillation | Built-in: Skills self-improve during use, periodic nudges |
| Dynamic Few-Shot Retrieval | Built-in: FTS5 session search with LLM summarization |
| Skill Creation | Built-in: Hermes creates skills from experience after complex tasks |
| Cross-Session Memory | Built-in: Honcho dialectic user modeling |

### Domain-Specific CoT Examples

Inject 3-5 solved examples into the Hermes agent's context file (`~/.hermes/context/agent-frosty.md`) showing how to:
1. Traverse the graph for multi-hop reasoning
2. Apply formulas from `formulas.db` to specific problems
3. Cite sources with proper provenance
4. Flag uncertainty when evidence is contradictory or missing

---

## Phase 5: Report Generation & Delivery

### Output Artifacts
- Technical reports
- Compliance analyses
- Research summaries
- Design reviews
- Contradiction reports (flagging conflicting sources)

### Multi-Platform Delivery (Hermes-Native)

Hermes-Agent delivers reports to any configured platform:

- **CLI**: Direct terminal output
- **Telegram**: Bot message with formatted report
- **Discord**: Channel message with embed
- **Slack**: Channel or DM
- **Email**: Formatted email body
- **WhatsApp/Signal**: Text message
- **Scheduled**: Via Hermes' built-in cron (`hermes cron add --schedule "0 9 * * 1" --deliver-to slack`)

### Report Quality Requirements
- All claims cite specific graph nodes
- Formulas rendered with proper LaTeX
- Diagrams referenced by symbolic graph ID
- Uncertainty explicitly flagged

---

## Phase 6: Evaluation

### Domain-Specific Evals

| Metric | Method | Target |
|---|---|---|
| Faithfulness | Automated citation check — every claim traceable to source node | 100% |
| Diagram accuracy | Human spot-check 10% of symbolic graph vs source image | 95%+ |
| Contradiction rate | % of queries where conflicting sources are flagged | 100% detection |
| Report quality | Expert human review against rubric (completeness, tone, accuracy) | Score ≥ 4/5 |

Without evals, you're flying blind.

### Leveraging Hermes Trajectory Infrastructure

Hermes' built-in trajectory generation and compression can be used for:
- Generating training data for fine-tuning (batch trajectory export)
- Analyzing agent behavior patterns
- Detecting recurring failure modes
- A/B testing different agent configurations

---

## Phase 7: Fine-Tuning (Optional, Later)

Fine-tune **only if**:
- Stylistic consistency needed in reports
- Structured output fidelity required
- Thousands of gold-standard examples available

Fine-tune:
- **Report Writer agent** only (LoRA / adapters)
- Not the reasoning agent
- Not the retrieval layer

Hermes' trajectory infrastructure provides the training data pipeline.

---

## Technology Stack

| Layer | Technology |
|---|---|
| Agent Runtime & Learning Loop | **Hermes-Agent** (NousResearch) — subagent spawning, built-in learning loop, skill system, FTS5 memory, trajectory tracking |
| Governance & Dashboard | **Paperclip** — org charts, budgets, goal alignment, audit logs, multi-company |
| LLM Abstraction (Optional) | **Pi** (earendil-works) — `@pi-ai` for multi-provider LLM API if Hermes' built-in routing is insufficient |
| Vector/Graph Knowledge | AgentDB (multi-database, hybrid search, HNSW, quantization, MCP) |
| Pipeline Stages (Optional) | n8n — MCP-connected tool for specific visual pipeline stages (diagram processing, document transformation) |
| Development Tool | OpenCode — build + plan agents for developing Agent Frosty itself via SPARC methodology |
| Vision | GPT-4V / Claude Sonnet (diagram → JSON symbolic graph) |
| Math OCR | Nougat / Pix2Struct |
| Audio Transcription | Whisper |
| Foundation Models | Any — Hermes supports OpenRouter (200+ models), Nous Portal, OpenAI, Anthropic, Google, local |
| Multi-Platform Delivery | Hermes-native (Telegram, Discord, Slack, WhatsApp, Signal, Email, CLI) |
| Scheduled Tasks | Hermes-native cron |
| Skill Format | agentskills.io open standard |

---

## Performance Targets

- **Vector search**: <100µs per query (HNSW indexing)
- **Pattern retrieval**: <1ms (AgentDB caching)
- **Batch ingestion**: 2ms per 100 vectors
- **Memory efficiency**: 4-32x reduction via quantization
- **Subagent handoff**: <1s via Hermes spawn
- **Learning improvement**: Automatic via Hermes skill creation trajectory

---

## Priority Order (Execution Roadmap)

### Week 1: Hermes-Agent Foundation
1. Install and configure Hermes-Agent (local or cloud runtime)
2. Configure model provider (Nous Portal, OpenRouter, or direct API keys)
3. Create Agent Frosty context file (`~/.hermes/context/agent-frosty.md`) with domain CoT examples
4. Install and configure AgentDB with MCP server for Hermes integration
5. Define domain ontology — list every entity type and relationship (schema contract)
6. Initialize AgentDB multi-database structure with quantization

### Week 2: Core Agent + Ingestion
7. Build minimal ingestion pipeline — process 5 papers end-to-end via Hermes skill
8. Create "Analyst Agent" as first Hermes skill: prompt + AgentDB MCP retrieval + answer
9. Measure citation accuracy on 20 test queries
10. Identify gaps: ontology, retrieval, or reasoning failures
11. Configure Paperclip basics (optional — can defer to later sprint)

### Week 3: Multimodal + Specialist Skills
12. Add Diagram Interpretation Hermes skill (vision → symbolic graph via LLM tool)
13. Add Research Analyst Hermes skill (paper evaluation, claim extraction)
14. Add Reasoning Agent Hermes skill (graph traversal, constraint checks)
15. Parent agent workflow: intent → spawn subagents → aggregate → report

### Week 4: Report Generation + Delivery
16. Add Report Writer Hermes skill with domain-specific templates
17. Configure multi-platform delivery (Telegram, Slack, or email)
18. Set up scheduled reports via Hermes cron
19. Run 50+ test queries, iterate on failures
20. Train Hermes on successful trajectories (automatic via built-in learning loop)

### Week 5+: Production + Polish
21. Add Paperclip governance layer (mission, budgets, dashboard)
22. Scale ingestion to full document corpus
23. Production deployment (Hermes supports Docker, Modal, Daytona, SSH)
24. Automated faithfulness evaluation
25. Consider LoRA fine-tuning for report style (if 1000+ examples available)

---

## Key Design Decisions

1. **AgentDB over Graphiti**: AgentDB provides vector search, hybrid search, HNSW, quantization in one stack — no separate Neo4j + Pinecone needed
2. **Hermes-Agent over custom ReasoningBank**: Hermes has built-in learning loop, trajectory tracking, skill creation, FTS5 memory — building this from scratch would take months
3. **Hermes subagents over Swarm Orchestration**: Hermes' native `spawn` replaces the custom LangGraph pipeline — less code, more capability
4. **Paperclip for governance**: Budgets, org charts, mission alignment, and audit are features Paperclip already provides
5. **n8n demoted to MCP tool**: No longer the orchestrator. Useful for specific pipeline stages where visual workflow construction adds value
6. **Pi as optional LLM abstraction**: Hermes already supports multi-provider LLM routing — Pi is only needed if finer-grained control is required
7. **OpenCode as development tool**: Build + plan agents for SPARC methodology during development
8. **No fine-tuning first**: Expertise comes from prompt architecture + graph constraints + tool-augmented reasoning
9. **Conflict detection at ingest**: Surface contradictions early; turn the system into an active knowledge validator
10. **MMR for diversity**: Maximal Marginal Relevance ensures the Reasoning Agent sees multiple perspectives
