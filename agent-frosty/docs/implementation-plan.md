# Agent Frosty — Implementation Plan

## Core Insight

> You do not train a foundation model to become an expert in your domain.
> You *construct* expertise using structured knowledge, multimodal grounding, and agent orchestration.

This plan replaces Graphiti with **AgentDB**, providing 150x faster vector search, HNSW indexing, hybrid search, and native ReasoningBank integration in a single stack.

---

## Three-Layer Architecture

```
FOUNDATION MODEL (reasoning + multimodal)
        ↑
AGENTDB MEMORY LAYER (ontology + documents + diagrams + formulas + trajectories)
        ↑
AGENT SWARM (Controller → Analysts → Reasoner → Writer)
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
└── trajectories.db   # ReasoningBank learning patterns
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

```typescript
const ontologyDB = await createAgentDBAdapter({ dbPath: '.agentdb/ontology.db', quantizationType: 'scalar' });
const documentsDB = await createAgentDBAdapter({ dbPath: '.agentdb/documents.db', quantizationType: 'binary' });
const diagramsDB = await createAgentDBAdapter({ dbPath: '.agentdb/diagrams.db', quantizationType: 'scalar' });
const formulasDB = await createAgentDBAdapter({ dbPath: '.agentdb/formulas.db', quantizationType: 'scalar' });
const trajectoriesDB = await createAgentDBAdapter({ dbPath: '.agentdb/trajectories.db', enableLearning: true });
```

- **Binary quantization** (32x) for large document volumes
- **Scalar quantization** (4x) for high-precision ontology and formula queries
- **HNSW indexing** for sub-100µs search across all databases

---

## Phase 3: Agent Swarm Architecture

### Topology: Hierarchical (Controller → Specialists)

```
                    ┌──────────────────────┐
                    │    Controller Agent   │
                    │  (intent parsing,     │
                    │   workflow planning,  │
                    │   result aggregation) │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
       ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
       │  Diagram     │ │  Research    │ │  Reasoning   │
       │  Analyst     │ │  Analyst     │ │  Agent       │
       │              │ │              │ │              │
       │ vision →     │ │ paper eval   │ │ graph        │
       │ component    │ │ claim        │ │ traversal    │
       │ graph →      │ │ extraction   │ │ constraint   │
       │ ontology     │ │ citation     │ │ checking     │
       │ linking      │ │ detection    │ │ formula      │
       └──────┬───────┘ └──────┬───────┘ │ application  │
              │                │         └──────┬───────┘
              └────────────────┼────────────────┘
                               ▼
                      ┌────────────────┐
                      │  Report Writer │
                      │  (formal       │
                      │   output,      │
                      │   citation,    │
                      │   tone)        │
                      └────────────────┘
```

### Agent Descriptions

**1. Ingestion Agent**
- Extracts structured knowledge from raw assets
- Updates AgentDB databases
- Flags conflicts during ingestion

**2. Diagram Interpretation Agent**
- Takes diagram images → produces component graph
- Links extracted components to ontology nodes
- Stores diagram semantics (load paths, connections), not pixels

**3. Research Analyst Agent**
- Evaluates papers against existing knowledge
- Extracts claims with confidence scoring
- Detects contradictions with existing graph

**4. Reasoning Agent**
- Traverses ontology graph via hybrid search
- Applies formulas from `formulas.db`
- Checks constraints and flags violations

**5. Report Authoring Agent**
- Produces formal outputs (technical, regulatory, research)
- Cites graph nodes with provenance
- Uses domain-specific tone and format

**6. Controller Agent (Meta-Agent)**
- Interprets user intent from ambiguous requests
- Plans workflow (which agents, what order, loops)
- Aggregates intermediate results
- Handles iterative refinement

### Orchestration Flow

```
User Input (diagram + request)
   ↓
Controller Agent (intent → plan)
   ↓
Diagram Analyst → component graph → diagrams.db query
   ↓
Research Analyst → relevant papers/claims → documents.db query
   ↓
Reasoning Agent → ontology traversal + constraint checks → formulas.db
   ↓
Controller (aggregate evidence bundle)
   ↓
