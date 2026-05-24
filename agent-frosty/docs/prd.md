# Agent Frosty — Product Requirements Document

**Status**: Draft  
**Version**: 1.0  
**Last Updated**: 2026-05-24

---

## 1. Product Overview

### 1.1 Vision

Agent Frosty is an **expert intelligent agent system** that ingests domain-specific multimodal knowledge (research papers, architectural diagrams, formulas, reports, audio, video) and produces grounded, citation-backed technical reports. It does not fine-tune a model to "know everything" — it constructs expertise through structured knowledge, graph-augmented retrieval, and a committee of specialized agents.

### 1.2 Core Principles

- **Constructed intelligence over fine-tuning** — expertise emerges from prompt architecture + graph constraints + tool-augmented reasoning
- **Provenance-first** — every claim must be traceable to a source node
- **Multimodal → symbolic** — diagrams, formulas, and audio are converted to structured representations, not stored as raw media
- **Agent specialization** — a committee of specialists beats a monolithic agent
- **Learn from every interaction** — ReasoningBank tracks trajectories, judges outcomes, and distills patterns

### 1.3 Target Users

- Domain experts who need to produce technical reports from research materials
- Engineers reviewing designs against standards/compliance
- Researchers synthesizing findings across papers, diagrams, and datasets
- Regulatory analysts producing compliance assessments

### 1.4 Success Metrics

| Metric | Target | Measurement |
|---|---|---|
| Faithfulness | 100% | Automated citation check — every claim traceable to source |
| Diagram accuracy | ≥95% | Human spot-check 10% of symbolic graphs vs source |
| Contradiction flagging | 100% | % of queries where conflicting sources are surfaced |
| Report quality | ≥4/5 | Expert human review against rubric |
| Query-to-report latency | <60s (MVP), <10s (full) | End-to-end timer |
| Knowledge scale | 100+ sources (MVP), 10K+ (full) | Source count in AgentDB |

---

## 2. Scope Definition

### 2.1 MVP Scope (Weeks 1-2)

The MVP proves the core loop: **ingest → store → retrieve → generate answer**.

**In scope:**
- AgentDB multi-database initialization (ontology + documents + trajectories)
- Domain ontology definition (entities, relationships, schema)
- Ingestion pipeline for research papers only (PDF → chunk → embed → store)
- Single Analyst Agent: prompt + AgentDB retrieval + text answer
- 20 test queries with citation accuracy measurement
- Manual ontology population (no automated extraction yet)

**Out of scope (MVP):**
- Diagram/multimodal ingestion
- Diagram Interpretation Agent
- Controller Agent
- ReasoningBank learning loop
- Evaluation pipeline
- Fine-tuning
- Audio/video processing
- Report Writer agent
- Conflict detection
- Production deployment

### 2.2 Full Project Scope

| Component | MVP | Full |
|---|---|---|
| Knowledge base | ontology.db + documents.db | + diagrams.db + formulas.db + trajectories.db |
| Ingestion | Research papers (text only) | Diagrams, formulas, audio, video, images |
| Agents | Analyst (single) | Diagram Analyst, Research Analyst, Reasoning Agent, Controller, Report Writer |
| Retrieval | Vector search | Hybrid search + graph traversal + MMR |
| Learning | None | ReasoningBank trajectory tracking + verdict judgment + distillation |
| Evaluation | Manual citation check | Automated faithfulness, diagram accuracy, contradiction detection |
| Multimodal | None | Vision → symbolic graph, math OCR, audio transcription |
| Fine-tuning | None | Report Writer only (LoRA, optional) |
| Deployment | Local | FastAPI + LangGraph production stack |

---

## 3. Epics, Stories, and Tasks

---

### Epic 1: Knowledge Foundation

**Description**: Establish the knowledge storage layer and domain ontology that underpins all agent reasoning.  
**Priority**: P0 (MVP)  
**Estimated Effort**: 3-4 days

