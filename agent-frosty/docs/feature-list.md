# Agent Frosty — Feature List

> Compiled from: [agent-frosty.md](agent-frosty.md) (original concept), [implementation-plan-v2.md](implementation-plan-v2.md) (architecture), [prd-v2.md](prd-v2.md) (product requirements)

---

## 1. Knowledge & Storage

| # | Feature | Description | Source | MVP | Full |
|---|---|---|---|---|---|
| 1.1 | Domain ontology | Formal entity types (Component, Formula, Concept, Constraint, Method, FailureMode, Metric, Diagram, Paper, Standard) and relationships (depends_on, implements, constrains, derived_from, validated_by, contradicts, illustrated_by, measured_by) | agent-frosty.md §3.2, impl-plan-v2 §1, prd-v2 §1.1 | ✓ | ✓ |
| 1.2 | Ontology schema with provenance | Each node includes version, validity_range, confidence_score, source_refs for temporal/versioned knowledge tracking | agent-frosty.md §8.1, impl-plan-v2 §1 | ✓ | ✓ |
| 1.3 | AgentDB multi-database | Separate databases for ontology, documents, diagrams, formulas — each with tuned quantization and HNSW indexing | impl-plan-v2 §1, prd-v2 §1.2 | ✓ | ✓ |
| 1.4 | Binary quantization (32x) | For large document volumes in documents.db | impl-plan-v2 §1 | ✓ | ✓ |
| 1.5 | Scalar quantization (4x) | For high-precision ontology and formula queries | impl-plan-v2 §1 | ✓ | ✓ |
| 1.6 | HNSW indexing | Sub-100µs vector search across all databases | impl-plan-v2 §1 | ✓ | ✓ |
| 1.7 | AgentDB MCP server | Exposes AgentDB as an MCP tool for Hermes-Agent integration | impl-plan-v2 §1, prd-v2 §1.2 | ✓ | ✓ |
| 1.8 | Hybrid retrieval strategy | Combines graph traversal (dependency tracing, contradiction detection) with vector search (semantic similarity) | agent-frosty.md §4.3, impl-plan-v2 §3 | ✗ | ✓ |
| 1.9 | MMR result diversification | Maximal Marginal Relevance ensures diverse evidence, not just most similar | impl-plan-v2 §10 | ✗ | ✓ |
| 1.10 | Manual ontology population | Seed entities and relationships defined by domain expert | prd-v2 §1.3 | ✓ | ✓ |
| 1.11 | AgentDB sharding | Horizontal scaling by domain for large knowledge volumes | prd-v2 §9.1 | ✗ | ✓ |
| 1.12 | Caching strategy | Cache frequent subgraphs and query results | prd-v2 §9.1 | ✗ | ✓ |

---

## 2. Ingestion Pipeline

| # | Feature | Description | Source | MVP | Full |
|---|---|---|---|---|---|
| 2.1 | PDF paper ingestion | PDF → section segmentation → semantic chunking → embedding → metadata extraction → store in AgentDB | agent-frosty.md §3.1, impl-plan-v2 §2, prd-v2 §2.1 | ✓ | ✓ |
| 2.2 | Batch ingestion | Process 5-10 documents in a single run | prd-v2 §2.1 | ✓ | ✓ |
| 2.3 | Diagram ingestion (vision LLM) | Architectural diagrams → GPT-4V/Claude Sonnet → structured JSON component graph with connections, labels, load paths | agent-frosty.md §3.1, impl-plan-v2 §2, prd-v2 §2.2 | ✗ | ✓ |
| 2.4 | Diagram → ontology linking | Extracted components linked to ontology entities via AgentDB hybrid search | impl-plan-v2 §2, prd-v2 §2.2 | ✗ | ✓ |
| 2.5 | Text-diagram cross-reference | Diagram elements linked to sentences/phrases in source papers | prd-v2 §2.2 | ✗ | ✓ |
| 2.6 | Formula ingestion (math OCR) | Nougat/Pix2Struct → LaTeX extraction → variable-to-ontology mapping → semantic meaning storage | agent-frosty.md §3.1, impl-plan-v2 §2, prd-v2 §2.3 | ✗ | ✓ |
| 2.7 | Audio/video ingestion | Whisper transcription → speaker diarization → claim extraction → storage | agent-frosty.md §3.1, impl-plan-v2 §2, prd-v2 §2.4 | ✗ | ✓ |
| 2.8 | Report ingestion | NLP chunking + claim extraction from existing reports | agent-frosty.md §3.1 | ✗ | ✓ |
| 2.9 | Image ingestion | Vision captioning + object graph extraction | agent-frosty.md §3.1 | ✗ | ✓ |
| 2.10 | Conflict detection at ingest | Compare new claims against existing graph; create ConflictNode linking contradictory claims with provenance; flag for human review or surface in reports | agent-frosty.md §8.2, impl-plan-v2 §2, prd-v2 §2.5 | ✗ | ✓ |
| 2.11 | Confidence scoring per claim | Score claims based on source recency, authority, methodology quality | prd-v2 §2.5 | ✗ | ✓ |
| 2.12 | Ingestion as Hermes tool | Ingestion pipeline wrapped as a Hermes tool/MCP resource callable from skills | prd-v2 §2.1 | ✓ | ✓ |