Report Writer → structured output with citations
```

### Swarm Pipeline Implementation

```typescript
await swarm.pipeline([
  { stage: 'intent', agent: 'controller', task: 'Parse user request' },
  { stage: 'diagram', agent: 'diagram-analyst', after: 'intent' },
  { stage: 'research', agent: 'research-analyst', after: 'intent' },
  { stage: 'reasoning', agent: 'reasoning-agent', after: ['diagram', 'research'] },
  { stage: 'writing', agent: 'report-writer', after: 'reasoning' },
]);
```

### Hybrid Retrieval Strategy

When a user asks "Analyze this diagram and produce a compliance report":
1. Diagram parsed → detected components
2. Graph traversal via AgentDB hybrid search: what standards apply? what formulas govern behavior?
3. Vector chunks retrieved: similar diagrams, prior reports
4. Evidence bundle assembled with MMR for diversity
5. Controller passes bundle to Reasoning Agent

---

## Phase 4: Expert Reasoning (No Fine-Tuning)

### Prompt Architecture

Every agent operates under a domain expert system prompt:

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

### ReasoningBank Integration

- **Trajectory Tracking**: Every agent interaction is stored in `trajectories.db`
- **Verdict Judgment**: After each report, judge success (user feedback + automated checks)
- **Memory Distillation**: Successful trajectories → pattern reinforcement
- **Dynamic Few-Shot Retrieval**: At query time, retrieve most relevant gold-standard example from `trajectories.db` based on diagram type or question class
- **Confidence-Weighted Retrieval**: ExperienceCurator filters by `minConfidence: 0.8` for high-quality evidence

### Domain-Specific CoT Examples

Inject 3-5 solved examples into prompts showing how to:
1. Traverse the graph for multi-hop reasoning
2. Apply formulas from `formulas.db` to specific problems
3. Cite sources with proper provenance
4. Flag uncertainty when evidence is contradictory or missing

---

## Phase 5: Report Generation

### Output Artifacts
- Technical reports
- Compliance analyses
- Research summaries
- Design reviews
- Contradiction reports (flagging conflicting sources)

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

---

## Technology Stack

| Layer | Technology |
|---|---|
| Vector/Graph DB | AgentDB (multi-database, hybrid search, HNSW, quantization) |
| Learning | ReasoningBank (trajectory tracking, verdict judgment, memory distillation) |
| Agent Framework | Swarm Orchestration (hierarchical topology, pipeline execution) |
| Development Process | SPARC methodology (Specification → Architecture → Refinement → Review → Completion) |
| Vision | GPT-4V / Claude Sonnet (diagram → JSON symbolic graph) |
| Math OCR | Nougat / Pix2Struct |
| Audio Transcription | Whisper |
| Agent Runtime | LangGraph + FastAPI (cyclic graphs for iterative reasoning) |
| Diagrams → Symbolic | Vision model structured output: `{components: [...], relationships: [...], load_paths: [...], constraints: [...]}` |

---

## Performance Targets

- **Vector search**: <100µs per query (HNSW indexing)
- **Pattern retrieval**: <1ms (AgentDB caching)
- **Batch ingestion**: 2ms per 100 vectors
- **Memory efficiency**: 4-32x reduction via quantization
- **Agent coordination**: Sub-second handoff via Swarm Orchestration

---

## Priority Order (Execution Roadmap)

### Week 1: Foundation
1. Define domain ontology — list every entity type and relationship (schema contract)
2. Initialize AgentDB multi-database structure with quantization
3. Build minimal ingestion pipeline — process 5 papers + 3 diagrams end-to-end
4. Manually verify graph correctness

### Week 2: Core Agent
5. Create single "Analyst Agent" — prompt + AgentDB retrieval + answer
6. Measure citation accuracy on 20 test queries
7. Identify gaps: ontology, retrieval, or reasoning failures

### Week 3: Multimodal + Orchestration
8. Add Diagram Interpretation Agent (vision → symbolic graph)
9. Add Controller Agent (intent parsing, workflow planning, result aggregation)
10. Wire up full pipeline via Swarm Orchestration

### Week 4: Learning + Evaluation
11. Integrate ReasoningBank trajectory tracking and verdict judgment
12. Implement dynamic few-shot retrieval from `trajectories.db`
13. Build evaluation pipeline (faithfulness, diagram accuracy, contradiction detection)
14. Run 50+ test queries, iterate on failures

### Week 5+: Polish
15. Add Report Writer with domain-specific formatting
16. Consider LoRA fine-tuning for report style (if 1000+ examples available)
17. Scale ingestion to full document corpus
18. Production deployment (FastAPI + LangGraph)

---

## Key Design Decisions

1. **AgentDB over Graphiti**: AgentDB provides vector search, hybrid search, HNSW, quantization, and ReasoningBank in one stack — no need for Neo4j + Pinecone separately
2. **No fine-tuning first**: Expertise comes from prompt architecture + graph constraints + tool-augmented reasoning. Fine-tuning only for stylistic consistency later
3. **Controller meta-agent**: A linear pipeline fails on ambiguous requests. The Controller plans, loops, and aggregates
4. **Conflict detection at ingest**: Surface contradictions early; turn the system into an active knowledge validator
5. **MMR for diversity**: When retrieving evidence, Maximal Marginal Relevance ensures the Reasoning Agent sees multiple perspectives, not just the most similar
6. **Trajectory learning**: Every interaction improves the system via ReasoningBank. Over time, the system gets faster and more accurate without retraining the model