#### Story 1.1: Domain Ontology Definition
*As a domain expert, I want a formal ontology so that the system understands entities and relationships in my field.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 1.1.1 | Define entity types | Specify all entity classes (Component, Formula, Concept, Constraint, Method, FailureMode, Metric, Diagram, Paper, Standard) | 2h |
| 1.1.2 | Define relationship types | Specify all relationship types (depends_on, implements, constrains, derived_from, validated_by, contradicts, illustrated_by, measured_by) | 2h |
| 1.1.3 | Define node properties | Schema for each entity: version, validity_range, confidence_score, source_refs, modality | 2h |
| 1.1.4 | Define edge properties | Schema for each relationship: relationship_type, extracted_from, context_snippet, is_validated | 2h |
| 1.1.5 | Create ontology document | Publish ontology as a formal reference document for the project | 1h |

#### Story 1.2: AgentDB Multi-Database Initialization
*As a developer, I want AgentDB databases initialized with proper quantization and indexing so that the system has high-performance storage.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 1.2.1 | Install AgentDB | Install agentdb package, verify CLI and API work | 1h |
| 1.2.2 | Initialize ontology.db | Create with scalar quantization, 768-dim, HNSW enabled | 1h |
| 1.2.3 | Initialize documents.db | Create with binary quantization, 1536-dim, HNSW enabled | 1h |
| 1.2.4 | Initialize trajectories.db | Create with enableLearning: true, scalar quantization | 1h |
| 1.2.5 | Add MCP server integration | Start AgentDB MCP server, add to Claude Code config | 1h |
| 1.2.6 | Write AgentDB adapter module | Singleton adapter factory that provides typed access to each database | 2h |

#### Story 1.3: Manual Ontology Population
*As a domain expert, I want to manually populate the ontology with initial entities and relationships so that the system has seed knowledge.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 1.3.1 | Seed entity creation script | Script to insert entities into ontology.db with CLI or API | 2h |
| 1.3.2 | Seed relationship creation script | Script to insert edges between entities | 2h |
| 1.3.3 | Populate 20 seed entities | Manually define 20 domain entities representing core concepts | 3h |
| 1.3.4 | Populate 40 seed relationships | Manually define relationships between seed entities | 3h |
| 1.3.5 | Verify graph integrity | Query ontology.db to confirm connectivity and correctness | 1h |

---

### Epic 2: Ingestion Pipeline

**Description**: Build the system that converts raw source materials into structured knowledge stored in AgentDB.  
**Priority**: P0 (MVP: text-only; Full: multimodal)  
**Estimated Effort**: 5-7 days (MVP text), +10 days (multimodal)

#### Story 2.1: Research Paper Ingestion (MVP)
*As a domain expert, I want to upload research papers so that their content is extracted, chunked, and stored in the document database.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 2.1.1 | PDF parsing module | Extract text from PDF using pdf-parse or similar | 2h |
| 2.1.2 | Section segmentation | Split PDF into sections (abstract, intro, method, results, conclusion) | 2h |
| 2.1.3 | Chunking strategy | Implement semantic chunking with configurable overlap | 2h |
| 2.1.4 | Embedding computation | Compute OpenAI embeddings for each chunk | 1h |
| 2.1.5 | Metadata extraction | Extract title, authors, date, citations from paper frontmatter | 2h |
| 2.1.6 | Document storage in AgentDB | Store chunk embeddings + metadata in documents.db | 2h |
| 2.1.7 | Ingestion CLI/script | Single command that takes a PDF path and runs the full pipeline | 2h |
| 2.1.8 | Batch ingestion support | Process 5-10 papers in a single run | 2h |

#### Story 2.2: Diagram Ingestion (Full Scope)
*As a domain expert, I want to upload architectural diagrams so that their components and relationships are extracted and linked to the ontology.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 2.2.1 | Vision model integration | Integrate GPT-4V or Claude Sonnet API with structured output | 3h |
| 2.2.2 | Diagram → component graph prompt | Prompt engineering for extracting labeled components and relationships | 4h |
| 2.2.3 | Symbolic graph storage | Store extracted component graph in diagrams.db | 2h |
| 2.2.4 | Ontology linking | Link extracted components to existing ontology entities | 3h |
| 2.2.5 | Text-diagram cross-reference | Link diagram elements to sentences/phrases in source papers | 3h |
| 2.2.6 | Batch diagram processing | Queue for processing diagrams at scale with cost management | 3h |