---

## 3. Agent Architecture

| # | Feature | Description | Source | MVP | Full |
|---|---|---|---|---|---|
| 3.1 | Hermes-Agent runtime | Self-improving AI agent with built-in learning loop, subagent spawning, MCP integration | impl-plan-v2 §3, prd-v2 §3 | ✓ | ✓ |
| 3.2 | Parent Controller agent | Interprets user intent, plans workflow, spawns subagents, aggregates results, handles iterative refinement | agent-frosty.md §8.5, impl-plan-v2 §3, prd-v2 §4.6 | ✗ | ✓ |
| 3.3 | Ingestion Agent (Hermes skill) | Extracts structured knowledge from raw assets, updates AgentDB via MCP, flags conflicts | agent-frosty.md §6.1, impl-plan-v2 §3 | ✗ | ✓ |
| 3.4 | Diagram Interpretation Agent (Hermes skill) | Vision LLM → component graph → ontology linking → store diagram semantics | agent-frosty.md §6.1, impl-plan-v2 §3, prd-v2 §4.2 | ✗ | ✓ |
| 3.5 | Research Analyst Agent (Hermes skill) | Paper evaluation, claim extraction, graph comparison, contradiction detection, confidence scoring | agent-frosty.md §6.1, impl-plan-v2 §3, prd-v2 §4.3 | ✗ | ✓ |
| 3.6 | Reasoning Agent (Hermes skill) | Graph traversal, formula retrieval/variable substitution, constraint checking, multi-hop reasoning | agent-frosty.md §6.1, impl-plan-v2 §3, prd-v2 §4.4 | ✗ | ✓ |
| 3.7 | Report Writer Agent (Hermes skill) | Structured report generation with citations, uncertainty flagging, domain tone, multi-format output | agent-frosty.md §6.1, impl-plan-v2 §3, prd-v2 §4.5 | ✗ | ✓ |
| 3.8 | Single Analyst skill (MVP) | Prompt + AgentDB MCP retrieval + grounded answer + citation formatting | prd-v2 §4.1 | ✓ | ✓ |
| 3.9 | Hermes skill framework | agentskills.io format skills stored in `~/.hermes/skills/agent-frosty/` | impl-plan-v2 §3, prd-v2 §3.2 | ✓ | ✓ |
| 3.10 | Subagent spawning | Parent agent spawns specialist subagents via `hermes spawn` or `/run` for parallel work | impl-plan-v2 §3, prd-v2 §4.6 | ✗ | ✓ |
| 3.11 | Evidence bundle assembly | Standardized format for passing retrieval context and intermediate results between skills | impl-plan-v2 §3, prd-v2 §4.6 | ✗ | ✓ |

---

## 4. Expert Reasoning & Learning

