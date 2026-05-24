# Agent Frosty — Product Requirements Document v2

**Status**: Draft  
**Version**: 2.0  
**Previous**: [prd.md](prd.md)  
**Key Change**: Architecture revised from n8n-heavy to hybrid agentic stack (Hermes-Agent + Paperclip + AgentDB + Pi + OpenCode). n8n demoted to MCP-connected pipeline tool.  
**Discussion**: See [discussions.md](discussions.md) for full rationale.  
**Last Updated**: 2026-05-24

---

## 1. Product Overview

### 1.1 Vision

Agent Frosty is an **expert intelligent agent system** that ingests domain-specific multimodal knowledge (research papers, architectural diagrams, formulas, reports, audio, video) and produces grounded, citation-backed technical reports. It does not fine-tune a model to "know everything" — it constructs expertise through structured knowledge, graph-augmented retrieval, and a specialist committee of agents running on **Hermes-Agent**, governed by **Paperclip**, with knowledge stored in **AgentDB**.

### 1.2 Core Principles

- **Constructed intelligence over fine-tuning** — expertise emerges from prompt architecture + graph constraints + tool-augmented reasoning
- **Provenance-first** — every claim must be traceable to a source node
- **Multimodal → symbolic** — diagrams, formulas, and audio are converted to structured representations, not stored as raw media
- **Agent specialization** — a committee of Hermes skill-based specialists beats a monolithic agent
- **Self-improving** — Hermes-Agent's built-in learning loop creates skills from experience, improves them during use, and persists knowledge across sessions
- **Governed autonomy** — Paperclip provides org charts, budgets, mission alignment, and full audit trails

### 1.3 Target Users

- Domain experts who need to produce technical reports from research materials
- Engineers reviewing designs against standards/compliance
- Researchers synthesizing findings across papers, diagrams, and datasets
- Regulatory analysts producing compliance assessments
- Organizations running multiple agent teams with budget and governance needs

### 1.4 Success Metrics

| Metric | Target | Measurement |
|---|---|---|
| Faithfulness | 100% | Automated citation check — every claim traceable to source |
| Diagram accuracy | ≥95% | Human spot-check 10% of symbolic graphs vs source |
| Contradiction flagging | 100% | % of queries where conflicting sources are surfaced |
| Report quality | ≥4/5 | Expert human review against rubric |
| Query-to-report latency | <60s (MVP), <10s (full) | End-to-end timer |
| Knowledge scale | 100+ sources (MVP), 10K+ (full) | Source count in AgentDB |
| Agent learning | Automatic | Hermes skill creation from trajectories |
| Delivery platforms | 3+ (MVP), 6+ (full) | Hermes gateway platform count |

---

## 2. Scope Definition

### 2.1 MVP Scope (Weeks 1-2)

The MVP proves the core loop: **ingest → store → retrieve → generate answer** using Hermes-Agent as the runtime.

**In scope:**
- Hermes-Agent installation and configuration (model provider, workspace, context file)
- AgentDB multi-database initialization (ontology + documents) with MCP server
- Domain ontology definition (entities, relationships, schema)
- Ingestion pipeline for research papers only (PDF → chunk → embed → store) as a Hermes tool
- Single "Analyst" Hermes skill: prompt + AgentDB MCP retrieval + text answer
- 20 test queries with citation accuracy measurement
- Manual ontology population (no automated extraction yet)
- Basic deliverable: CLI output (multi-platform delivery deferred)