#### Story 2.3: Formula Ingestion (Full Scope)
*As a domain expert, I want to upload documents with mathematical formulas so that they are recognized and stored with semantic meaning.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 2.3.1 | Math OCR integration | Integrate Nougat or Pix2Struct for formula recognition | 4h |
| 2.3.2 | LaTeX extraction | Extract LaTeX from recognized formulas | 2h |
| 2.3.3 | Variable-to-ontology mapping | Link formula variables to ontology nodes (e.g., `F = ma` links F(force) → Metric, m(mass) → Component, a(acceleration) → Metric) | 4h |
| 2.3.4 | Formula storage in AgentDB | Store LaTeX + semantic meaning + variable mappings in formulas.db | 2h |

#### Story 2.4: Audio/Video Ingestion (Full Scope)
*As a domain expert, I want to upload lecture recordings and meeting transcripts so that spoken knowledge is captured.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 2.4.1 | Whisper transcription | Integrate Whisper API for audio-to-text | 2h |
| 2.4.2 | Speaker diarization | Identify speaker roles and segments | 3h |
| 2.4.3 | Claim extraction from transcripts | NLP pipeline for extracting factual claims from conversational text | 4h |
| 2.4.4 | Transcript storage in AgentDB | Store claims as document chunks with source tracking | 2h |

#### Story 2.5: Conflict Detection (Full Scope)
*As a domain expert, I want the system to detect contradictory claims during ingestion so that uncertainty is surfaced rather than silently merged.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 2.5.1 | Conflict detection algorithm | Implement comparison of new claims against existing graph for contradictions | 4h |
| 2.5.2 | ConflictNode schema | Define ConflictNode structure in trajectories.db linking both claims with provenance | 2h |
| 2.5.3 | Human review flagging | Generate report of conflicts requiring manual resolution | 2h |
| 2.5.4 | Confidence scoring | Assign confidence scores to conflicting claims based on source recency, authority | 3h |

---

### Epic 3: Agent Swarm

**Description**: Build the committee of specialized agents that interpret user requests, reason over knowledge, and produce reports.  
**Priority**: P0 (MVP: single agent), P1 (Full: swarm)  
**Estimated Effort**: 5-7 days (MVP), +15 days (full swarm)

#### Story 3.1: Single Analyst Agent (MVP)
*As a user, I want to ask a question and get an answer grounded in my domain knowledge so that I trust the response is based on my sources.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 3.1.1 | Agent system prompt | Write domain-expert system prompt with grounding rules | 2h |
| 3.1.2 | AgentDB retrieval integration | Agent performs hybrid search across documents.db and ontology.db | 3h |
| 3.1.3 | Evidence bundle assembly | Retrieved chunks are assembled into a prompt context with citation metadata | 2h |
| 3.1.4 | Answer generation | LLM call with context + prompt → grounded answer | 2h |
| 3.1.5 | Citation formatting | Format citations with source, section, and line references | 2h |
| 3.1.6 | 20 test queries | Run queries, measure citation accuracy, identify gaps | 3h |

#### Story 3.2: Controller Agent (Full Scope)
*As a user, I want the system to handle ambiguous or complex requests by planning the right workflow across multiple agents.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 3.2.1 | Intent parsing | Controller interprets user request and classifies intent (analysis, comparison, compliance check, etc.) | 4h |
| 3.2.2 | Workflow planner | Controller builds agent execution plan based on intent (parallel, sequential, conditional) | 4h |
| 3.2.3 | Swarm pipeline implementation | Wire Controller into Swarm Orchestration pipeline with stage definitions | 3h |
| 3.2.4 | Result aggregation | Controller merges outputs from multiple agents into unified evidence bundle | 3h |
| 3.2.5 | Iterative refinement | Controller detects incomplete results and re-queries agents with refined context | 4h |