| # | Feature | Description | Source | MVP | Full |
|---|---|---|---|---|---|
| 4.1 | Domain-expert system prompt | Every agent operates under: ground all claims, use formulas exactly, reference diagrams, flag uncertainty, no hallucination | agent-frosty.md §5.2, impl-plan-v2 §4 | ✓ | ✓ |
| 4.2 | Domain-specific CoT examples | 3-5 solved examples in context file showing graph traversal, formula application, citation, uncertainty handling | agent-frosty.md §8.3, impl-plan-v2 §4, prd-v2 §5.3 | ✓ | ✓ |
| 4.3 | Per-skill prompt specialization | Customized base prompt for each agent's specific role | prd-v2 §5.1 | ✗ | ✓ |
| 4.4 | Anti-pattern guide | Common mistakes to avoid (hallucination, speculation, missing citations) | prd-v2 §5.3 | ✓ | ✓ |
| 4.5 | Hermes trajectory tracking | Built-in recording of all agent interactions to `~/.hermes/sessions/` with FTS5 search | impl-plan-v2 §4, prd-v2 §5.2 | ✓ (auto) | ✓ |
| 4.6 | Hermes verdict judgment | Agent autonomously judges task success/failure | impl-plan-v2 §4 | ✓ (auto) | ✓ |
| 4.7 | Hermes skill creation from experience | After complex tasks, Hermes auto-creates skills capturing the successful pattern | impl-plan-v2 §4, prd-v2 §5.2 | ✓ (auto) | ✓ |
| 4.8 | Hermes skill self-improvement | Skills improve during use based on outcomes | impl-plan-v2 §4 | ✓ (auto) | ✓ |
| 4.9 | Cross-session memory (Honcho) | Dialectic user modeling persists preferences and knowledge across sessions | impl-plan-v2 §4, prd-v2 §5.2 | ✗ | ✓ |
| 4.10 | Dynamic few-shot retrieval | Hermes searches past sessions for relevant examples to inject into current context | agent-frosty.md §8.3, impl-plan-v2 §4 | ✗ | ✓ |
| 4.11 | Context file system | `~/.hermes/context/agent-frosty.md` contains domain primer, solved examples, anti-patterns | prd-v2 §5.3 | ✓ | ✓ |
| 4.12 | Report Writer LoRA fine-tuning | Optional fine-tuning of Report Writer only for stylistic consistency (requires 1000+ examples) | agent-frosty.md §6, impl-plan-v2 §7, prd-v2 §8.3 | ✗ | ✓ |

---

## 5. Report Generation & Delivery

| # | Feature | Description | Source | MVP | Full |
|---|---|---|---|---|---|
| 5.1 | Technical reports | Formal technical document with citations, formulas, diagrams | agent-frosty.md §7, impl-plan-v2 §5 | ✗ | ✓ |
| 5.2 | Compliance analyses | Regulatory/compliance focused reports with standard references | impl-plan-v2 §5 | ✗ | ✓ |
| 5.3 | Research summaries | Condensed findings from multiple sources | impl-plan-v2 §5 | ✗ | ✓ |
| 5.4 | Design reviews | Analysis of designs against standards and constraints | impl-plan-v2 §5 | ✗ | ✓ |
| 5.5 | Contradiction reports | Reports specifically surfacing conflicting sources | impl-plan-v2 §5 | ✗ | ✓ |
| 5.6 | Citation rendering | Every claim cites specific graph nodes with full provenance | impl-plan-v2 §5, prd-v2 §4.5 | ✓ | ✓ |
| 5.7 | Uncertainty flagging | Auto-flag unsupported claims or contradictory evidence in output | impl-plan-v2 §5, prd-v2 §4.5 | ✓ | ✓ |
| 5.8 | LaTeX formula rendering | Formulas rendered properly in reports | impl-plan-v2 §5 | ✗ | ✓ |
| 5.9 | Domain tone/style configuration | Configurable tone (regulatory, academic, engineering) | prd-v2 §4.5 | ✗ | ✓ |
| 5.10 | Multi-format output | Markdown, PDF, JSON | prd-v2 §4.5 | ✓ (markdown) | ✓ |
| 5.11 | CLI delivery | Direct terminal output | impl-plan-v2 §5, prd-v2 §7.3 | ✓ | ✓ |
| 5.12 | Telegram delivery | Bot message with formatted report | impl-plan-v2 §5, prd-v2 §7.3 | ✗ | ✓ |
| 5.13 | Discord delivery | Channel message with embed | impl-plan-v2 §5, prd-v2 §7.3 | ✗ | ✓ |
| 5.14 | Slack delivery | Channel or DM with formatted report | impl-plan-v2 §5, prd-v2 §7.3 | ✗ | ✓ |
| 5.15 | Email delivery | Formatted email body | impl-plan-v2 §5, prd-v2 §7.3 | ✗ | ✓ |
| 5.16 | WhatsApp/Signal delivery | Text message delivery | impl-plan-v2 §5 | ✗ | ✓ |
| 5.17 | Scheduled reports (Hermes cron) | Recurring generation via `hermes cron add --schedule "0 9 * * 1" --deliver-to slack` | impl-plan-v2 §5, prd-v2 §7.3 | ✗ | ✓ |

