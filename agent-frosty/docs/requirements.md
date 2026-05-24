# Agent Frosty — Requirements

> Comprehensive requirements for all 11 feature categories. Each feature includes description, user stories, acceptance criteria, priority, and dependencies.

---

## 1. Knowledge & Storage

### Overview
Foundation layer providing structured domain ontology, vector databases, and retrieval infrastructure. All agent reasoning is grounded in knowledge stored here.

### Features

---

#### 1.1 Domain Ontology

**Description**: Formal entity types (Component, Formula, Concept, Constraint, Method, FailureMode, Metric, Diagram, Paper, Standard) and relationships (depends_on, implements, constrains, derived_from, validated_by, contradicts, illustrated_by, measured_by) forming the knowledge graph schema.

**Priority**: MVP

**Dependencies**: None

**User Stories**:
- As a domain expert, I want to define entity types and relationships so knowledge is consistently structured
- As a system architect, I want a formal ontology so agents can reason over typed relationships

**Acceptance Criteria**:
- Ontology supports all 10 entity types defined in the schema
- All relationships listed are bidirectional queryable
- AgentDB schema is initialized with the ontology on setup

---

#### 1.2 Ontology Schema with Provenance

**Description**: Each node includes version, validity_range, confidence_score, source_refs for temporal/versioned knowledge tracking.

**Priority**: MVP

**Dependencies**: 1.1

**User Stories**:
- As a researcher, I want to know when a claim was made and by whom so I can assess recency
- As a reviewer, I want to see confidence scores so I can weigh evidence quality

**Acceptance Criteria**:
- Every node has version, validity_range, confidence_score, source_refs fields
- Provenance fields are rendered in report citations
- Queries can filter by validity_range

---

#### 1.3 AgentDB Multi-Database

**Description**: Separate databases for ontology, documents, diagrams, formulas — each with tuned quantization and HNSW indexing.

**Priority**: MVP

**Dependencies**: 1.1

**User Stories**:
- As a developer, I want separate databases so each data type is optimized independently
- As a system administrator, I want independent scaling of storage per data type

**Acceptance Criteria**:
- Four databases created on setup: ontology.db, documents.db, diagrams.db, formulas.db
- Each database has its own quantization and indexing configuration
- Databases are independently queryable via MCP

---

#### 1.4 Binary Quantization (32x)

**Description**: 32x memory compression for large document volumes in documents.db.

**Priority**: MVP

**Dependencies**: 1.3

**User Stories**:
- As a system operator, I want compression so large document collections fit in available memory

**Acceptance Criteria**:
- documents.db uses binary quantization
- Recall remains within 95% of full-precision search on benchmark queries
- Documents.db memory usage is reduced vs full precision

---

#### 1.5 Scalar Quantization (4x)

**Description**: 4x compression for high-precision ontology and formula queries.

**Priority**: MVP

**Dependencies**: 1.3

**User Stories**:
- As a reasoning agent, I need high-precision vector matching for formulas and ontology lookups

**Acceptance Criteria**:
- ontology.db and formulas.db use scalar quantization
- Precision loss <1% compared to full-precision
- Sub-100µs query latency maintained

---

#### 1.6 HNSW Indexing

**Description**: Hierarchical Navigable Small World indexing for sub-100µs vector search across all databases.

**Priority**: MVP

**Dependencies**: 1.3

**User Stories**:
- As an agent, I want fast retrieval so responses are generated in real-time

**Acceptance Criteria**:
- All four databases use HNSW indexing
- P99 query latency <100µs
- Index builds complete within 5 minutes per 100K vectors

---

#### 1.7 AgentDB MCP Server

**Description**: Exposes AgentDB databases as MCP tools for Hermes-Agent integration.

**Priority**: MVP

**Dependencies**: 1.3

**User Stories**:
- As an agent developer, I want to query AgentDB from Hermes using standard MCP tool calls
- As a skill author, I want to retrieve knowledge without writing database code

**Acceptance Criteria**:
- MCP server starts with `npx agentdb@latest mcp`
- Hermes can call tools: `search_documents`, `search_diagrams`, `search_formulas`, `traverse_graph`
- Error responses are returned in standardized format

---

#### 1.8 Hybrid Retrieval Strategy

**Description**: Combines graph traversal (dependency tracing, contradiction detection) with vector search (semantic similarity).

**Priority**: Full

**Dependencies**: 1.7

**User Stories**:
- As a reasoning agent, I want to trace dependency chains through the graph while also finding semantically similar content
- As a researcher, I want results that include both structurally related and conceptually similar knowledge

**Acceptance Criteria**:
- Retrieval results contain both graph-traversal and vector-search results
- Results can be fused with configurable weighting (graph vs vector)
- Contradiction detection actively traverses contradicts edges

---

#### 1.9 MMR Result Diversification

**Description**: Maximal Marginal Relevance ensures diverse evidence, not just most similar.

**Priority**: Full

**Dependencies**: 1.8

**User Stories**:
- As an analyst, I want diverse evidence so reports cover multiple perspectives

**Acceptance Criteria**:
- MMR implemented with configurable lambda (relevance vs diversity)
- Top-K results show visibly different content vs pure similarity search
- Configurable per-query or global default

---

#### 1.10 Manual Ontology Population

**Description**: Seed entities and relationships defined by domain expert.

**Priority**: MVP

**Dependencies**: 1.1

**User Stories**:
- As a domain expert, I want to manually add entities and relationships to bootstrap the knowledge base

**Acceptance Criteria**:
- CLI or config-file method for adding ontology entities
- Validation of entity types and relationship types against schema
- Batch import from JSON/YAML supported

---

#### 1.11 AgentDB Sharding

**Description**: Horizontal scaling by domain for large knowledge volumes.

**Priority**: Full

**Dependencies**: 1.3

**User Stories**:
- As a system administrator, I want to shard databases across multiple nodes for horizontal scaling

**Acceptance Criteria**:
- Sharding supported by domain or data type
- Queries transparently routed across shards
- Consistent results across all shards

---

#### 1.12 Caching Strategy

**Description**: Cache frequent subgraphs and query results.

**Priority**: Full

**Dependencies**: 1.7

**User Stories**:
- As a system operator, I want repeated queries to return from cache for lower latency
- As a budget manager, I want cached results to reduce API costs

**Acceptance Criteria**:
- In-memory cache with configurable TTL per query type
- Cache hit rate >60% for common query patterns
- Cache invalidation on data updates

---

## 2. Ingestion Pipeline

### Overview
Pipeline for ingesting raw materials (papers, diagrams, formulas, audio, video, images) into structured knowledge in AgentDB.

### Features

---

#### 2.1 PDF Paper Ingestion

**Description**: PDF → section segmentation → semantic chunking → embedding → metadata extraction → store in AgentDB.

**Priority**: MVP

**Dependencies**: 1.3, 1.7

**User Stories**:
- As a researcher, I want to upload a PDF paper and have it ingested as searchable knowledge
- As an analyst, I want papers automatically chunked and embedded for similarity search

**Acceptance Criteria**:
- PDF text extracted with section boundary preservation
- Chunks sized 512-1024 tokens with 10% overlap
- Metadata extracted: title, authors, date, DOI, abstract
- Stored in documents.db with full provenance

---

#### 2.2 Batch Ingestion

**Description**: Process 5-10 documents in a single run.

**Priority**: MVP

**Dependencies**: 2.1

**User Stories**:
- As a researcher, I want to upload multiple papers at once for batch ingestion