#### Story 3.3: Diagram Interpretation Agent (Full Scope)
*As a user, I want to upload a diagram and have it automatically analyzed so that its components are available for reasoning.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 3.3.1 | Vision model integration | Integrate multimodal LLM (GPT-4V/Claude Sonnet) for diagram analysis | 3h |
| 3.3.2 | Component extraction prompt | Structured prompt: extract components, connections, labels, load paths | 4h |
| 3.3.3 | Ontology linking | Map extracted components to ontology nodes via hybrid search | 3h |
| 3.3.4 | Symbolic graph output | Produce structured JSON: `{components, relationships, load_paths, constraints}` | 2h |

#### Story 3.4: Research Analyst Agent (Full Scope)
*As a user, I want the system to evaluate research papers against existing knowledge so that new claims are validated or flagged.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 3.4.1 | Paper evaluation prompt | Agent prompt for evaluating papers: extract claims, assess evidence quality | 3h |
| 3.4.2 | Claim extraction | Extract discrete factual claims from paper sections | 3h |
| 3.4.3 | Graph comparison | Compare extracted claims against ontology for alignment or contradiction | 3h |
| 3.4.4 | Confidence scoring per claim | Score each extracted claim (source quality, methodology, citation count) | 2h |

#### Story 3.5: Reasoning Agent (Full Scope)
*As a user, I want the system to apply formulas and constraints from my domain knowledge so that reports include quantitative analysis.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 3.5.1 | Formula retrieval | Query formulas.db for relevant formulas based on problem context | 2h |
| 3.5.2 | Variable substitution | Map variables from formula to actual values from diagram/graph | 3h |
| 3.5.3 | Constraint checking | Check computed values against constraint nodes in ontology | 3h |
| 3.5.4 | Multi-hop graph traversal | Traverse ontology edges for multi-step reasoning chains | 4h |

#### Story 3.6: Report Writer Agent (Full Scope)
*As a user, I want the system to produce polished, structured reports in my domain's expected format.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 3.6.1 | Report template system | Define templates for technical reports, compliance analyses, research summaries | 4h |
| 3.6.2 | Citation rendering | Format graph node citations with full provenance in report body | 2h |
| 3.6.3 | Uncertainty flagging | Automatically flag unsupported claims or contradictory evidence in output | 2h |
| 3.6.4 | Domain tone/style configuration | Configurable tone (regulatory, academic, engineering) | 2h |
| 3.6.5 | Structured output formats | Support markdown, PDF, and JSON output | 3h |

---

### Epic 4: Expert Reasoning Without Fine-Tuning

**Description**: Implement the prompt architecture, ReasoningBank learning, and dynamic few-shot retrieval that make the system behave like a domain expert without modifying the underlying model.  
**Priority**: P1 (Full Scope)  
**Estimated Effort**: 8-10 days

#### Story 4.1: Domain-Specific Prompt Architecture
*As a developer, I want every agent to operate under enforced domain-expert constraints so that outputs are grounded, citable, and precise.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 4.1.1 | Base expert prompt template | Define the grounding rules shared across all agents | 2h |
| 4.1.2 | Domain-specific CoT examples | Write 3-5 solved examples showing graph traversal, formula application, citation | 4h |
| 4.1.3 | Per-agent prompt specialization | Customize base prompt for each agent's role (Analyst, Reasoner, Writer) | 3h |
| 4.1.4 | Evidence bundle format | Standardized format for passing retrieval context into agent prompts | 2h |

#### Story 4.2: ReasoningBank Trajectory Tracking
*As a developer, I want every agent interaction to be tracked so that the system learns from successful outcomes.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 4.2.1 | Trajectory schema definition | Define trajectory structure: task, steps, outcomes, metrics | 1h |
| 4.2.2 | Trajectory recording hook | Middleware that records each agent action to trajectories.db | 3h |
| 4.2.3 | Verdict judgment implementation | After report delivery, assess success (user feedback + automated checks) | 4h |
| 4.2.4 | Memory distillation | Consolidate successful trajectories into high-level patterns | 3h |