---

## 6. Evaluation

| # | Feature | Description | Source | MVP | Full |
|---|---|---|---|---|---|
| 6.1 | Automated faithfulness check | Parse report claims → query AgentDB for source nodes → verify citation → score | agent-frosty.md §8.6, impl-plan-v2 §6, prd-v2 §6.1 | ✗ | ✓ |
| 6.2 | Diagram accuracy evaluation | Human spot-check 10% of symbolic graphs vs source images | agent-frosty.md §8.6, impl-plan-v2 §6, prd-v2 §6.2 | ✗ | ✓ |
| 6.3 | Report quality rubric | 1-5 scale: completeness, tone, technical accuracy, citation quality, uncertainty handling | agent-frosty.md §8.6, impl-plan-v2 §6, prd-v2 §6.3 | ✗ | ✓ |
| 6.4 | Contradiction detection rate | % of queries where conflicting sources are surfaced; test suite of known contradictions | agent-frosty.md §8.6, impl-plan-v2 §6, prd-v2 §6.4 | ✗ | ✓ |
| 6.5 | Hermes trajectory analysis | Export sessions, analyze success/failure patterns, detect recurring failure modes | impl-plan-v2 §6, prd-v2 §6.5 | ✗ | ✓ |
| 6.6 | Human review interface | Simple CLI for presenting diagram → symbolic graph pairs for review | prd-v2 §6.2 | ✗ | ✓ |
| 6.7 | A/B testing | Compare different agent configurations or fine-tuned vs prompted quality | prd-v2 §8.3 | ✗ | ✓ |
| 6.8 | Citation accuracy measurement | Metric: % of answer claims traceable to source nodes | prd-v2 §1.4 | ✓ | ✓ |

---

## 7. Governance & Infrastructure

| # | Feature | Description | Source | MVP | Full |
|---|---|---|---|---|---|
| 7.1 | Paperclip governance layer | Org charts, agent roles, reporting structure | impl-plan-v2 §3, prd-v2 §7.2 | ✗ | ✓ |
| 7.2 | Budget management | Monthly budgets per agent with automatic stop at limit | impl-plan-v2 §3, prd-v2 §7.2 | ✗ | ✓ |
| 7.3 | Mission/goal alignment | Top-level goals that trace down to agent tasks | prd-v2 §7.2 | ✗ | ✓ |
| 7.4 | Paperclip audit log | Full tool-call tracing and immutable record of all agent actions | prd-v2 §7.2 | ✗ | ✓ |
| 7.5 | Multi-company isolation | One deployment, many companies with complete data isolation | prd-v2 §7.2 | ✗ | ✓ |
| 7.6 | Paperclip dashboard | React UI for monitoring agent activity, costs, and work | impl-plan-v2 §1, prd-v2 §9.3 | ✗ | ✓ |
| 7.7 | FastAPI REST API (optional) | Programmatic endpoints for ingestion, query, report status, knowledge exploration | prd-v2 §7.4 | ✗ | ✓ |
| 7.8 | Authentication (optional) | API key or JWT-based auth for REST API | prd-v2 §7.4 | ✗ | ✓ |
| 7.9 | n8n as MCP pipeline tool (optional) | Visual workflow construction for specific pipeline stages (document processing, diagram transformation) | impl-plan-v2 §3, prd-v2 §7.5 | ✗ | ✓ |