**Acceptance Criteria**:
- Batch of 5-10 PDFs processed in a single command
- Individual progress reported per document
- Failures isolated (one bad PDF doesn't halt the batch)

---

#### 2.3 Diagram Ingestion (Vision LLM)

**Description**: Architectural diagrams → GPT-4V/Claude Sonnet → structured JSON component graph with connections, labels, load paths.

**Priority**: Full

**Dependencies**: 1.3, 1.7

**User Stories**:
- As an engineer, I want to upload architecture diagrams and have them parsed into structured component graphs

**Acceptance Criteria**:
- Vision LLM extracts: components, connections, labels, directional flow
- Output is structured JSON graph stored in diagrams.db
- Graph nodes linkable to ontology entities

---

#### 2.4 Diagram → Ontology Linking

**Description**: Extracted components linked to ontology entities via AgentDB hybrid search.

**Priority**: Full

**Dependencies**: 2.3, 1.1

**User Stories**:
- As a reasoning agent, I want diagram components automatically linked to their ontology definitions

**Acceptance Criteria**:
- Components matched to ontology entities with confidence score
- Links stored as edges in the graph
- Manual override option for incorrect matches

---

#### 2.5 Text-Diagram Cross-Reference

**Description**: Diagram elements linked to sentences/phrases in source papers.

**Priority**: Full

**Dependencies**: 2.3, 2.1

**User Stories**:
- As a researcher, I want to click a diagram element and see where it is discussed in the source paper

**Acceptance Criteria**:
- Cross-reference edges stored between diagram nodes and text chunks
- Bidirectional lookup (text→diagram and diagram→text)
- Display in report as (see Diagram X, ref [Y])

---

#### 2.6 Formula Ingestion (Math OCR)

**Description**: Nougat/Pix2Struct → LaTeX extraction → variable-to-ontology mapping → semantic meaning storage.

**Priority**: Full

**Dependencies**: 1.3

**User Stories**:
- As an engineer, I want formulas extracted from documents and mapped to their variables in the ontology

**Acceptance Criteria**:
- Math OCR correctly extracts LaTeX with >90% accuracy on benchmark
- Variables mapped to ontology entities (e.g., "P" → Power)
- Formula stored with semantic description and variable references

---

#### 2.7 Audio/Video Ingestion

**Description**: Whisper transcription → speaker diarization → claim extraction → storage.

**Priority**: Full

**Dependencies**: 1.3

**User Stories**:
- As a researcher, I want lecture recordings transcribed and ingested as knowledge
- As an analyst, I want claims extracted from audio/video content

**Acceptance Criteria**:
- Whisper transcription with speaker labels
- Claims extracted with timestamps
- Stored with source media reference in documents.db

---

#### 2.8 Report Ingestion

**Description**: NLP chunking + claim extraction from existing reports.

**Priority**: Full

**Dependencies**: 2.1

**User Stories**:
- As an organization, I want previously written reports ingested so new reports build on existing knowledge

**Acceptance Criteria**:
- Reports chunked and embedded same as paper ingestion
- Claim extraction identifies key statements with attribution
- Cross-references to source papers preserved

---

#### 2.9 Image Ingestion

**Description**: Vision captioning + object graph extraction from images (non-diagram images such as photographs).

**Priority**: Full

**Dependencies**: 11.1, 11.4

**User Stories**:
- As a researcher, I want photographs and figures ingested with auto-generated descriptions
- As a cataloguer, I want objects and scenes extracted from images as structured data

**Acceptance Criteria**:
- Vision LLM generates caption and extracts objects, scenes, relationships
- Output stored in image catalogue graph with metadata
- Objects linked to ontology where applicable

---

#### 2.10 Conflict Detection at Ingest

**Description**: Compare new claims against existing graph; create ConflictNode linking contradictory claims with provenance; flag for human review or surface in reports.

**Priority**: Full

**Dependencies**: 1.1, 1.8

**User Stories**:
- As a researcher, I want contradictory claims flagged automatically when new knowledge is ingested
- As a reviewer, I want conflict nodes visible so I can investigate and resolve

**Acceptance Criteria**:
- New claims compared against existing graph
- Contradictory claims linked via ConflictNode with provenance
- Conflicts surfaced in reports as uncertainty flags
- Configurable auto-resolve or human-review mode

---

#### 2.11 Confidence Scoring per Claim

**Description**: Score claims based on source recency, authority, methodology quality.

**Priority**: Full

**Dependencies**: 2.10

**User Stories**:
- As a decision-maker, I want confidence scores on claims so I can assess reliability

**Acceptance Criteria**:
- Scores calculated from source authority, recency, methodology factors
- Scores stored on each claim node
- Reports optionally filter by minimum confidence threshold

---

#### 2.12 Ingestion as Hermes Tool

**Description**: Ingestion pipeline wrapped as a Hermes tool/MCP resource callable from skills.

**Priority**: MVP

**Dependencies**: 2.1, 1.7

**User Stories**:
- As an agent, I want to trigger ingestion from within a Hermes session
- As a skill author, I want to call ingestion as part of automated workflows

**Acceptance Criteria**:
- Hermes tool `ingest` accepts file path or URL
- Progress streamed back during ingestion
- Result returns database entry ID

---

## 3. Agent Architecture

### Overview
Hermes-Agent runtime and specialist skill architecture for multi-agent collaboration.

### Features

---

#### 3.1 Hermes-Agent Runtime

**Description**: Self-improving AI agent with built-in learning loop, subagent spawning, MCP integration.

**Priority**: MVP

**Dependencies**: None

**User Stories**:
- As a user, I want to interact with the system through a conversational agent
- As a developer, I want the agent to self-improve based on outcomes

**Acceptance Criteria**:
- Hermes-Agent installed and running
- Can spawn subagents
- MCP tools discovered and callable
- Learning loop records sessions and outcomes

---

#### 3.2 Parent Controller Agent

**Description**: Interprets user intent, plans workflow, spawns subagents, aggregates results, handles iterative refinement.

**Priority**: Full

**Dependencies**: 3.1

**User Stories**:
- As a user, I want to describe a goal and have the system figure out the workflow
- As a power user, I want iterative refinement with feedback loops

**Acceptance Criteria**:
- Controller interprets free-form user requests
- Plans multi-step workflow using available skills
- Spawns subagents in parallel where possible
- Aggregates results into coherent response
- Supports refinement: user feedback → revised output

---

#### 3.3 Ingestion Agent (Hermes Skill)

**Description**: Hermes skill wrapping ingestion pipeline.

**Priority**: Full

**Dependencies**: 3.1, 2.12

**User Stories**:
- As an agent operator, I want a dedicated skill for ingesting new knowledge

**Acceptance Criteria**:
- Skill invocable via `hermes /run agent-frosty/ingestion-agent`
- Accepts file path, URL, or directory
- Reports ingestion results

---

#### 3.4 Diagram Interpretation Agent (Hermes Skill)

**Description**: Vision LLM → component graph → ontology linking → store diagram semantics.

**Priority**: Full

**Dependencies**: 3.1, 2.3, 2.4

**User Stories**:
- As an engineer, I want to upload diagrams and have them semantically interpreted by a specialist agent

**Acceptance Criteria**:
- Skill invocable with image attachment
- Returns structured component graph
- Components linked to ontology

---

#### 3.5 Research Analyst Agent (Hermes Skill)

**Description**: Paper evaluation, claim extraction, graph comparison, contradiction detection, confidence scoring.

**Priority**: Full

**Dependencies**: 3.1, 2.1

**User Stories**:
- As a researcher, I want a specialist agent that evaluates papers and extracts key claims
- As an analyst, I want cross-paper comparison with contradiction detection

**Acceptance Criteria**:
- Evaluates paper quality (methodology, novelty, limitations)
- Extracts claims as graph nodes
- Compares against existing knowledge graph
- Flags contradictions with confidence scoring

---

#### 3.6 Reasoning Agent (Hermes Skill)

**Description**: Graph traversal, formula retrieval/variable substitution, constraint checking, multi-hop reasoning.

**Priority**: Full

**Dependencies**: 3.1, 1.1, 1.8

**User Stories**:
- As an engineer, I want an agent that can trace dependencies and apply formulas across the knowledge graph

**Acceptance Criteria**:
- Traverses multi-hop relationships (e.g., "what depends on X?")
- Retrieves formulas and substitutes known variables
- Checks constraints defined in ontology
- Returns reasoning chain with evidence

---

#### 3.7 Report Writer Agent (Hermes Skill)

**Description**: Structured report generation with citations, uncertainty flagging, domain tone, multi-format output.

**Priority**: Full

**Dependencies**: 3.1, 5.x

**User Stories**:
- As a user, I want a specialist agent that produces polished, citation-backed reports from raw knowledge

**Acceptance Criteria**:
- Generates reports in configured format (markdown, PDF, JSON)
- Every claim includes citation to source node
- Uncertainty auto-flagged where evidence is weak or contradictory
- Tone configurable (regulatory, academic, engineering)

---

#### 3.8 Single Analyst Skill (MVP)

**Description**: Simplified single skill combining prompt + AgentDB MCP retrieval + grounded answer + citation formatting.

**Priority**: MVP

**Dependencies**: 3.1, 1.7

**User Stories**:
- As an MVP user, I want a single agent that can answer questions grounded in ingested knowledge

**Acceptance Criteria**:
- Single skill streamlines retrieval-answer workflow
- Answers include citations
- Works with text-only knowledge (MVP scope)
- 20 test queries answered with citation accuracy >90%

---

#### 3.9 Hermes Skill Framework

**Description**: agentskills.io format skills stored in `~/.hermes/skills/agent-frosty/`.

**Priority**: MVP

**Dependencies**: 3.1

**User Stories**:
- As a developer, I want skills to follow a standard format for maintainability

**Acceptance Criteria**:
- Skills follow agentskills.io specification
- Skills stored in standard location
- Hermes discovers and loads skills automatically

---

#### 3.10 Subagent Spawning

**Description**: Parent agent spawns specialist subagents via `hermes spawn` or `/run` for parallel work.

**Priority**: Full

**Dependencies**: 3.2

**User Stories**:
- As a user, I want complex queries handled by multiple specialist agents working in parallel

**Acceptance Criteria**:
- Parent agent can spawn 2+ subagents simultaneously
- Subagents return results independently
- Parent agent aggregates results

---

#### 3.11 Evidence Bundle Assembly

**Description**: Standardized format for passing retrieval context and intermediate results between skills.

**Priority**: Full

**Dependencies**: 3.2

**User Stories**:
- As a skill author, I want a standard contract for passing evidence between agents

**Acceptance Criteria**:
- Evidence bundle schema defined (sources, claims, confidence scores)
- Skills accept and return evidence bundles
- Bundle traceable through the full agent workflow

---

## 4. Expert Reasoning & Learning

### Overview
Prompt engineering strategies, chain-of-thought patterns, and Hermes built-in learning mechanisms that make agent reasoning expert-level.

### Features

---

#### 4.1 Domain-Expert System Prompt

**Description**: Every agent operates under: ground all claims, use formulas exactly, reference diagrams, flag uncertainty, no hallucination.

**Priority**: MVP

**Dependencies**: 3.1

**User Stories**:
- As a domain expert, I want agents to follow strict reasoning rules that prevent hallucination and ensure grounded answers

**Acceptance Criteria**:
- System prompt loaded for all agents on startup
- Prompt enforces: grounding, formula accuracy, diagram references, uncertainty flagging, no hallucination
- Prompt is versioned and auditable

---

#### 4.2 Domain-Specific CoT Examples

**Description**: 3-5 solved examples in context file showing graph traversal, formula application, citation, uncertainty handling.

**Priority**: MVP

**Dependencies**: 4.1

**User Stories**:
- As a new user, I want to see example reasoning chains so I understand how agents solve problems
- As a prompt engineer, I want CoT examples that guide model reasoning toward expert patterns

**Acceptance Criteria**:
- 3-5 solved examples in context file
- Examples cover: graph traversal, formula application, citation, uncertainty handling
- Examples follow the domain-expert system prompt rules

---

#### 4.3 Per-Skill Prompt Specialization

**Description**: Customized base prompt for each agent's specific role.

**Priority**: Full

**Dependencies**: 4.1

**User Stories**:
- As a skill author, I want each specialist agent to have a prompt tailored to its role

**Acceptance Criteria**:
- Each Hermes skill has its own base prompt
- Prompts inherit from domain-expert system prompt
- Role-specific instructions added for each skill

---

#### 4.4 Anti-Pattern Guide

**Description**: Common mistakes to avoid (hallucination, speculation, missing citations).

**Priority**: MVP

**Dependencies**: 4.1

**User Stories**:
- As a prompt engineer, I want explicit anti-pattern examples to reduce error rates

**Acceptance Criteria**:
- Anti-pattern guide included in context file
- Covers: hallucination, speculation, missing citations, over-confidence, ignoring contradictory evidence
- Referenced in system prompt

---

#### 4.5 Hermes Trajectory Tracking

**Description**: Built-in recording of all agent interactions to `~/.hermes/sessions/` with FTS5 search.

**Priority**: MVP (auto)

**Dependencies**: 3.1

**User Stories**:
- As a developer, I want full session logs for debugging and analysis

**Acceptance Criteria**:
- All interactions recorded automatically
- Sessions searchable via FTS5
- Sessions exportable

---

#### 4.6 Hermes Verdict Judgment

**Description**: Agent autonomously judges task success/failure.

**Priority**: MVP (auto)

**Dependencies**: 3.1

**User Stories**:
- As a system operator, I want agents to self-assess task outcomes for quality monitoring

**Acceptance Criteria**:
- Each session records verdict (success/failure/partial)
- Verdict includes reasoning
- Verdict accessible in session logs

---

#### 4.7 Hermes Skill Creation from Experience

**Description**: After complex tasks, Hermes auto-creates skills capturing the successful pattern.

**Priority**: MVP (auto)

**Dependencies**: 3.1

**User Stories**:
- As a user, I want the system to capture successful workflows as reusable skills

**Acceptance Criteria**:
- After complex task completion, Hermes offers to create a skill
- Created skill captures the workflow pattern
- Skill available for future use

---

#### 4.8 Hermes Skill Self-Improvement

**Description**: Skills improve during use based on outcomes.

**Priority**: MVP (auto)

**Dependencies**: 3.1

**User Stories**:
- As a user, I want skills to get better over time as they're used

**Acceptance Criteria**:
- Skills automatically refined based on verdict patterns
- Improvements reflected in future invocations
- Change history tracked for each skill

---

#### 4.9 Cross-Session Memory (Honcho)

**Description**: Dialectic user modeling persists preferences and knowledge across sessions.

**Priority**: Full

**Dependencies**: 3.1

**User Stories**:
- As a returning user, I want the system to remember my preferences and past work

**Acceptance Criteria**:
- User preferences persist across sessions
- Past session knowledge accessible in current session
- User model updates based on interactions

---

#### 4.10 Dynamic Few-Shot Retrieval

**Description**: Hermes searches past sessions for relevant examples to inject into current context.

**Priority**: Full

**Dependencies**: 4.5

**User Stories**:
- As an agent, I want relevant past examples auto-injected to improve current reasoning

**Acceptance Criteria**:
- Past sessions searched for similar tasks
- Relevant examples injected as few-shot prompts
- Performance improvement measured vs static examples

---

#### 4.11 Context File System

**Description**: `~/.hermes/context/agent-frosty.md` contains domain primer, solved examples, anti-patterns.

**Priority**: MVP

**Dependencies**: 4.1, 4.2, 4.4

**User Stories**:
- As a developer, I want a single documented context file that configures agent behavior

**Acceptance Criteria**:
- Context file contains: domain primer, solved examples, anti-patterns, system prompt
- File is version controlled
- Hermes loads context file on startup

---

#### 4.12 Report Writer LoRA Fine-Tuning

**Description**: Optional fine-tuning of Report Writer only for stylistic consistency (requires 1000+ examples).

**Priority**: Full

**Dependencies**: 3.7

**User Stories**:
- As a quality manager, I want the Report Writer fine-tuned for consistent organizational tone

**Acceptance Criteria**:
- LoRA adapter trainable from 1000+ report examples
- Fine-tuned model produces consistent style
- Base model remains unchanged (LoRA overlay only)
- Fine-tuning pipeline documented

---

## 5. Report Generation & Delivery

### Overview
Multi-format report generation and multi-platform delivery system.

### Features

---

#### 5.1 Technical Reports

**Description**: Formal technical document with citations, formulas, diagrams.

**Priority**: Full

**Dependencies**: 3.7

**User Stories**:
- As an engineer, I want formal technical reports that include formulas and diagrams with proper citations

**Acceptance Criteria**:
- Report includes: title, abstract, sections, citations, formulas, diagrams
- All claims cite source nodes
- Exportable as PDF and markdown

---

#### 5.2 Compliance Analyses

**Description**: Regulatory/compliance focused reports with standard references.

**Priority**: Full

**Dependencies**: 3.7

**User Stories**:
- As a compliance officer, I want reports that map findings to specific regulatory standards

**Acceptance Criteria**:
- Report references regulatory standards
- Findings mapped to specific standard clauses
- Compliance status per clause (pass/fail/unknown)

---

#### 5.3 Research Summaries

**Description**: Condensed findings from multiple sources.

**Priority**: Full

**Dependencies**: 3.7

**User Stories**:
- As a researcher, I want concise summaries of multiple papers on a topic

**Acceptance Criteria**:
- Summary covers 3+ sources
- Key claims extracted and compared
- Agreements and contradictions highlighted

---

#### 5.4 Design Reviews

**Description**: Analysis of designs against standards and constraints.

**Priority**: Full

**Dependencies**: 3.7

**User Stories**:
- As a design reviewer, I want automated analysis of a design against known constraints and standards

**Acceptance Criteria**:
- Design compared against relevant constraints from ontology
- Violations flagged with references
- Recommendations provided

---

#### 5.5 Contradiction Reports

**Description**: Reports specifically surfacing conflicting sources.

**Priority**: Full

**Dependencies**: 3.7, 2.10

**User Stories**:
- As a researcher, I want a dedicated report type that focuses on contradictory evidence

**Acceptance Criteria**:
- Report lists all known contradictions on a topic
- Each contradiction includes both conflicting claims with sources
- Confidence assessment per claim

---

#### 5.6 Citation Rendering

**Description**: Every claim cites specific graph nodes with full provenance.

**Priority**: MVP

**Dependencies**: 3.8

**User Stories**:
- As a reader, I want every claim backed by a specific, traceable source

**Acceptance Criteria**:
- Every factual claim in a report includes citation
- Citation links to graph node with full provenance
- Citations rendered as [Author, Year, Section] or similar format

---

#### 5.7 Uncertainty Flagging

**Description**: Auto-flag unsupported claims or contradictory evidence in output.

**Priority**: MVP

**Dependencies**: 3.8, 2.10

**User Stories**:
- As a decision-maker, I want uncertainty clearly flagged so I know where evidence is weak

**Acceptance Criteria**:
- Claims without source backing flagged as [Unsupported]
- Contradictory evidence flagged as [Contradiction: see ref X vs ref Y]
- Low-confidence claims flagged with confidence score

---

#### 5.8 LaTeX Formula Rendering

**Description**: Formulas rendered properly in reports.

**Priority**: Full

**Dependencies**: 5.1, 2.6

**User Stories**:
- As an engineer, I want formulas rendered correctly in technical reports

**Acceptance Criteria**:
- LaTeX formulas render correctly in PDF output
- Formulas readable in markdown (LaTeX delimiters)
- Variables hyperlinked to ontology definitions

---

#### 5.9 Domain Tone/Style Configuration

**Description**: Configurable tone (regulatory, academic, engineering).

**Priority**: Full

**Dependencies**: 3.7

**User Stories**:
- As a user, I want to select the tone/style of reports based on audience

**Acceptance Criteria**:
- Three tones available: regulatory, academic, engineering
- Tone affects vocabulary, sentence structure, citation style
- Configurable per report or as default

---

#### 5.10 Multi-Format Output

**Description**: Reports exportable as markdown, PDF, JSON.

**Priority**: MVP (markdown)

**Dependencies**: 3.7

**User Stories**:
- As a user, I want reports in formats I can use with my existing tools

**Acceptance Criteria**:
- Markdown output works for MVP
- PDF output via LaTeX/pandoc for Release 2
- JSON output for programmatic consumption for Release 3

---

#### 5.11 CLI Delivery

**Description**: Direct terminal output.

**Priority**: MVP

**Dependencies**: 3.8

**User Stories**:
- As a CLI user, I want reports printed directly to terminal

**Acceptance Criteria**:
- Report displayed in terminal after generation
- Markdown rendered for readability
- No external dependencies required

---

#### 5.12 Telegram Delivery

**Description**: Bot message with formatted report.

**Priority**: Full

**Dependencies**: 3.1

**User Stories**:
- As a mobile user, I want reports delivered to Telegram

**Acceptance Criteria**:
- Hermes gateway configured for Telegram
- Report formatted for Telegram message
- Long reports split into multiple messages

---

#### 5.13 Discord Delivery

**Description**: Channel message with embed.

**Priority**: Full

**Dependencies**: 3.1

**User Stories**:
- As a team member, I want reports posted to Discord channels

**Acceptance Criteria**:
- Hermes gateway configured for Discord
- Reports sent as embeds with proper formatting
- Thread support for long reports

---

#### 5.14 Slack Delivery

**Description**: Channel or DM with formatted report.

**Priority**: Full

**Dependencies**: 3.1

**User Stories**:
- As a professional, I want reports delivered to Slack channels or DMs

**Acceptance Criteria**:
- Hermes gateway configured for Slack
- Reports sent as formatted messages
- Channel and DM delivery supported

---

#### 5.15 Email Delivery

**Description**: Formatted email body.

**Priority**: Full

**Dependencies**: 3.1

**User Stories**:
- As a professional, I want reports delivered via email

**Acceptance Criteria**:
- Hermes gateway configured for email
- Report sent as formatted email body
- Attachment option for PDF reports

---

#### 5.16 WhatsApp/Signal Delivery

**Description**: Text message delivery.

**Priority**: Full

**Dependencies**: 3.1

**User Stories**:
- As a mobile user, I want reports delivered to WhatsApp or Signal

**Acceptance Criteria**:
- Hermes gateway configured for WhatsApp and Signal
- Messages formatted for chat platforms
- Long reports split into digestible chunks

---

#### 5.17 Scheduled Reports (Hermes Cron)

**Description**: Recurring generation via `hermes cron add --schedule "0 9 * * 1" --deliver-to slack`.

**Priority**: Full

**Dependencies**: 5.x delivery channel

**User Stories**:
- As a manager, I want weekly reports generated and delivered automatically

**Acceptance Criteria**:
- Cron scheduling via Hermes cron
- Configurable schedule (cron expression)
- Configurable delivery channel(s)
- Failure notifications on missed reports

---

## 6. Evaluation

### Overview
Quality measurement, testing, and human review infrastructure.

### Features

---

#### 6.1 Automated Faithfulness Check

**Description**: Parse report claims → query AgentDB for source nodes → verify citation → score.

**Priority**: Full

**Dependencies**: 1.7, 5.6

**User Stories**:
- As a quality manager, I want automated verification that every claim in a report is backed by a source

**Acceptance Criteria**:
- Report parsed into individual claims
- Each claim queried against AgentDB
- Score calculated: % of claims with valid source backing
- Results stored with report

---

#### 6.2 Diagram Accuracy Evaluation

**Description**: Human spot-check 10% of symbolic graphs vs source images.

**Priority**: Full

**Dependencies**: 2.3

**User Stories**:
- As a domain expert, I want to verify diagram interpretations are accurate

**Acceptance Criteria**:
- CLI presents diagram → symbolic graph pair for review
- Reviewer scores accuracy (1-5)
- Scores aggregated for quality metrics
- 10% sample rate configurable

---

#### 6.3 Report Quality Rubric

**Description**: 1-5 scale: completeness, tone, technical accuracy, citation quality, uncertainty handling.

**Priority**: Full

**Dependencies**: 5.x

**User Stories**:
- As a quality manager, I want standardized scoring of report quality

**Acceptance Criteria**:
- Rubric defined: completeness, tone, technical accuracy, citation quality, uncertainty handling
- Each dimension scored 1-5
- Scores stored with report metadata
- Trend view over time

---

#### 6.4 Contradiction Detection Rate

**Description**: % of queries where conflicting sources are surfaced; test suite of known contradictions.

**Priority**: Full

**Dependencies**: 2.10

**User Stories**:
- As a quality manager, I want to measure how effectively the system surfaces contradictions

**Acceptance Criteria**:
- Test suite of known contradictions created
- System's detection rate measured
- Target: >80% detection rate

---

#### 6.5 Hermes Trajectory Analysis

**Description**: Export sessions, analyze success/failure patterns, detect recurring failure modes.

**Priority**: Full

**Dependencies**: 4.5

**User Stories**:
- As a developer, I want to analyze agent trajectories to identify failure patterns

**Acceptance Criteria**:
- Sessions exportable as JSON
- Analysis tool identifies common failure modes
- Reports generated with improvement recommendations

---

#### 6.6 Human Review Interface

**Description**: Simple CLI for presenting diagram → symbolic graph pairs for review.

**Priority**: Full

**Dependencies**: 6.2

**User Stories**:
- As a reviewer, I want a simple CLI for batch review of diagram interpretations

**Acceptance Criteria**:
- CLI presents one pair at a time
- Accepts score and optional comment
- Progress tracked through review queue

---

#### 6.7 A/B Testing

**Description**: Compare different agent configurations or fine-tuned vs prompted quality.

**Priority**: Full

**Dependencies**: 6.3

**User Stories**:
- As a developer, I want to compare different agent configurations to find the best performing

**Acceptance Criteria**:
- Two configurations can be run on same query set
- Results scored per rubric
- Statistical comparison report

---

#### 6.8 Citation Accuracy Measurement

**Description**: Metric: % of answer claims traceable to source nodes.

**Priority**: MVP

**Dependencies**: 6.1

**User Stories**:
- As a user, I want visibility into how grounded the system's answers are

**Acceptance Criteria**:
- Citation accuracy metric calculated per response
- Goal: >90% for MVP
- Metric logged with each session

---

## 7. Governance & Infrastructure

### Overview
Paperclip-based governance layer providing org structure, budgets, audit, and optional API/n8n infrastructure.

### Features

---

#### 7.1 Paperclip Governance Layer

**Description**: Org charts, agent roles, reporting structure.

**Priority**: Full

**Dependencies**: Paperclip installed

**User Stories**:
- As an organization, I want to define agent roles and reporting structures

**Acceptance Criteria**:
- Org chart defined in Paperclip
- Agent roles assigned
- Reporting hierarchy established

---

#### 7.2 Budget Management

**Description**: Monthly budgets per agent with automatic stop at limit.

**Priority**: Full

**Dependencies**: 7.1

**User Stories**:
- As a finance manager, I want to set budgets per agent and stop spending when limits are reached

**Acceptance Criteria**:
- Budget defined per agent (token/$ limits)
- Automatic stop when budget exceeded
- Budget usage reporting dashboard

---

#### 7.3 Mission/Goal Alignment

**Description**: Top-level goals that trace down to agent tasks.

**Priority**: Full

**Dependencies**: 7.1

**User Stories**:
- As a leader, I want to define organizational goals that guide agent priorities

**Acceptance Criteria**:
- Top-level goals defined in Paperclip
- Agent tasks traceable to goals
- Goal completion tracking

---

#### 7.4 Paperclip Audit Log

**Description**: Full tool-call tracing and immutable record of all agent actions.

**Priority**: Full

**Dependencies**: 7.1

**User Stories**:
- As a compliance officer, I want an immutable record of all agent actions

**Acceptance Criteria**:
- All tool calls logged with timestamp, agent, parameters, result
- Log is append-only and immutable
- Log searchable and exportable

---

#### 7.5 Multi-Company Isolation

**Description**: One deployment, many companies with complete data isolation.

**Priority**: Full

**Dependencies**: 7.1

**User Stories**:
- As a SaaS provider, I want complete data isolation between tenants

**Acceptance Criteria**:
- Companies fully isolated in Paperclip
- No cross-company data leakage
- Per-company configuration

---

#### 7.6 Paperclip Dashboard

**Description**: React UI for monitoring agent activity, costs, and work.

**Priority**: Full

**Dependencies**: 7.1, 7.2, 7.4

**User Stories**:
- As a manager, I want a dashboard for real-time monitoring of agent activity

**Acceptance Criteria**:
- Dashboard shows: active agents, recent actions, costs, budget status
- React UI with filtering and search
- Real-time updates

---

#### 7.7 FastAPI REST API (Optional)

**Description**: Programmatic endpoints for ingestion, query, report status, knowledge exploration.

**Priority**: Full

**Dependencies**: 3.x, 2.x

**User Stories**:
- As a developer, I want to integrate Agent Frosty into my applications via REST API

**Acceptance Criteria**:
- Endpoints: POST /ingest, GET /query, GET /report/{id}, GET /knowledge/search
- Authentication via API key
- OpenAPI documentation

---

#### 7.8 Authentication (Optional)

**Description**: API key or JWT-based auth for REST API.

**Priority**: Full

**Dependencies**: 7.7

**User Stories**:
- As a security officer, I want API access authenticated

**Acceptance Criteria**:
- API key authentication implemented
- JWT token auth as alternative
- Token expiration and rotation supported

---

#### 7.9 n8n as MCP Pipeline Tool (Optional)

**Description**: Visual workflow construction for specific pipeline stages (document processing, diagram transformation).

**Priority**: Full

**Dependencies**: n8n installed, MCP integration

**User Stories**:
- As a workflow designer, I want to visually construct pipelines for data processing

**Acceptance Criteria**:
- n8n connected via MCP
- Workflows defined for document processing and diagram transformation
- Workflows triggerable from Hermes

---

## 8. Security & Compliance

### Overview
Security measures including encryption, access control, redaction, and command approval.

### Features

---

#### 8.1 AgentDB Encryption

**Description**: Database encryption at rest.

**Priority**: Full

**Dependencies**: 1.3

**User Stories**:
- As a security officer, I want stored knowledge encrypted at rest

**Acceptance Criteria**:
- All AgentDB databases encrypted at rest
- Encryption key managed via environment variable or key management service
- No performance degradation >10%

---

#### 8.2 Paperclip Role-Based Access

**Description**: Granular access to ingestion, query, and admin.

**Priority**: Full

**Dependencies**: 7.1

**User Stories**:
- As a security officer, I want role-based access to system functions

**Acceptance Criteria**:
- Roles: admin, editor (ingestion), viewer (query only)
- Permissions enforced at API and CLI levels
- Role assignment via Paperclip

---

#### 8.3 Source Redaction

**Description**: Mark sources as confidential; restrict output visibility to authorized users.

**Priority**: Full

**Dependencies**: 8.2

**User Stories**:
- As a security officer, I want to mark sensitive sources and restrict who can see them

**Acceptance Criteria**:
- Sources can be flagged as confidential
- Confidential sources hidden from unauthorized users
- Reports redact citations to confidential sources for unauthorized readers

---

#### 8.4 Hermes Command Approval

**Description**: Security: command approval patterns for agent actions.

**Priority**: Full

**Dependencies**: 3.1

**User Stories**:
- As a security officer, I want destructive or expensive actions to require approval

**Acceptance Criteria**:
- Configurable approval patterns (e.g., delete, batch ingest, API calls)
- Approval requested via Hermes before execution
- Approval timeout and expiration

---

## 9. Monitoring & Observability

### Overview
Performance metrics, session monitoring, alerting, and quality dashboards.

### Features

---

#### 9.1 Performance Metrics

**Description**: Vector search latency, agent response time, token usage.

**Priority**: Full

**Dependencies**: 1.7, 3.1

**User Stories**:
- As a system operator, I want performance metrics for capacity planning and troubleshooting

**Acceptance Criteria**:
- Metrics collected: search latency, agent response time, token usage per session
- Metrics stored with timestamps
- Queryable via CLI or dashboard

---

#### 9.2 Hermes Session Monitoring

**Description**: Track sessions for failures and slow responses.

**Priority**: MVP (built-in)

**Dependencies**: 3.1

**User Stories**:
- As a developer, I want session monitoring to identify issues

**Acceptance Criteria**:
- Session list with status, duration, token count
- Filterable by success/failure
- Slow session detection (configurable threshold)

---

#### 9.3 Alerting

**Description**: Alerts on failures, slow queries, low faithfulness scores.

**Priority**: Full

**Dependencies**: 9.2, 6.1

**User Stories**:
- As a system operator, I want alerts when things go wrong

**Acceptance Criteria**:
- Alert rules configurable: failure rate, latency threshold, faithfulness score
- Alert channels: CLI notification, email, Slack
- Alert history and acknowledgment

---

#### 9.4 Quality Trend Dashboard

**Description**: Report quality scores over time.

**Priority**: Full

**Dependencies**: 6.3, 9.2

**User Stories**:
- As a quality manager, I want to see quality trends to detect regression

**Acceptance Criteria**:
- Quality scores plotted over time
- Filterable by date range, report type, agent
- Regression alerts when scores drop below threshold

---

## 10. Technology Integration

### Overview
Core platform integrations: Hermes, AgentDB, Paperclip, Pi, n8n, OpenCode, MCP, model providers.

### Features

---

#### 10.1 Hermes-Agent Runtime

**Description**: Self-improving agent with built-in learning loop, subagents, MCP, cron, multi-platform delivery.

**Priority**: MVP

**Dependencies**: None

**User Stories**:
- As a user, I want a capable agent runtime that handles the heavy lifting

**Acceptance Criteria**:
- Hermes installed and configured
- All core features functional: learning loop, subagents, MCP, cron, delivery

---

#### 10.2 AgentDB Vector Knowledge Store

**Description**: High-performance vector database with HNSW, quantization, hybrid search.

**Priority**: MVP

**Dependencies**: None

**User Stories**:
- As a developer, I want a fast, embedded vector database for knowledge storage

**Acceptance Criteria**:
- AgentDB initialized with multiple databases
- HNSW indexing, quantization, hybrid search working
- MCP server running

---

#### 10.3 Paperclip Governance

**Description**: Org charts, budgets, goal alignment, audit.

**Priority**: Full

**Dependencies**: Paperclip installed

**User Stories**:
- As an organization, I want governance over agent operations

**Acceptance Criteria**:
- Paperclip installed and connected
- Org chart, budgets, goals, audit working

---

#### 10.4 Pi LLM Abstraction (Optional)

**Description**: Unified multi-provider LLM API for finer-grained model control.

**Priority**: Full

**Dependencies**: None

**User Stories**:
- As a developer, I want a unified API for different LLM providers

**Acceptance Criteria**:
- Pi installed and configured
- Multi-provider switching working
- Integrated with Hermes if needed

---

#### 10.5 n8n Pipeline Tool (Optional)

**Description**: MCP-connected visual workflow for specific pipeline stages.

**Priority**: Full

**Dependencies**: n8n installed

**User Stories**:
- As a workflow designer, I want visual pipeline construction

**Acceptance Criteria**:
- n8n connected via MCP
- Workflows executable from Hermes

---

#### 10.6 OpenCode Development Tool

**Description**: Build + plan agents for SPARC methodology during development.

**Priority**: N/A

**Dependencies**: None

**User Stories**:
- As a developer, I want agent-assisted development using SPARC methodology

**Acceptance Criteria**:
- OpenCode used for development workflow
- SPARC methodology followed

---

#### 10.7 MCP Integration

**Description**: Standardized tool protocol connecting Hermes to AgentDB, n8n, Pi, and other services.

**Priority**: MVP

**Dependencies**: 3.1

**User Stories**:
- As a developer, I want standardized tool protocol for all integrations

**Acceptance Criteria**:
- MCP server for AgentDB running
- Hermes discovers and calls MCP tools
- Error handling and retry logic

---

#### 10.8 Multi-Model Provider Support

**Description**: OpenRouter (200+ models), Nous Portal, OpenAI, Anthropic, Google, local.

**Priority**: MVP

**Dependencies**: 3.1

**User Stories**:
- As a user, I want to choose my LLM provider and switch freely

**Acceptance Criteria**:
- Hermes configured with multiple providers
- Single command switches provider
- All providers work with Agent Frosty skills

---

#### 10.9 agentskills.io Skill Format

**Description**: Open standard for modular, reusable agent skills.

**Priority**: MVP

**Dependencies**: 3.1

**User Stories**:
- As a developer, I want skills that follow an open standard for portability

**Acceptance Criteria**:
- Skills follow agentskills.io specification
- Skills shareable and reusable
- Documentation generated from skill metadata

---

## 11. Image Catalogue

### Overview
Image catalogue system for ingesting, describing, organizing, searching, generating, and managing thousands of images. Combines vision LLM auto-description, user text/voice input, graph-based storage, similarity search, and diffusion-based image generation.

### Features

---

#### 11.1 Vision LLM Image Description

**Description**: Specialist Image Description Agent uses vision LLM to auto-generate detailed descriptions of uploaded images — extracting objects, scenes, text, relationships, color palettes, composition.

**Priority**: MVP

**Dependencies**: 11.4 (graph storage)

**User Stories**:
- As a cataloguer, I want images automatically described so I don't have to write descriptions manually for every image
- As a researcher, I want detailed structured descriptions extracted from images including objects, scenes, and text

**Acceptance Criteria**:
- Vision LLM processes uploaded images and generates description with: objects detected, scene type, visible text, color palette, composition notes
- Description stored as structured data in the image graph node
- Processing handles common formats: JPEG, PNG, WEBP, TIFF
- Batch processing supported for multiple images
- Minimum description accuracy: objects correctly identified in >80% of test images

---

#### 11.2 User Text Description Update

**Description**: User can create or update image descriptions via text input — supports both editing existing auto-generated descriptions and writing new ones from scratch.

**Priority**: MVP

**Dependencies**: 11.1

**User Stories**:
- As a cataloguer, I want to correct or improve auto-generated descriptions with my own text
- As a user, I want to write descriptions from scratch for images where auto-description wasn't run

**Acceptance Criteria**:
- User can edit any field of the auto-generated description
- User can replace the entire description with their own text
- Previous versions of descriptions are preserved (see 11.9 versioning)
- Edits attributed to the user with timestamp

---

#### 11.3 Voice Description Input

**Description**: User can create or update image descriptions via voice recording — speech-to-text transcription mapped to description fields.

**Priority**: Full

**Dependencies**: 11.2

**User Stories**:
- As a field researcher, I want to dictate descriptions hands-free while reviewing images
- As an accessibility user, I want voice input as an alternative to typing

**Acceptance Criteria**:
- Voice recording captures audio and transcribes via speech-to-text (Whisper)
- Transcription mapped to description fields
- User can review and edit transcription before saving
- Multiple languages supported (matching Whisper language support)

---

#### 11.4 Graph-Based Description Storage

**Description**: Descriptions stored in graph with nodes for image, description version, metadata; edges link to source file path, derived-from relationships, related images.

**Priority**: MVP

**Dependencies**: 1.1, 1.3

**User Stories**:
- As a developer, I want descriptions stored as graph nodes so relationships between images and their descriptions are queryable
- As a user, I want to trace from a description back to the original image file

**Acceptance Criteria**:
- Image node stores: file path, hash, dimensions, format, description
- Description versions stored as child nodes with version edges
- Metadata stored as node properties
- Bidirectional edges linking related images
- Graph queryable via AgentDB MCP

---

#### 11.5 Image Metadata Extraction

**Description**: Auto-extract EXIF, camera specs, GPS location, timestamp, file format, resolution, color profile, compression — stored as graph node properties.

**Priority**: MVP

**Dependencies**: 11.4

**User Stories**:
- As a photographer, I want camera metadata automatically extracted and stored with each image
- As a cataloguer, I want to search images by camera, lens, GPS location, date taken

**Acceptance Criteria**:
- EXIF data extracted: camera make/model, lens, aperture, shutter speed, ISO, focal length
- GPS coordinates extracted where present
- File metadata: format, resolution, color profile, file size, compression
- All metadata stored as properties on the image graph node
- Missing metadata fields gracefully handled (null/unknown)

---

#### 11.6 Context-Aware Image Generation

**Description**: Generate new images based on combined context of metadata, descriptions, and similarity to existing images using diffusion models (Stable Diffusion, DALL-E, Midjourney).

**Priority**: Full

**Dependencies**: 11.1, 11.4, 11.7

**User Stories**:
- As a designer, I want to generate new images that match the style and content of my existing catalogue
- As a researcher, I want to visualize variations of an existing image concept

**Acceptance Criteria**:
- Generation accepts: reference image(s), description text, similarity parameters
- Context from reference images (metadata + descriptions) injected into prompt
- Multiple diffusion model providers supported (configurable)
- Generated images auto-ingested into catalogue with provenance linking to source images
- Generation parameters (steps, guidance, seed, model) stored with generated image metadata

---

#### 11.7 Similarity-Based Image Search

**Description**: Find visually similar images via vector embedding comparison across the image catalogue.

**Priority**: Full

**Dependencies**: 11.4, 11.5

**User Stories**:
- As a cataloguer, I want to find all images visually similar to a reference image
- As a user, I want to discover unexpected visual relationships in my catalogue

**Acceptance Criteria**:
- Images embedded as vectors (via CLIP or similar vision embedding model)
- Similarity search returns ranked results with similarity score
- Search by: uploaded reference image, selected existing image
- Threshold configurable (minimum similarity score)
- Results display similarity score percentage

---

#### 11.8 Image Tagging & Taxonomy

**Description**: Auto-tag images with hierarchical taxonomy tags; user-curated tag sets; faceted tag-based filtering.

**Priority**: Full

**Dependencies**: 11.1, 11.4

**User Stories**:
- As a cataloguer, I want images automatically tagged so I can filter and organize
- As a taxonomy manager, I want to define custom tag hierarchies

**Acceptance Criteria**:
- Auto-tagging from vision LLM description (objects, scenes, colors, styles)
- User can add, remove, or modify tags
- Hierarchical taxonomy supported (e.g., Vehicle > Car > Sedan)
- Faceted filtering: filter by multiple tags simultaneously
- Tag suggestions based on existing tags in similar images

---

#### 11.9 Image Versioning

**Description**: Track multiple versions of the same image; provenance chain (original → edited → derived); diff view between versions.

**Priority**: Full

**Dependencies**: 11.4

**User Stories**:
- As a designer, I want to track all versions of an image and understand the edit history
- As a reviewer, I want to compare different versions of the same image

**Acceptance Criteria**:
- Each image version stored as a child node linked via `derived_from` edge
- Version metadata includes: timestamp, editor, tool used, description of change
- Visual diff between any two versions
- Version tree display showing branching and merges

---

#### 11.10 Provenance Tracking

**Description**: Record image origin (uploaded, generated, imported from tool), edit history, transformations applied, source attribution.

**Priority**: Full

**Dependencies**: 11.4, 11.9

**User Stories**:
- As a compliance officer, I want to know the complete provenance of every image
- As a user, I want to trace generated images back to their source images and prompts

**Acceptance Criteria**:
- Origin recorded: upload, generation, import, external API
- For generated images: model, prompt, seed, parameters, source reference images
- For edited images: tool, parameters, before/after hash
- Provenance chain fully queryable through graph traversal
- Exportable as provenance report

---

#### 11.11 Image Relationship Mapping

**Description**: Define and visualize relationships between images: derived-from, similar-to, contains, part-of, predecessor-of.

**Priority**: Full

**Dependencies**: 11.4, 11.7

**User Stories**:
- As a cataloguer, I want to define and see relationships between images
- As a researcher, I want to understand how images relate to each other in the collection

**Acceptance Criteria**:
- Relationship types: derived_from, similar_to, contains, part_of, predecessor_of
- User can create, edit, delete relationships
- Auto-similarity edges created by 11.7
- Graph visualization of image relationships
- Relationship paths queryable (e.g., "show all images derived from this original")

---

#### 11.12 Bulk Image Import

**Description**: Upload images in bulk with drag-and-drop, folder recursion, progress tracking, duplicate detection.

**Priority**: Full

**Dependencies**: 11.1, 11.4, 11.5

**User Stories**:
- As a cataloguer, I want to import thousands of images at once without manual processing
- As a user, I want to drag and drop entire folders for import

**Acceptance Criteria**:
- Supports: drag-and-drop, folder selection, file dialog
- Recursive folder import (preserves folder structure as tags)
- Progress tracking: X of Y images processed
- Duplicate detection via perceptual hash (see 11.13)
- Configurable auto-description on import (on/off)
- Import report: success count, failure count, duplicates skipped

---

#### 11.13 Image Deduplication

**Description**: Perceptual hashing (pHash/dHash) to detect near-duplicate images during import and on-demand.

**Priority**: Full

**Dependencies**: 11.4

**User Stories**:
- As a cataloguer, I want to avoid importing the same image twice
- As a user, I want to find and remove near-duplicate images in my existing catalogue

**Acceptance Criteria**:
- Perceptual hash computed on import (pHash and dHash)
- Near-duplicates detected with configurable threshold
- On-demand dedup scan of existing catalogue
- Duplicates presented for user decision: skip, keep both, replace
- Similarity score displayed for near-duplicate pairs

---

#### 11.14 Image Export

**Description**: Export images with metadata, descriptions, and provenance as ZIP/SQLite bundle; individual image download.

**Priority**: Full

**Dependencies**: 11.4, 11.5

**User Stories**:
- As a user, I want to export images with all their metadata and descriptions for backup or transfer
- As a cataloguer, I want to share a subset of images with full context

**Acceptance Criteria**:
- Export format: ZIP with image files + JSON/SQLite metadata bundle
- Selectable image subset (individual, by tag, by search results)
- Metadata included: descriptions, tags, EXIF, provenance, relationships
- Individual image download with embedded metadata
- Import function to re-import exported bundles

---

#### 11.15 Image Grid/Browser UI

**Description**: Searchable, filterable grid view of catalogue with thumbnail previews, sorting, pagination, zoom.

**Priority**: MVP

**Dependencies**: 11.4, 11.5

**User Stories**:
- As a user, I want to browse my image catalogue visually in a grid layout
- As a cataloguer, I want to sort and filter to quickly find specific images

**Acceptance Criteria**:
- Grid layout with configurable thumbnail size
- Sort by: name, date, size, rating, upload date
- Filter by: tag, metadata field, date range, GPS location
- Pagination or infinite scroll
- Click thumbnail to open detail view (see 11.16)
- Zoom on hover for quick preview

---

#### 11.16 Metadata Editor UI

**Description**: Inline editor for viewing/editing all image metadata fields, description text, tags.

**Priority**: MVP

**Dependencies**: 11.2, 11.5

**User Stories**:
- As a cataloguer, I want a single screen where I can view and edit all image information
- As a user, I want to update descriptions, tags, and metadata in one place

**Acceptance Criteria**:
- Image preview displayed at full resolution
- All metadata fields editable (description, tags, title, notes)
- Auto-generated description shown with edit button
- Save/cancel for edits
- Version automatically created on save
- Keyboard shortcuts for efficient editing workflow

---

#### 11.17 Image Comparison View UI

**Description**: Side-by-side and overlay comparison of two or more images; metadata diff display.

**Priority**: Full

**Dependencies**: 11.16

**User Stories**:
- As a reviewer, I want to compare two versions of the same image side by side
- As a quality controller, I want to overlay images to spot differences

**Acceptance Criteria**:
- Side-by-side mode: two images displayed with synchronized zoom/pan
- Overlay mode: images stacked with opacity slider
- Metadata diff: fields that changed highlighted between versions
- Supported for 2+ images (up to 4 for side-by-side)

---

#### 11.18 Bulk Operations UI

**Description**: Select multiple images for batch tag/edit/export/delete with confirmation and undo.

**Priority**: Full

**Dependencies**: 11.15, 11.14, 11.8

**User Stories**:
- As a cataloguer, I want to apply tags or edits to many images at once
- As a user, I want to export or delete multiple images in one action

**Acceptance Criteria**:
- Multi-select via checkbox, shift-click, Ctrl+A, drag-select
- Actions: add tag, remove tag, edit description field, export, delete
- Batch operation preview (shows affected count)
- Confirmation dialog for destructive operations
- Undo/rollback for recent batch operations

---

#### 11.19 Search & Filter UI

**Description**: Full-text search across descriptions/metadata; faceted filters by tag, date, resolution, camera, GPS location.

**Priority**: MVP

**Dependencies**: 11.15, 11.4

**User Stories**:
- As a user, I want to search my image catalogue by keywords found in descriptions or metadata
- As a cataloguer, I want to drill down using multiple filter criteria simultaneously

**Acceptance Criteria**:
- Full-text search across: description, title, tags, notes, filename
- Faceted filters: tag hierarchy, date range, camera make, resolution range, GPS bounding box
- Search results update in real-time as filters change
- Saved searches for reuse
- Search history with recent searches

---

#### 11.20 Voice Input UI

**Description**: Microphone button on image detail view; voice recording waveform; transcription preview before save.

**Priority**: Full

**Dependencies**: 11.3, 11.16

**User Stories**:
- As a mobile user, I want to dictate descriptions using voice input
- As an accessibility user, I want voice input integrated into the UI

**Acceptance Criteria**:
- Microphone button in metadata editor
- Recording indicator with waveform visualization
- Transcription preview before committing
- Edit transcription before save
- Cancel and re-record supported

---

#### 11.21 Image Annotation Tools UI

**Description**: Draw bounding boxes, arrows, text labels on images; attach annotations to specific coordinates with descriptions.

**Priority**: Full

**Dependencies**: 11.16

**User Stories**:
- As a researcher, I want to annotate specific regions of an image with notes
- As a reviewer, I want to highlight areas of interest with visual markers

**Acceptance Criteria**:
- Drawing tools: bounding box, arrow, line, freehand, text label
- Annotations stored with coordinates, description, author, timestamp
- Annotation layers toggle on/off
- Export annotations as COCO JSON or similar standard format
- Annotations searchable and filterable

---

#### 11.22 Image Generation Parameters UI

**Description**: Prompt builder with context injection; model selector (SD/DALL-E/Midjourney); parameter sliders (steps, guidance, seed); reference image selector; generation history.

**Priority**: Full

**Dependencies**: 11.6, 11.16

**User Stories**:
- As a designer, I want a visual interface for configuring image generation
- As a user, I want to browse and reuse past generation configurations

**Acceptance Criteria**:
- Prompt builder with context from selected reference images
- Model selector dropdown (SDXL, DALL-E 3, Midjourney)
- Sliders for: steps, guidance scale, seed (or random)
- Reference image selector with similarity preview
- Generation history with prompt, parameters, results
- One-click regeneration with different seed
- Generated images auto-saved to catalogue

---

## 12. Quotation & Proposal Management

### Overview
End-to-end quotation and proposal management system. Customers send requests via email, Slack, Discord, or Telegram; the system parses requirements, retrieves knowledge from AgentDB, generates quotes/proposals using Hermes, routes through an approval workflow, delivers to the customer by email, and keeps the internal user notified via their configured channels.

### Features

---

#### 12.1 Multi-Channel Inbound Requests

**Description**: Receive quote/proposal requests via email, Slack, Discord, and Telegram — unified inbox with channel attribution.

**Priority**: MVP

**Dependencies**: 5.12, 5.13, 5.14, 5.15 (delivery channels — reused for inbound)

**User Stories**:
- As a customer, I want to send a quote request via email and have it automatically picked up by the system
- As a customer, I want to send a quote request via Slack, Discord, or Telegram and get the same experience
- As a user, I want all inbound requests visible in a single unified inbox regardless of source channel

**Acceptance Criteria**:
- Email inbox monitored (IMAP or webhook) for quote requests; emails parsed for sender, subject, body, attachments
- Slack app/DM monitored for quote requests; messages parsed for intent, requirements, file attachments
- Discord bot monitored for quote requests in designated channel or DM
- Telegram bot monitored for quote requests
- All requests stored in a unified queue with channel attribution tag
- Duplicate detection: same customer sending via multiple channels merged to single request

---

#### 12.2 Request Parsing & Intent Extraction

**Description**: Parse incoming messages to extract customer details, requirements, quantities, deadlines, attachments; classify request type (quote, proposal, RFP response).

**Priority**: MVP

**Dependencies**: 12.1

**User Stories**:
- As a user, I want the system to automatically extract customer name, contact info, and requirements from the inbound message
- As a user, I want attachments on the request (specs, RFQ documents) automatically processed and linked

**Acceptance Criteria**:
- NLP extraction of: customer name, email, company, phone, project description, quantities, required delivery date
- Attachments classified (spec sheet, RFQ, reference image) and linked to request
- Request type classification: quote (simple pricing), proposal (detailed solution), RFP response (formal tender)
- Confidence score per extracted field; low-confidence fields flagged for manual review
- Extraction failures logged with raw message preserved for manual handling

---

#### 12.3 Knowledge-Backed Quote Generation

**Description**: Generate quotes using Hermes agent with retrieval from AgentDB — pricing data, standards, prior quotes, component costs, formula-based calculations.

**Priority**: MVP

**Dependencies**: 12.2, 1.7 (AgentDB MCP), 3.1 (Hermes)

**User Stories**:
- As a user, I want the system to automatically generate a quote based on the customer's requirements and my configured pricing data
- As a user, I want the system to reference previous similar quotes for consistency

**Acceptance Criteria**:
- Hermes agent retrieves relevant pricing data from AgentDB based on extracted requirements
- Prior quotes searched for similar requests and used as reference
- Formula-based calculations applied where pricing formulas exist in knowledge base
- Generated quote includes: line items with descriptions, quantities, unit prices, totals, terms, validity period
- All pricing decisions traceable to source data in AgentDB (provenance)
- Quote generated in under 60 seconds from request receipt

---

#### 12.4 Quote/Proposal Templates

**Description**: Configurable templates for different quote types, proposal formats, and RFP responses; domain-specific formatting and compliance language.

**Priority**: MVP

**Dependencies**: 12.3

**User Stories**:
- As a user, I want different templates for simple quotes vs detailed proposals vs formal RFP responses
- As a user, I want to customize templates with my company branding, terms, and boilerplate language

**Acceptance Criteria**:
- Template library with minimum 3 templates: standard quote, detailed proposal, RFP response
- Templates support variables (customer name, date, line items, totals) that are auto-populated
- Company branding (logo, colors, disclaimer) configurable per template
- Compliance boilerplate auto-inserted based on request type and jurisdiction
- Users can create and edit templates via configuration

---

#### 12.5 Approval Workflow

**Description**: Optional routing: draft → user approval → send; configurable approvers, thresholds (auto-approve under $X), rejection with revision notes.

**Priority**: MVP

**Dependencies**: 12.3

**User Stories**:
- As a manager, I want quotes over a certain value to require my approval before being sent
- As a user, I want to review and approve/reject quotes from my notification channel

**Acceptance Criteria**:
- Approval routing configurable: auto-approve under threshold, route to specific approver(s) over threshold
- Approval request sent to approver via their configured notification channel (see 12.7)
- Approver can approve, reject with notes, or request revision
- Rejected quotes return to drafting state with revision notes attached
- Approval history logged for audit
- Escalation if approver does not respond within configurable timeout

---

#### 12.6 Email Delivery to Customer

**Description**: Send generated quotes/proposals to customer email with branded formatting, PDF attachment, and tracked delivery status.

**Priority**: MVP

**Dependencies**: 12.3, 5.15

**User Stories**:
- As a customer, I want to receive the quote as a professional, branded email with a PDF attachment
- As a user, I want to know when the customer has received and opened the quote

**Acceptance Criteria**:
- Quote rendered as HTML email with company branding
- PDF attachment of the full quote/proposal generated automatically
- Delivery tracking: sent, delivered, opened, bounced
- Bounce handling: auto-notify user, flag for alternative delivery
- Email sent from configured sender address (e.g., quotes@company.com)

---

#### 12.7 Multi-Channel User Notifications

**Description**: Notify the internal user of status changes (request received, quote drafted, approved, sent, customer responded) via their configured channels (email, Slack, Discord, Telegram).

**Priority**: MVP

**Dependencies**: 12.1, 12.3, 12.5, 12.6

**User Stories**:
- As a user, I want to be notified on Slack when a new quote request arrives
- As a user, I want to receive status updates on my quote via Telegram so I can approve/reject on the go

**Acceptance Criteria**:
- User-configurable notification preferences: which events trigger notification, which channel(s)
- Notification events: request received, quote drafted, pending approval, approved, sent, delivered, opened, customer replied
- Notifications include: request ID, customer name, amount, status, action link (if applicable)
- Channel-specific formatting: Slack rich message, Discord embed, Telegram formatted text, email summary

---

#### 12.8 Quote Status Lifecycle

**Description**: Track full lifecycle: received → drafting → pending_approval → sent → viewed → accepted → rejected → revised; status visible in UI and notifications.

**Priority**: MVP

**Dependencies**: 12.1, 12.3, 12.5, 12.6

**User Stories**:
- As a user, I want to see the current status of every quote at a glance
- As a user, I want status changes to trigger appropriate notifications

**Acceptance Criteria**:
- Status machine: received → drafting → pending_approval → sent → viewed → accepted/rejected/expired → (if rejected) → revised
- Status transitions logged with timestamp and actor
- Dashboard or CLI command shows all quotes grouped by status
- Aging report: quotes stuck in a status beyond configurable threshold flagged

---

#### 12.9 Quote Versioning

**Description**: Track revisions; each edit creates a new version; full audit trail of who changed what and when; compare versions.

**Priority**: Full

**Dependencies**: 12.3

**User Stories**:
- As a user, I want to see the revision history of a quote so I can track changes
- As a manager, I want to compare different versions of a quote to see what was changed

**Acceptance Criteria**:
- Each save/edit creates a new version with incrementing version number
- Version metadata: timestamp, editor, change summary, status at version
- Diff view between any two versions showing line-item, pricing, and terms changes
- Full audit trail queryable by quote ID

---

#### 12.10 Customer Communication History

**Description**: Full transcript of all communications with a customer stored in graph — inbound requests, outbound quotes, follow-ups, revisions — linked to customer node.

**Priority**: Full

**Dependencies**: 12.1, 12.6, 1.1

**User Stories**:
- As a user, I want to see the complete communication history with a customer in one place
- As a manager, I want to review all interactions for quality assurance

**Acceptance Criteria**:
- All customer-facing communications stored as nodes in the knowledge graph
- Nodes linked to customer entity via `customer_of` relationship
- Communication types: inbound_request, outbound_quote, follow_up, revision, status_update
- Each node stores: timestamp, channel, content summary, attachments, agent/user attribution
- History queryable by customer, date range, communication type

---

#### 12.11 Quote Analytics Dashboard

**Description**: Acceptance rates, average response time, conversion by channel, revenue pipeline, most-requested items.

**Priority**: Full

**Dependencies**: 12.8

**User Stories**:
- As a manager, I want to see how many quotes are being accepted vs rejected
- As a sales manager, I want to know which channels generate the most valuable leads

**Acceptance Criteria**:
- Metrics: acceptance rate, average response time (request to send), average quote value, conversion by channel
- Revenue pipeline view: total value of quotes by status (draft, sent, accepted)
- Most-requested products/services ranking
- Trend view over configurable time periods (7d, 30d, 90d)
- Data exportable as CSV

---

#### 12.12 n8n Workflow Templates

**Description**: Pre-built n8n workflows covering the full quote lifecycle: inbound channel listener → parsing → knowledge retrieval → generation → approval → delivery → notification.

**Priority**: MVP

**Dependencies**: 7.9, 12.1–12.7

**User Stories**:
- As a workflow designer, I want pre-built n8n templates that I can customize for my quote process
- As a developer, I want to see the full quote flow as a visual n8n workflow

**Acceptance Criteria**:
- Minimum 3 n8n workflow templates: full lifecycle, approval-only, notification-only
- Templates include nodes for: channel listener (email/Slack/Discord/Telegram), LLM parsing, Hermes agent call, email delivery, notification dispatch
- Templates are documented with input/output specifications
- Templates importable into n8n with one command
- Templates can be modified and extended by users

---

#### 12.13 Paperclip Governance Integration

**Description**: Budget limits per quote, approval chains mapped to org chart, full audit trail of all quote actions, cost tracking.

**Priority**: Full

**Dependencies**: 7.2, 12.5

**User Stories**:
- As a finance manager, I want budget limits enforced on quote values
- As a compliance officer, I want a full audit trail of every quote action

**Acceptance Criteria**:
- Paperclip org chart defines approval hierarchy for quotes
- Per-agent or per-team budget limits enforced; quotes over budget flagged and routed for special approval
- Every quote action (create, edit, approve, send, reject) logged to Paperclip audit
- Cost tracking: total value of quotes sent, accepted, by agent/team

---

#### 12.14 CRM Integration (Optional)

**Description**: Sync customers, quotes, and status with external CRM (HubSpot, Salesforce) via API or n8n.

**Priority**: Full

**Dependencies**: 12.8

**User Stories**:
- As a sales user, I want quotes generated in Agent Frosty to appear in my CRM automatically
- As a manager, I want quote status to sync back from CRM to keep both systems consistent

**Acceptance Criteria**:
- Integration with HubSpot and Salesforce (configurable)
- Sync: create/update contact, create deal/opportunity, attach quote document
- Bidirectional status sync: CRM status changes reflected in Agent Frosty
- Sync configurable (real-time or periodic)
- Sync failures logged with retry logic