#### Story 4.3: Dynamic Few-Shot Retrieval
*As a user, I want the system to use past successful analyses as examples when handling my request so that quality improves over time.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 4.3.1 | Gold-standard example storage | Store high-quality past analyses in trajectories.db with embedding | 2h |
| 4.3.2 | Query-time example retrieval | At query time, find most relevant past example by similarity | 3h |
| 4.3.3 | Example injection into prompt | Inject retrieved example as few-shot demonstration | 2h |
| 4.3.4 | Confidence-weighted retrieval | ExperienceCurator filters by minConfidence: 0.8 | 2h |

---

### Epic 5: Evaluation Pipeline

**Description**: Build the automated and human-in-the-loop evaluation system that measures expertise quality.  
**Priority**: P1 (Full Scope)  
**Estimated Effort**: 5-7 days

#### Story 5.1: Automated Faithfulness Check
*As a developer, I want every claim in the output to be automatically verified against source nodes so that hallucination is detected.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 5.1.1 | Claim extraction from output | Parse report to extract discrete factual claims | 3h |
| 5.1.2 | Source node lookup | For each claim, query AgentDB for supporting source nodes | 3h |
| 5.1.3 | Citation validation | Verify that cited sources actually contain the claimed information | 3h |
| 5.1.4 | Faithfulness scoring | Compute percentage of claims with verifiable source | 2h |

#### Story 5.2: Diagram Accuracy Evaluation
*As a domain expert, I want the system's diagram interpretation to be validated so that symbolic graphs match source images.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 5.2.1 | Human review interface | Simple UI or CLI for presenting diagram → symbolic graph pairs for review | 4h |
| 5.2.2 | Sampling strategy | Random 10% sample of all processed diagrams for review | 1h |
| 5.2.3 | Accuracy scoring | Calculate pass/fail rate per reviewer judgment | 2h |

#### Story 5.3: Report Quality Rubric
*As a domain expert, I want a formal rubric for evaluating report quality so that system improvements are measurable.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 5.3.1 | Define rubric dimensions | Completeness, tone, technical accuracy, citation quality, uncertainty handling | 2h |
| 5.3.2 | Scoring scale definition | 1-5 scale with anchored descriptors per dimension | 2h |
| 5.3.3 | Expert review collection | Process for domain experts to score 20-30 sample reports | 3h |
| 5.3.4 | Trend tracking | Dashboard or log showing quality scores over time | 2h |

#### Story 5.4: Contradiction Detection Evaluation
*As a developer, I want to measure how well the system surfaces conflicting information so that we can improve detection.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 5.4.1 | Test suite of known contradictions | Curate 20 queries where source materials contain known conflicts | 3h |
| 5.4.2 | Detection rate tracking | Run test suite, measure % of contradictions surfaced in output | 2h |
| 5.4.3 | False positive monitoring | Track cases where system flags non-contradictory information | 2h |

---

### Epic 6: Orchestration & API

**Description**: Build the runtime infrastructure that connects all components into a coherent system.  
**Priority**: P1 (Full Scope)  
**Estimated Effort**: 7-10 days

#### Story 6.1: Swarm Orchestration Pipeline
*As a developer, I want the agents to execute in a coordinated pipeline so that complex requests are handled reliably.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 6.1.1 | Swarm initialization | Initialize hierarchical topology with all agent types | 2h |
| 6.1.2 | Pipeline stage definitions | Define each stage: intent → diagram → research → reasoning → writing | 3h |
| 6.1.3 | Inter-agent memory sharing | Shared memory for passing intermediate results between stages | 3h |
| 6.1.4 | Fault tolerance | Retry logic, timeout handling, graceful degradation | 3h |
| 6.1.5 | Load balancing | Automatic work distribution across agents | 2h |

#### Story 6.2: FastAPI Backend
*As a developer, I want a REST API so that the system can be called programmatically.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 6.2.1 | API endpoint: ingest document | POST endpoint for uploading papers/diagrams/media | 3h |
| 6.2.2 | API endpoint: query | POST endpoint for submitting questions with optional diagram attachment | 3h |
| 6.2.3 | API endpoint: report status | GET endpoint to check generation progress | 2h |
| 6.2.4 | API endpoint: knowledge explorer | GET endpoint to browse ontology graph | 3h |
| 6.2.5 | Authentication | API key or JWT-based auth | 3h |
| 6.2.6 | Rate limiting | Prevent abuse of expensive agent calls | 2h |