---

## 8. Security & Compliance

| # | Feature | Description | Source | MVP | Full |
|---|---|---|---|---|---|
| 8.1 | AgentDB encryption | Database encryption at rest | prd-v2 §9.2 | ✗ | ✓ |
| 8.2 | Paperclip role-based access | Granular access to ingestion, query, and admin | prd-v2 §9.2 | ✗ | ✓ |
| 8.3 | Source redaction | Mark sources as confidential; restrict output visibility to authorized users | prd-v2 §9.2 | ✗ | ✓ |
| 8.4 | Hermes command approval | Security: command approval patterns for agent actions | impl-plan-v2 §3 | ✗ | ✓ |

---

## 9. Monitoring & Observability

| # | Feature | Description | Source | MVP | Full |
|---|---|---|---|---|---|
| 9.1 | Performance metrics | Vector search latency, agent response time, token usage | prd-v2 §9.3 | ✗ | ✓ |
| 9.2 | Hermes session monitoring | Track sessions for failures and slow responses | prd-v2 §9.3 | ✓ (built-in) | ✓ |
| 9.3 | Alerting | Alerts on failures, slow queries, low faithfulness scores | prd-v2 §9.3 | ✗ | ✓ |
| 9.4 | Quality trend dashboard | Report quality scores over time | prd-v2 §9.3 | ✗ | ✓ |

---

## 10. Technology Integration

| # | Feature | Description | Source | MVP | Full |
|---|---|---|---|---|---|
| 10.1 | Hermes-Agent runtime | Self-improving agent with built-in learning loop, subagents, MCP, cron, multi-platform delivery | impl-plan-v2 §1, prd-v2 §3 | ✓ | ✓ |
| 10.2 | AgentDB vector knowledge store | High-performance vector database with HNSW, quantization, hybrid search | impl-plan-v2 §1, prd-v2 §1.2 | ✓ | ✓ |
| 10.3 | Paperclip governance | Org charts, budgets, goal alignment, audit | impl-plan-v2 §1, prd-v2 §7.2 | ✗ | ✓ |
| 10.4 | Pi LLM abstraction (optional) | Unified multi-provider LLM API for finer-grained model control | impl-plan-v2 §3 | ✗ | ✓ |
| 10.5 | n8n pipeline tool (optional) | MCP-connected visual workflow for specific pipeline stages | impl-plan-v2 §3, prd-v2 §7.5 | ✗ | ✓ |
| 10.6 | OpenCode development tool | Build + plan agents for SPARC methodology during development | impl-plan-v2 §1 | N/A | N/A |
| 10.7 | MCP integration | Standardized tool protocol connecting Hermes to AgentDB, n8n, Pi, and other services | impl-plan-v2 §3 | ✓ | ✓ |
| 10.8 | Multi-model provider support | OpenRouter (200+ models), Nous Portal, OpenAI, Anthropic, Google, local | impl-plan-v2 §3 | ✓ | ✓ |
| 10.9 | agentskills.io skill format | Open standard for modular, reusable agent skills | impl-plan-v2 §3 | ✓ | ✓ |

---

## Summary Counts

| Category | MVP Features | Full Features | Total |
|---|---|---|---|
| 1. Knowledge & Storage | 8 | 4 | 12 |
| 2. Ingestion Pipeline | 3 | 9 | 12 |
| 3. Agent Architecture | 3 | 8 | 11 |
| 4. Expert Reasoning & Learning | 6 | 6 | 12 |
| 5. Report Generation & Delivery | 3 | 14 | 17 |
| 6. Evaluation | 1 | 7 | 8 |
| 7. Governance & Infrastructure | 0 | 9 | 9 |
| 8. Security & Compliance | 0 | 4 | 4 |
| 9. Monitoring & Observability | 1 | 3 | 4 |
| 10. Technology Integration | 5 | 4 | 9 |
| **Total** | **30** | **68** | **98** |