**Out of scope (MVP):**
- Hermes skill self-improvement loop (automatic — don't build, just configure)
- Diagram/multimodal ingestion
- Paperclip governance layer
- n8n integration
- Evaluation pipeline
- Fine-tuning
- Audio/video processing
- Multi-platform delivery (Telegram, Slack, etc.)
- Production deployment (Docker, Modal, etc.)
- Quotation & Proposal Management

### 2.2 Full Project Scope

| Component | MVP | Full |
|---|---|---|
| Runtime | Hermes-Agent (basic) | Hermes-Agent + Paperclip governance |
| Knowledge base | ontology.db + documents.db | + diagrams.db + formulas.db |
| Ingestion | Research papers (text only) | Diagrams, formulas, audio, video, images |
| Agents | Single Analyst Hermes skill | Diagram Analyst, Research Analyst, Reasoning Agent, Report Writer — all as Hermes skills |
| Retrieval | Vector search via AgentDB MCP | Hybrid search + graph traversal + MMR |
| Learning | Hermes built-in (automatic) | Hermes trajectory tracking + skill creation + Honcho user modeling |
| Evaluation | Manual citation check | Automated faithfulness, diagram accuracy, contradiction detection |
| Multimodal | None | Vision → symbolic graph, math OCR, audio transcription |
| Delivery | CLI only | Telegram, Discord, Slack, WhatsApp, Signal, Email |
| Orchestration | Manual Hermes tool calls | Paperclip org chart + budgets + audit + cron scheduling |
| n8n | None | MCP-connected pipeline tool for visual workflow stages |
| Fine-tuning | None | Report Writer only (LoRA, optional) |
| Quotation & Proposal | None | Multi-channel inbound, knowledge-backed generation, approval workflow, email delivery, user notifications, analytics, Paperclip governance, CRM sync |

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
| 1.2.4 | Configure AgentDB MCP server | Start AgentDB MCP server for Hermes tool integration | 1h |
| 1.2.5 | Write AgentDB adapter module | Singleton adapter factory that provides typed access to each database | 2h |

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
| 2.1.7 | Ingestion Hermes tool | Wrap ingestion as a Hermes tool/MCP resource callable from skills | 2h |
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
| 2.5.2 | ConflictNode schema | Define ConflictNode structure linking both claims with provenance | 2h |
| 2.5.3 | Human review flagging | Generate report of conflicts requiring manual resolution | 2h |
| 2.5.4 | Confidence scoring | Assign confidence scores to conflicting claims based on source recency, authority | 3h |

---

### Epic 3: Hermes-Agent Platform Setup

**Description**: Install, configure, and customize Hermes-Agent as the runtime for Agent Frosty's specialist agents.  
**Priority**: P0 (MVP)  
**Estimated Effort**: 2-3 days

#### Story 3.1: Hermes-Agent Installation & Configuration
*As a developer, I want Hermes-Agent installed and configured so that it serves as the runtime for Agent Frosty.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 3.1.1 | Install Hermes-Agent | Run install script for target platform (local, WSL2, or cloud VM) | 1h |
| 3.1.2 | Configure model provider | Set up Nous Portal, OpenRouter, or direct API keys for LLM access | 1h |
| 3.1.3 | Create Agent Frosty context file | Write `~/.hermes/context/agent-frosty.md` with domain CoT examples, expert prompt | 2h |
| 3.1.4 | Configure AgentDB MCP tool | Add AgentDB MCP server to Hermes' MCP configuration | 1h |
| 3.1.5 | Test end-to-end | Verify Hermes can call AgentDB via MCP and produce grounded answers | 2h |

#### Story 3.2: Agent Frosty Skill Framework
*As a developer, I want a skill framework so that specialist agents are modular, reusable Hermes skills.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 3.2.1 | Create skill directory structure | Set up `~/.hermes/skills/agent-frosty/` following agentskills.io format | 1h |
| 3.2.2 | Base skill template | Define base template with domain-expert system prompt, citation rules, uncertainty flagging | 2h |
| 3.2.3 | Skill registration | Register all Agent Frosty skills with Hermes so they are discoverable via `/skills` | 1h |
| 3.2.4 | Skill testing harness | Manual trigger to invoke each skill independently and verify output | 2h |

---

### Epic 4: Hermes Skill-Based Specialist Agents

**Description**: Build the committee of specialist agents as Hermes skills that interpret user requests, reason over knowledge, and produce reports.  
**Priority**: P0 (MVP: single skill), P1 (Full: all skills)  
**Estimated Effort**: 3-5 days (MVP), +10 days (all skills)

#### Story 4.1: Single Analyst Skill (MVP)
*As a user, I want to ask a question and get an answer grounded in my domain knowledge so that I trust the response is based on my sources.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 4.1.1 | Base expert system prompt | Write domain-expert system prompt with grounding rules | 2h |
| 4.1.2 | AgentDB MCP retrieval integration | Skill calls AgentDB via MCP for hybrid search across documents.db and ontology.db | 3h |
| 4.1.3 | Evidence bundle assembly | Retrieved chunks are assembled into a prompt context with citation metadata | 2h |
| 4.1.4 | Answer generation | LLM call with context + prompt → grounded answer | 2h |
| 4.1.5 | Citation formatting | Format citations with source, section, and line references | 2h |
| 4.1.6 | 20 test queries | Run queries, measure citation accuracy, identify gaps | 3h |

#### Story 4.2: Diagram Interpretation Skill (Full Scope)
*As a user, I want to upload a diagram and have it automatically analyzed so that its components are available for reasoning.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 4.2.1 | Vision model integration | Hermes skill calls multimodal LLM via tool or MCP for diagram analysis | 3h |
| 4.2.2 | Component extraction prompt | Structured prompt: extract components, connections, labels, load paths | 4h |
| 4.2.3 | Ontology linking | Map extracted components to ontology nodes via AgentDB hybrid search | 3h |
| 4.2.4 | Symbolic graph output | Produce structured JSON: `{components, relationships, load_paths, constraints}` | 2h |

#### Story 4.3: Research Analyst Skill (Full Scope)
*As a user, I want the system to evaluate research papers against existing knowledge so that new claims are validated or flagged.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 4.3.1 | Paper evaluation prompt | Hermes skill prompt for evaluating papers: extract claims, assess evidence quality | 3h |
| 4.3.2 | Claim extraction | Extract discrete factual claims from paper sections | 3h |
| 4.3.3 | Graph comparison | Compare extracted claims against ontology for alignment or contradiction | 3h |
| 4.3.4 | Confidence scoring per claim | Score each extracted claim (source quality, methodology, citation count) | 2h |

#### Story 4.4: Reasoning Skill (Full Scope)
*As a user, I want the system to apply formulas and constraints from my domain knowledge so that reports include quantitative analysis.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 4.4.1 | Formula retrieval | Query formulas.db for relevant formulas based on problem context | 2h |
| 4.4.2 | Variable substitution | Map variables from formula to actual values from diagram/graph | 3h |
| 4.4.3 | Constraint checking | Check computed values against constraint nodes in ontology | 3h |
| 4.4.4 | Multi-hop graph traversal | Traverse ontology edges for multi-step reasoning chains | 4h |

#### Story 4.5: Report Writer Skill (Full Scope)
*As a user, I want the system to produce polished, structured reports in my domain's expected format.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 4.5.1 | Report template system | Define templates for technical reports, compliance analyses, research summaries | 4h |
| 4.5.2 | Citation rendering | Format graph node citations with full provenance in report body | 2h |
| 4.5.3 | Uncertainty flagging | Automatically flag unsupported claims or contradictory evidence in output | 2h |
| 4.5.4 | Domain tone/style configuration | Configurable tone (regulatory, academic, engineering) | 2h |
| 4.5.5 | Structured output formats | Support markdown, PDF, and JSON output | 3h |

#### Story 4.6: Parent Agent Workflow (Full Scope)
*As a user, I want the parent Hermes agent to orchestrate the right skills based on my request so that complex analyses run end-to-end.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 4.6.1 | Workflow prompt engineering | Parent agent prompt for intent classification and skill routing | 3h |
| 4.6.2 | Subagent spawning pattern | Define patterns for spawning skills via `hermes spawn` or `/run` | 2h |
| 4.6.3 | Evidence bundle aggregation | Parent merges outputs from multiple skills into unified evidence bundle | 3h |
| 4.6.4 | Iterative refinement | Parent detects incomplete results and re-invokes skills with refined context | 3h |

---

### Epic 5: Expert Reasoning Without Fine-Tuning

**Description**: Configure the prompt architecture, context files, and Hermes-native learning that make the system behave like a domain expert without modifying the underlying model.  
**Priority**: P1 (Full Scope)  
**Estimated Effort**: 3-5 days

#### Story 5.1: Domain-Specific Prompt Architecture
*As a developer, I want every agent to operate under enforced domain-expert constraints so that outputs are grounded, citable, and precise.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 5.1.1 | Base expert prompt template | Define the grounding rules shared across all skills | 2h |
| 5.1.2 | Domain-specific CoT examples | Write 3-5 solved examples showing graph traversal, formula application, citation | 4h |
| 5.1.3 | Per-skill prompt specialization | Customize base prompt for each skill's role (Analyst, Reasoner, Writer) | 3h |
| 5.1.4 | Evidence bundle format | Standardized format for passing retrieval context into skill prompts | 2h |
| 5.1.5 | Context file publication | Write Agent Frosty context to `~/.hermes/context/agent-frosty.md` | 1h |

#### Story 5.2: Hermes Learning Loop Configuration
*As a developer, I want Hermes' built-in learning loop configured so that the system improves automatically without custom code.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 5.2.1 | Verify trajectory tracking | Confirm Hermes is recording trajectories to `~/.hermes/sessions/` | 1h |
| 5.2.2 | Skill creation from experience | Enable Hermes' autonomous skill creation after complex tasks | 1h |
| 5.2.3 | Cross-session memory | Configure Honcho or FTS5 for cross-session recall | 2h |
| 5.2.4 | Dynamic few-shot retrieval | Configure Hermes to search past sessions for relevant examples | 1h |

#### Story 5.3: Context File Optimization
*As a domain expert, I want the agent's context to contain domain-specific examples and rules so that it reasons correctly from the start.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 5.3.1 | Write domain primer | Concise domain overview (key concepts, terminology, conventions) | 2h |
| 5.3.2 | Write solved examples | 3-5 worked examples with correct graph traversal, citation, and uncertainty handling | 3h |
| 5.3.3 | Write anti-patterns | Common mistakes the agent should avoid (hallucination, speculation, missing citations) | 2h |
| 5.3.4 | Iterate based on test queries | Refine context file based on accuracy gaps from test queries | Ongoing |

---

### Epic 6: Evaluation Pipeline

**Description**: Build the automated and human-in-the-loop evaluation system that measures expertise quality.  
**Priority**: P1 (Full Scope)  
**Estimated Effort**: 5-7 days

#### Story 6.1: Automated Faithfulness Check
*As a developer, I want every claim in the output to be automatically verified against source nodes so that hallucination is detected.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 6.1.1 | Claim extraction from output | Parse report to extract discrete factual claims | 3h |
| 6.1.2 | Source node lookup | For each claim, query AgentDB for supporting source nodes | 3h |
| 6.1.3 | Citation validation | Verify that cited sources actually contain the claimed information | 3h |
| 6.1.4 | Faithfulness scoring | Compute percentage of claims with verifiable source | 2h |

#### Story 6.2: Diagram Accuracy Evaluation
*As a domain expert, I want the system's diagram interpretation to be validated so that symbolic graphs match source images.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 6.2.1 | Human review interface | Simple CLI for presenting diagram → symbolic graph pairs for review | 3h |
| 6.2.2 | Sampling strategy | Random 10% sample of all processed diagrams for review | 1h |
| 6.2.3 | Accuracy scoring | Calculate pass/fail rate per reviewer judgment | 2h |

#### Story 6.3: Report Quality Rubric
*As a domain expert, I want a formal rubric for evaluating report quality so that system improvements are measurable.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 6.3.1 | Define rubric dimensions | Completeness, tone, technical accuracy, citation quality, uncertainty handling | 2h |
| 6.3.2 | Scoring scale definition | 1-5 scale with anchored descriptors per dimension | 2h |
| 6.3.3 | Expert review collection | Process for domain experts to score 20-30 sample reports | 3h |
| 6.3.4 | Trend tracking | Dashboard or log showing quality scores over time | 2h |

#### Story 6.4: Contradiction Detection Evaluation
*As a developer, I want to measure how well the system surfaces conflicting information so that we can improve detection.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 6.4.1 | Test suite of known contradictions | Curate 20 queries where source materials contain known conflicts | 3h |
| 6.4.2 | Detection rate tracking | Run test suite, measure % of contradictions surfaced in output | 2h |
| 6.4.3 | False positive monitoring | Track cases where system flags non-contradictory information | 2h |

#### Story 6.5: Hermes Trajectory Analysis
*As a developer, I want to leverage Hermes' trajectory infrastructure for evaluation so that we don't build custom logging.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 6.5.1 | Trajectory export | Export Hermes sessions for analysis and training data | 1h |
| 6.5.2 | Session search | Use Hermes FTS5 search to find relevant past sessions | 1h |
| 6.5.3 | Success/failure pattern analysis | Analyze trajectories to identify recurring failure modes | 3h |

---

### Epic 7: Orchestration & Governance

**Description**: Build the runtime infrastructure for coordinating agents, governing work, and managing delivery. Replaces the prior LangGraph/Swarm approach with Hermes-native subagent spawning + Paperclip governance.  
**Priority**: P1 (Full Scope)  
**Estimated Effort**: 5-8 days

#### Story 7.1: Parent Agent Orchestration
*As a developer, I want the parent Hermes agent to coordinate specialist skills so that complex multi-step analyses execute correctly.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 7.1.1 | Intent classification prompt | Parent agent identifies request type (analysis, comparison, compliance check, etc.) | 3h |
| 7.1.2 | Skill routing logic | Parent determines which skills to invoke and in what order | 2h |
| 7.1.3 | Subagent lifecycle management | Hermes-native subagent spawning for parallel work | 2h |
| 7.1.4 | Error recovery | Parent detects failed sub-skills and retries or escalates | 3h |
| 7.1.5 | Evidence bundle aggregation | Parent merges outputs into standardized context for Report Writer | 2h |

#### Story 7.2: Paperclip Governance Layer (Full Scope)
*As an administrator, I want a governance layer with org charts and budgets so that I can manage multiple agent deployments.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 7.2.1 | Install Paperclip | Deploy Paperclip Node.js server + React UI | 2h |
| 7.2.2 | Define org chart | Create agent roles and reporting structure (CEO, CTO, specialists) | 2h |
| 7.2.3 | Set budgets | Configure monthly budgets per agent in Paperclip | 1h |
| 7.2.4 | Mission/goal alignment | Define top-level goals that trace down to agent tasks | 2h |
| 7.2.5 | Audit log review | Verify Paperclip captures all agent actions with tool-call tracing | 2h |

#### Story 7.3: Multi-Platform Delivery (Full Scope)
*As a user, I want reports delivered to my preferred platform so that I don't need to check a separate dashboard.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 7.3.1 | Telegram delivery | Configure Hermes Telegram bot, connect to Agent Frosty | 2h |
| 7.3.2 | Slack delivery | Configure Hermes Slack integration | 2h |
| 7.3.3 | Email delivery | Configure Hermes email gateway | 2h |
| 7.3.4 | Scheduled reports via cron | Configure Hermes cron for recurring report generation | 2h |

#### Story 7.4: FastAPI Backend (Optional, Full Scope)
*As a developer, I want a REST API so that the system can be called programmatically (alternative to Hermes CLI/gateway).*

| ID | Task | Description | Effort |
|---|---|---|---|
| 7.4.1 | API endpoint: ingest document | POST endpoint for uploading papers/diagrams/media | 3h |
| 7.4.2 | API endpoint: query | POST endpoint for submitting questions with optional diagram attachment | 3h |
| 7.4.3 | API endpoint: report status | GET endpoint to check generation progress | 2h |
| 7.4.4 | API endpoint: knowledge explorer | GET endpoint to browse ontology graph | 3h |
| 7.4.5 | Authentication | API key or JWT-based auth | 3h |

#### Story 7.5: n8n as MCP Pipeline Tool (Optional, Full Scope)
*As a developer, I want n8n connected via MCP for specific visual pipeline stages so that diagram processing and document transformation can be built visually.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 7.5.1 | n8n MCP server setup | Configure n8n MCP server for Hermes integration | 2h |
| 7.5.2 | Example pipeline: document processing | Build n8n workflow for document chunking + embedding as a callable MCP tool | 3h |
| 7.5.3 | Integration test | Verify Hermes can invoke n8n pipeline via MCP and receive results | 2h |

---

### Epic 8: Learning & Improvement Loop

**Description**: Close the feedback loop via Hermes' built-in learning mechanisms.  
**Priority**: P2 (Post-MVP)  
**Estimated Effort**: 3-5 days

#### Story 8.1: User Feedback Integration
*As a user, I want to rate report quality so that the system learns from my preferences.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 8.1.1 | Feedback collection | Use Hermes' existing feedback mechanisms (thumbs up/down, ratings) | 1h |
| 8.1.2 | Feedback → skill improvement | Configure Hermes to use feedback for skill self-improvement | 2h |
| 8.1.3 | Quality trend reporting | Export trajectory analysis for quality trends | 2h |

#### Story 8.2: Skill Refinement from Experience
*As a developer, I want the system to automatically strengthen successful patterns so that quality compounds.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 8.2.1 | Verify autonomous skill creation | Confirm Hermes creates skills from successful complex tasks | 1h |
| 8.2.2 | Skill pruning | Review and prune low-quality auto-created skills | 2h |
| 8.2.3 | Cross-session learning | Verify Honcho user model improves across sessions | 2h |

#### Story 8.3: Report Writer LoRA Fine-Tuning (Optional)
*As a user, I want reports to have consistent style and formatting so that they match my organization's standards.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 8.3.1 | Gold-standard report collection | Collect 1000+ examples of desired report format | Ongoing |
| 8.3.2 | LoRA training pipeline | Fine-tune Report Writer skill only on style/output format | 5h |
| 8.3.3 | A/B testing | Compare fine-tuned vs prompted report quality | 3h |

---

### Epic 9: Production Readiness

**Description**: Hardening, scaling, and deployment for production use.  
**Priority**: P2 (Post-MVP)  
**Estimated Effort**: 6-8 days

#### Story 9.1: Scalability
*As a developer, I want the system to handle growing knowledge volume without performance degradation.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 9.1.1 | AgentDB sharding | Shard documents.db by domain for horizontal scaling | 4h |
| 9.1.2 | Hermes backend configuration | Configure Hermes for production runtime (Modal, Daytona, Docker, or SSH) | 3h |
| 9.1.3 | Caching strategy | Cache frequent subgraphs and query results | 3h |
| 9.1.4 | Batch processing | Queue-based ingestion at scale | 3h |

#### Story 9.2: Security & Compliance
*As an administrator, I want the system to handle sensitive domain knowledge securely.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 9.2.1 | Data encryption | Encrypt AgentDB databases at rest | 2h |
| 9.2.2 | Access control | Paperclip role-based access to ingestion/query/admin | 3h |
| 9.2.3 | Audit logging | Paperclip audit log for all agent actions | 2h |
| 9.2.4 | Source redaction | Ability to mark sources as confidential and restrict output visibility | 3h |

#### Story 9.3: Monitoring & Observability
*As a developer, I want visibility into system performance and agent behavior.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 9.3.1 | Performance metrics | Track vector search latency, agent response time, token usage | 3h |
| 9.3.2 | Hermes session monitoring | Monitor Hermes sessions for failures, slow responses | 2h |
| 9.3.3 | Alerting | Alert on failures, slow queries, low faithfulness scores | 2h |
| 9.3.4 | Paperclip dashboard | Real-time dashboard for system health and usage | 3h |

---

### Epic 10: Quotation & Proposal Management

**Description**: End-to-end quotation and proposal system — customers send requests via email, Slack, Discord, or Telegram; the system parses requirements, retrieves pricing from AgentDB, generates quotes/proposals via Hermes, routes through approval, delivers to customer by email, and notifies the user via their configured channels. Orchestrated via n8n workflows with Paperclip governance.  
**Priority**: P1 (Full Scope)  
**Estimated Effort**: 5-8 days

#### Story 10.1: Multi-Channel Inbound Request Intake
*As a customer, I want to send a quote request via email, Slack, Discord, or Telegram so that the system automatically receives and processes it.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 10.1.1 | Email inbound listener | Configure IMAP/webhook monitoring for quote request emails; parse sender, subject, body, attachments | 3h |
| 10.1.2 | Slack inbound listener | Configure Slack app event subscription for DMs and channel messages with quote intent | 3h |
| 10.1.3 | Discord inbound listener | Configure Discord bot for quote requests in designated channel or DMs | 3h |
| 10.1.4 | Telegram inbound listener | Configure Telegram bot for quote requests | 3h |
| 10.1.5 | Unified request queue | Store all inbound requests in a normalized format with channel attribution; deduplicate cross-channel | 2h |

#### Story 10.2: Request Parsing & Quote Generation
*As a user, I want the system to automatically extract customer details and requirements, then generate a quote backed by my knowledge base.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 10.2.1 | NLP request parsing | Extract customer name, company, contact info, requirements, quantities, deadlines using LLM | 4h |
| 10.2.2 | Attachment processing | Classify and store attachments (specs, RFQs, reference images); link to request | 2h |
| 10.2.3 | Pricing knowledge retrieval | Hermes agent queries AgentDB for pricing data, standards, prior quote references | 3h |
| 10.2.4 | Quote generation prompt | Hermes skill prompt for generating structured quotes with line items, pricing, terms | 4h |
| 10.2.5 | Quote template system | Configurable templates for standard quote, detailed proposal, RFP response with branding | 3h |

#### Story 10.3: Approval, Delivery & Notifications
*As a user, I want to approve quotes before sending, have them delivered to the customer by email, and receive status updates on my preferred channels.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 10.3.1 | Approval workflow engine | Configurable routing: auto-approve under threshold, route to specific approvers, escalation timer | 4h |
| 10.3.2 | Customer email delivery | Send branded HTML email with PDF attachment; track delivery, opens, bounces | 3h |
| 10.3.3 | User notification dispatch | Notify user of status changes (received, drafted, approved, sent, customer replied) via configured channels | 3h |
| 10.3.4 | Quote status lifecycle | State machine: received → drafting → pending_approval → sent → viewed → accepted/rejected → revised | 2h |

#### Story 10.4: n8n Workflow Orchestration
*As a developer, I want the full quote lifecycle implemented as n8n workflows so that it can be visually managed and customized.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 10.4.1 | Quotation workflow template | n8n workflow: inbound listener → parse → generate → approval → deliver → notify | 4h |
| 10.4.2 | n8n-Hermes MCP integration | Connect n8n to Hermes via MCP for agent-driven pricing retrieval and quote generation | 2h |
| 10.4.3 | n8n-Paperclip integration | Connect n8n to Paperclip for budget checks and audit logging | 2h |

#### Story 10.5: Quote Analytics & CRM Integration (Full Scope)
*As a manager, I want visibility into quote performance and optional sync with my CRM.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 10.5.1 | Quote analytics dashboard | Acceptance rates, response times, conversion by channel, revenue pipeline | 3h |
| 10.5.2 | Quote versioning | Revision tracking with audit trail and diff view | 2h |
| 10.5.3 | Customer communication history | Full transcript of all customer interactions stored in graph | 3h |
| 10.5.4 | CRM sync (HubSpot/Salesforce) | Bidirectional sync of customers, quotes, and status with external CRM | 4h |

#### Story 10.6: Paperclip Governance for Quotes (Full Scope)
*As a finance manager, I want budget limits, approval chains, and audit trails on all quote actions.*

| ID | Task | Description | Effort |
|---|---|---|---|
| 10.6.1 | Budget limits per quote | Enforce per-agent/team budget caps; flag over-budget for special approval | 2h |
| 10.6.2 | Approval chain mapping | Map quote approval hierarchy to Paperclip org chart | 2h |
| 10.6.3 | Quote audit trail | All quote actions logged to Paperclip audit with full provenance | 2h |
| 10.6.4 | Cost tracking per agent | Track total quote value generated by each agent/team | 1h |

---

## 4. Release Plan

### Release 1 (MVP) — Weeks 1-2

**Goal**: Prove the core retrieve-and-answer loop with text-only knowledge on Hermes-Agent.

**Epics included**: Epic 1 (Knowledge Foundation), Epic 2 Story 2.1 (Text Ingestion), Epic 3 (Hermes Platform), Epic 4 Story 4.1 (Single Analyst Skill)

**Deliverables:**
- Hermes-Agent installed and configured with model provider
- AgentDB initialized with ontology.db, documents.db — MCP server running
- Agent Frosty context file with domain CoT examples
- 20 seed entities + 40 relationships in ontology
- PDF ingestion pipeline as a Hermes tool
- Single Analyst Hermes skill producing grounded answers
- 20 test queries with citation accuracy report

**Acceptance criteria:**
- Hermes agent answers questions using only ingested knowledge via AgentDB MCP
- Every answer claim is traceable to a source node
- ≥80% citation accuracy on test queries
- Agent Frosty context file loaded and effective

### Release 2 (Multimodal + Specialist Skills + Quote MVP) — Weeks 3-4

**Goal**: Add multimodal ingestion, full specialist skill set, and basic quotation flow on Hermes.

**New epics**: Epic 2 Stories 2.2-2.5, Epic 4 Stories 4.2-4.6, Epic 5 (Expert Reasoning), Epic 10 Stories 10.1-10.4 (Quote MVP)

**Deliverables:**
- Diagram ingestion (vision → symbolic graph) as Hermes skill
- Formula ingestion (math OCR → LaTeX)
- All Hermes skills: Diagram Analyst, Research Analyst, Reasoning Agent, Report Writer
- Parent agent workflow: intent → spawn skills → aggregate → report
- Multi-platform delivery (Telegram or Slack)
- Hermes built-in learning loop active
- Multi-channel quote request intake (email + 1 chat platform)
- Knowledge-backed quote generation with templates
- Approval workflow (auto-approve under threshold, manual approval over)
- Customer email delivery with PDF attachment
- User notifications via configured channel
- n8n workflow template for basic quote lifecycle

**Acceptance criteria:**
- Diagram → symbolic graph accuracy ≥90%
- Full pipeline executes end-to-end from Hermes parent agent
- Skills spawn as subagents for parallel work
- Reports delivered to at least one non-CLI platform
- Hermes automatically records trajectories and creates skills from experience
- Quote requests accepted via email and Slack, quotes generated and sent to customer
- Approval workflow routes correctly based on configurable threshold

### Release 3 (Governance + Evaluation + Production + Quote Full) — Week 5+

**Goal**: Governance, quality measurement, production deployment, and full quotation features.

**New epics**: Epic 6 (Evaluation), Epic 7 (Orchestration & Governance), Epic 8 (Learning), Epic 9 (Production), Epic 10 Stories 10.5-10.6 (Quote Full)

**Deliverables:**
- Paperclip governance layer (org chart, budgets, audit)
- Automated faithfulness checker
- Report quality rubric and expert review process
- Scheduled reports via Hermes cron
- Production deployment on Hermes-supported infra (Docker, Modal, or SSH)
- Paperclip governance for quotes (budget limits, approval chains, audit trails)
- Quote analytics dashboard (acceptance rates, conversion by channel, revenue pipeline)
- Customer communication history stored in knowledge graph
- Optional: CRM sync (HubSpot/Salesforce), n8n MCP integration, FastAPI backend, LoRA fine-tuning

**Acceptance criteria:**
- 100% faithfulness on automated checks
- Report quality ≥4/5 on expert rubric
- Paperclip dashboard shows agent activity, budgets, audit log, and quote metrics
- System recovers gracefully from agent failures
- Cron-delivered reports reach the right platforms
- Quote budget limits enforced; over-budget quotes routed for special approval

---

## 5. Architecture Decisions

| Decision | Rationale |
|---|---|
| **Hermes-Agent over custom runtime** | Hermes provides built-in learning loop, subagent spawning, trajectory tracking, FTS5 memory, cron, and multi-platform delivery — building all this from scratch would take months |
| **AgentDB over Graphiti** | AgentDB provides vector search, hybrid search, HNSW, quantization in one stack — no separate Neo4j + Pinecone needed |
| **Paperclip for governance** | Org charts, budgets, goal alignment, and audit logs are features Paperclip already provides — no custom development needed |
| **Hermes skills over custom agents** | skills.agentskills.io format provides modular, reusable, self-improving agent units |
| **n8n demoted to MCP tool** | No longer the orchestrator. Useful for specific pipeline stages where visual workflow construction adds value |
| **Pi as optional LLM abstraction** | Hermes already supports multi-provider LLM routing via Nous Portal and OpenRouter — Pi only needed if finer-grained control is required |
| **OpenCode as development tool** | Build + plan agents for SPARC methodology during development |
| **No fine-tuning first** | Expertise comes from prompt architecture + graph constraints + tool-augmented reasoning |
| **Conflict detection at ingest** | Surface contradictions early; turn the system into an active knowledge validator |
| **MMR for diversity** | Maximal Marginal Relevance ensures agents see multiple perspectives, not just the most similar |

---

## 6. Open Questions

1. **Domain**: What specific domain is this being built for? Ontology, diagram parsers, and report templates depend heavily on this.
2. **Volume**: How many papers/diagrams/audio files? (100s vs 100,000s changes the sharding and cost strategy)
3. **Latency**: Are reports needed in real-time (<30s), interactive (<5min), or batch (asynchronous)?
4. **Deployment**: Local (WSL2), cloud VM, or serverless (Modal/Daytona)? If local, what GPU constraints?
5. **Foundation model**: Which model(s) for reasoning and vision? (GPT-4o, Claude Sonnet, local Llama/Mistral)
6. **Domain experts available**: Who will define the ontology and evaluate report quality?
7. **Hermes vs Pi**: Is Pi's LLM abstraction needed alongside Hermes' built-in routing?
8. **Paperclip timing**: Add in Release 2 or defer to Release 3?
9. **n8n priority**: Is visual pipeline construction valuable enough to integrate, or can all stages be implemented as Hermes skills?