#### Story 6.3: LangGraph Runtime (Optional)
*As a developer, I want cyclic agent graphs so that the system can handle iterative reasoning and self-correction.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 6.3.1 | State graph definition | Define LangGraph state with agent outputs and evidence bundle | 4h |
| 6.3.2 | Conditional routing | Controller routes based on intermediate results (e.g., re-query if insufficient) | 3h |
| 6.3.3 | Loop handling | Detect when agent output requires re-processing by a previous stage | 3h |

---

### Epic 7: Learning & Improvement Loop

**Description**: Close the feedback loop so the system improves with every interaction.  
**Priority**: P2 (Post-MVP)  
**Estimated Effort**: 6-8 days

#### Story 7.1: User Feedback Integration
*As a user, I want to rate report quality so that the system learns from my preferences.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 7.1.1 | Feedback UI/API | Simple 1-5 rating per report with optional free-text | 3h |
| 7.1.2 | Feedback storage in trajectories.db | Link rating to trajectory for Reinforcement learning | 2h |
| 7.1.3 | Quality trend reporting | Dashboard showing rating trends by agent, domain, query type | 3h |

#### Story 7.2: Pattern Reinforcement
*As a developer, I want the system to automatically strengthen patterns from successful interactions so that quality compounds.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 7.2.1 | Success pattern detection | Identify trajectories with high ratings and low correction need | 3h |
| 7.2.2 | Confidence boosting | Increase confidence scores on patterns from successful trajectories | 2h |
| 7.2.3 | Pattern pruning | Reduce confidence on outdated or contradicted patterns | 2h |
| 7.2.4 | Periodic model optimization | Run MemoryOptimizer to consolidate and prune | 2h |

#### Story 7.3: Report Writer LoRA Fine-Tuning (Optional)
*As a user, I want reports to have consistent style and formatting so that they match my organization's standards.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 7.3.1 | Gold-standard report collection | Collect 1000+ examples of desired report format | Ongoing |
| 7.3.2 | LoRA training pipeline | Fine-tune Report Writer agent only on style/output format | 5h |
| 7.3.3 | A/B testing | Compare fine-tuned vs prompted report quality | 3h |

---

### Epic 8: Production Readiness

**Description**: Hardening, scaling, and deployment for production use.  
**Priority**: P2 (Post-MVP)  
**Estimated Effort**: 8-10 days

#### Story 8.1: Scalability
*As a developer, I want the system to handle growing knowledge volume without performance degradation.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 8.1.1 | AgentDB sharding | Shard documents.db by domain for horizontal scaling | 4h |
| 8.1.2 | QUIC synchronization | Enable QUIC sync across distributed AgentDB instances | 3h |
| 8.1.3 | Caching strategy | Cache frequent subgraphs and query results | 3h |
| 8.1.4 | Batch processing | Queue-based ingestion at scale | 3h |

#### Story 8.2: Security & Compliance
*As an administrator, I want the system to handle sensitive domain knowledge securely.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 8.2.1 | Data encryption | Encrypt AgentDB databases at rest | 2h |
| 8.2.2 | Access control | Role-based access to ingestion/query/admin endpoints | 3h |
| 8.2.3 | Audit logging | Log all queries and generated reports for compliance | 2h |
| 8.2.4 | Source redaction | Ability to mark sources as confidential and restrict output visibility | 3h |

#### Story 8.3: Monitoring & Observability
*As a developer, I want visibility into system performance and agent behavior.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 8.3.1 | Performance metrics | Track vector search latency, agent response time, token usage | 3h |
| 8.3.2 | Agent behavior logging | Log each agent's actions, decisions, and intermediate outputs | 3h |
| 8.3.3 | Alerting | Alert on failures, slow queries, low faithfulness scores | 2h |
| 8.3.4 | Dashboard | Real-time dashboard for system health and usage | 4h |

---

## 4. Release Plan

### Release 1 (MVP) — Weeks 1-2

**Goal**: Prove the core retrieve-and-answer loop with text-only knowledge.

**Epics included**: Epic 1 (Knowledge Foundation), Epic 2 Story 2.1 (Text Ingestion), Epic 3 Story 3.1 (Single Analyst Agent)

**Deliverables:**
- AgentDB initialized with ontology.db, documents.db, trajectories.db
- 20 seed entities + 40 relationships in ontology
- PDF ingestion pipeline (5-10 papers)
- Single Analyst Agent producing grounded answers
- 20 test queries with citation accuracy report

**Acceptance criteria:**
- Agent answers questions using only ingested knowledge
- Every answer claim is traceable to a source node
- ≥80% citation accuracy on test queries

### Release 2 (Multimodal + Swarm) — Weeks 3-4

**Goal**: Add multimodal ingestion and full agent swarm.

**New epics**: Epic 2 Stories 2.2-2.5, Epic 3 Stories 3.2-3.6, Epic 4 (Expert Reasoning)

**Deliverables:**
- Diagram ingestion (vision → symbolic graph)
- Formula ingestion (math OCR → LaTeX)
- Full agent swarm: Controller, Diagram Analyst, Research Analyst, Reasoning Agent, Report Writer
- Domain-specific CoT prompts and dynamic few-shot retrieval
- ReasoningBank trajectory tracking

**Acceptance criteria:**
- Diagram → symbolic graph accuracy ≥90%
- Full pipeline executes end-to-end (diagram in → report out)
- Trajectories recorded for all interactions

### Release 3 (Evaluation + Polish) — Week 5+

**Goal**: Measure quality, close the learning loop, productionize.

**New epics**: Epic 5 (Evaluation), Epic 6 (Orchestration), Epic 7 (Learning), Epic 8 (Production)

**Deliverables:**
- Automated faithfulness checker
- Report quality rubric and expert review process
- FastAPI backend with REST endpoints
- User feedback integration
- Monitoring and alerting
- Optional: Report Writer LoRA fine-tuning

**Acceptance criteria:**
- 100% faithfulness on automated checks
- Report quality ≥4/5 on expert rubric
- API endpoints functional with auth
- System recovers gracefully from agent failures

---

## 5. Architecture Decisions

| Decision | Rationale |
|---|---|
| **AgentDB over Graphiti** | AgentDB provides vector search, hybrid search, HNSW, quantization, and ReasoningBank in one stack — no separate Neo4j + Pinecone needed. 150x faster than traditional solutions. |
| **No fine-tuning first** | Expertise comes from prompt architecture + graph constraints + tool-augmented reasoning. Fine-tuning only for Report Writer stylistic consistency later. |
| **Controller meta-agent** | Linear pipelines fail on ambiguous requests. Controller plans, branches, loops, and aggregates. |
| **Conflict detection at ingest** | Surface contradictions early; turn the system into an active knowledge validator. |
| **MMR for diversity** | Maximal Marginal Relevance ensures the Reasoning Agent sees multiple perspectives, not just the most similar result. |
| **Hierarchical swarm topology** | Controller as coordinator + specialized workers matches the "committee of specialists" pattern. |
| **LangGraph for cyclic graphs** | Enables iterative reasoning where agents can re-query if initial results are insufficient. |

---

## 6. Open Questions

1. **Domain**: What specific domain is this being built for? Ontology, diagram parsers, and report templates depend heavily on this.
2. **Volume**: How many papers/diagrams/audio files? (100s vs 100,000s changes the sharding and cost strategy)
3. **Latency**: Are reports needed in real-time (<30s), interactive (<5min), or batch (asynchronous)?
4. **Deployment**: Local models on GPU (WSL) or cloud APIs? If local, what GPU constraints?
5. **Foundation model**: Which model(s) for reasoning and vision? (GPT-4o, Claude Sonnet, local Llama/Mistral)
6. **Domain experts available**: Who will define the ontology and evaluate report quality?
