# Agent Frosty — Activity Log

> Tracks all iterations, changes, decisions, and feature additions throughout the project.

---

## Iteration 1: Initial Architecture & Document Foundation

**Date**: 2026-05-24

### What Was Done

1. **Reviewed original concept** (`agent-frosty.md`) — a user request to build an expert AI agent system using GraphRAG (Graphiti), multimodal knowledge ingestion, and agent orchestration.

2. **Created `implementation-plan.md`** — first implementation plan using n8n-heavy architecture:
   - AgentDB multi-database for knowledge storage
   - Swarm Orchestration for multi-agent coordination
   - ReasoningBank for learning/trajectory tracking
   - SPARC methodology for development process
   - LangGraph + FastAPI for agent runtime

3. **Created `prd.md`** — comprehensive product requirements with:
   - 8 Epics: Knowledge Foundation, Ingestion Pipeline, Agent Swarm, Expert Reasoning, Evaluation Pipeline, Orchestration & API, Learning & Improvement Loop, Production Readiness
   - ~30 Stories with user-facing acceptance criteria
   - ~100+ Tasks with estimated effort
   - 3 Release milestones (MVP Weeks 1-2, Release 2 Weeks 3-4, Release 3 Week 5+)

### Key Decisions Made
- AgentDB chosen over Graphiti for unified vector + graph + reasoning stack
- No fine-tuning first; expertise constructed via prompt architecture + graph constraints
- Controller meta-agent for dynamic workflow planning
- n8n as primary orchestration engine

---

## Iteration 2: Architecture Pivot — Hybrid Agentic Stack

**Date**: 2026-05-24

### Trigger

User challenged the n8n-heavy approach and suggested evaluating four alternative tools:
- Paperclip
- Pi
- Hermes-Agent
- OpenCode

### What Was Done

1. **Researched all four tools** — fetched READMEs, analyzed capabilities, mapped to Agent Frosty architecture needs.

2. **Created `discussions.md`** — full trade-off analysis documenting:
   - Initial n8n-heavy position and its key gaps
   - Detailed tool profiles for Paperclip, Pi, Hermes-Agent, OpenCode
   - Mapping: which Hermes features replace what was being built from scratch
   - Final recommendation: Hybrid agentic stack

3. **Revised architecture decision**: n8n demoted from orchestrator to MCP-connected pipeline tool. Hermes-Agent becomes the runtime backbone.

### Key Decisions Made
| ID | Decision | Rationale |
|---|---|---|
| D001 | Hermes-Agent replaces ReasoningBank + Swarm Orchestration + Controller Agent | Hermes has all three built-in (learning loop, subagent spawning, MCP) |
| D002 | Paperclip added for governance layer | Provides org charts, budgets, audit logs |
| D003 | n8n demoted to MCP tool | No longer the orchestrator |
| D004 | AgentDB retained for domain knowledge | Hermes memory handles session context; AgentDB handles structured domain knowledge |
| D005 | OpenCode used for development | Build + Plan agents for SPARC methodology |

---

## Iteration 3: v2 Documents & README

**Date**: 2026-05-24

### What Was Done

1. **Created `implementation-plan-v2.md`** — complete rewrite reflecting the hybrid agentic stack:
   - Four-layer architecture: Paperclip → Hermes → AgentDB → Foundation
   - Phase 3: Swarm Orchestration replaced with Hermes subagent spawning
   - Phase 4: ReasoningBank replaced with Hermes built-in learning loop
   - Phase 5: Multi-platform delivery via Hermes gateway
   - Technology stack completely updated
   - Roadmap reordered: Hermes-first (install Hermes Week 1)

2. **Created `prd-v2.md`** — product requirements restructured:
   - New Epic 3: Hermes-Agent Platform Setup
   - Old Epic 3 → new Epic 4: Skill-Based Specialist Agents
   - Old Epic 4 merged into Epic 5 (simplified — Hermes learning is built-in)
   - Old Epic 6 → new Epic 7: Orchestration & Governance
   - Release plan updated: MVP delivers Hermes + Analyst skill

3. **Created `README.md`** — comprehensive repository README with:
   - Project vision and four-layer architecture diagram
   - Feature table with 10 capabilities
   - Full technology stack with links to all tools
   - Quick start (5 steps)
   - Agent skills table with invocation examples
   - Project structure, MVP roadmap, key design decisions

### Files Preserved
- Original `implementation-plan.md` kept as-is
- Original `prd.md` kept as-is

---

## Iteration 4: Feature List

**Date**: 2026-05-24

### What Was Done

1. **Created `feature-list.md`** — comprehensive feature list synthesized from all planning documents:
   - 10 categories originally, expanded to 11
   - 98 total features (30 MVP, 68 Full)
   - Each feature tagged with source document(s), MVP/Full scope
   - Coverage across: Knowledge & Storage, Ingestion Pipeline, Agent Architecture, Expert Reasoning & Learning, Report Generation & Delivery, Evaluation, Governance & Infrastructure, Security & Compliance, Monitoring & Observability, Technology Integration, Image Catalogue

### Source Documents Used
- `agent-frosty.md` — original concept and architecture review
- `implementation-plan-v2.md` — revised implementation plan
- `prd-v2.md` — revised product requirements

---

## Iteration 5: Activity Log & Image Features

**Date**: 2026-05-24

### What Was Done

1. **Created `activity.md`** (this file) — iteration tracking and change log.

2. **Updated `feature-list.md`** — added image catalogue features (category 11).

3. **Created `requirements.md`** — comprehensive requirements document covering all 11 categories with detailed specifications for each feature.

### New Features Added

- **Image Catalogue & Description** — specialist Image Description Agent using vision LLM, user text/voice updates, graph-based storage with metadata
- **Image Generation** — generate new images based on context, metadata, descriptions, and similarity to existing images
- **UI Features** — image grid/browser, metadata editor, comparison view, bulk operations, search/filter, voice input, annotation tools, image generation parameters
- **Additional features**: image tagging/taxonomy, versioning, provenance tracking, relationship mapping, bulk import, deduplication, export

---

## Iteration 6: Comprehensive Requirements & Image Catalogue Expansion

**Date**: 2026-05-24

### What Was Done

1. **Updated `feature-list.md`** — added full Image Catalogue category (category 11) with 22 features (7 MVP, 15 Full):
   - Vision LLM auto-description, user text/voice description updates, graph-based storage
   - Image metadata extraction, similarity search, tagging/taxonomy
   - Context-aware image generation via diffusion models
   - Versioning, provenance tracking, relationship mapping
   - Bulk import, deduplication, export
   - Full UI suite: grid/browser, metadata editor, comparison view, bulk operations, search/filter, voice input, annotation tools, generation parameters

2. **Updated `feature-list.md` summary counts** — totals updated from 98 features (30 MVP, 68 Full) to 120 features (37 MVP, 83 Full)

3. **Created `requirements.md`** — comprehensive requirements document covering all 11 feature categories:
   - Each feature includes: description, priority, dependencies, user stories, acceptance criteria
   - 158 total requirements specifications across Knowledge & Storage, Ingestion Pipeline, Agent Architecture, Expert Reasoning & Learning, Report Generation & Delivery, Evaluation, Governance & Infrastructure, Security & Compliance, Monitoring & Observability, Technology Integration, and Image Catalogue

### Key Decisions Made
- Image Catalogue treated as full feature category (not sub-category) due to scope
- Graph-based storage for images (not relational) to leverage existing AgentDB infrastructure
- MVP image features focus on core workflow: import → auto-describe → search → manual edit

---

## Iteration 7: Quotation & Proposal Management Feature

**Date**: 2026-05-24

### What Was Done

1. **Updated all four planning documents** with new Quotation & Proposal Management feature (category 12):

2. **`feature-list.md`** — added category 12 with 14 features (8 MVP, 6 Full):
   - Multi-channel inbound requests (email, Slack, Discord, Telegram)
   - Request parsing & intent extraction
   - Knowledge-backed quote generation via Hermes + AgentDB
   - Quote/proposal templates, approval workflow
   - Email delivery to customer, multi-channel user notifications
   - Quote status lifecycle, versioning, analytics dashboard
   - Customer communication history, n8n workflow templates
   - Paperclip governance integration, CRM sync (optional)
   - Summary counts updated: 120 → 134 total features (45 MVP, 89 Full)

3. **`requirements.md`** — added category 12 with 14 detailed feature specs:
   - Each feature includes: description, priority, dependencies, user stories, acceptance criteria
   - Full coverage of: inbound channels, NLP parsing, pricing retrieval, generation, templates, approval routing, email delivery, notifications, status lifecycle, versioning, analytics, n8n workflows, Paperclip governance, CRM sync

4. **`prd-v2.md`** — added Epic 10 with 6 stories and 25 tasks:
   - Story 10.1: Multi-channel inbound request intake (email, Slack, Discord, Telegram)
   - Story 10.2: Request parsing & quote generation (NLP, pricing retrieval, templates)
   - Story 10.3: Approval, delivery & notifications (approval engine, email, user notifications)
   - Story 10.4: n8n workflow orchestration (workflow templates, Hermes/Paperclip integration)
   - Story 10.5: Quote analytics & CRM integration (analytics, versioning, comms history, CRM sync)
   - Story 10.6: Paperclip governance for quotes (budget limits, approval chains, audit)
   - Release plan updated: Quote MVP added to Release 2, full quote features to Release 3
   - Scope table updated with Quotation row

5. **`implementation-plan-v2.md`** — added Phase 8 with:
   - Full architecture diagram for the quotation system
   - Key components: n8n workflows, Hermes quote agent, AgentDB quote ontology, Paperclip governance
   - Integration points table (n8n ↔ Hermes, AgentDB, Paperclip)
   - Reporting & analytics section
   - Technology stack updated with quote entries
   - Roadmap updated: Weeks 4-5 include quote intake, n8n workflow, Hermes quote skill, approval engine, and Paperclip governance
   - Added 3 new design decisions (D011-D013)

### Key Decisions Made
- **n8n for quotation workflows**: n8n handles multi-channel inbound/outbound, conditional routing, and API integrations visually — ideal for process workflows
- **Hermes for quote intelligence**: Hermes handles cognitive parts (parsing, knowledge retrieval, generation); n8n handles procedural parts (listening, routing, delivering)
- **Quote ontology in AgentDB**: Customers, pricing, and quote history stored in the same knowledge graph as domain knowledge for cross-domain reasoning
- Quote MVP (Release 2) includes: email + Slack intake, basic generation, approval, email delivery, notifications
- Quote Full (Release 3) adds: Paperclip governance, analytics, versioning, CRM sync

---

## Document Inventory

| File | Version | Status | Description |
|---|---|---|---|
| `agent-frosty.md` | Original | Complete | Original concept and architecture review |
| `implementation-plan.md` | v1 | Superseded | n8n-heavy architecture plan |
| `implementation-plan-v2.md` | v2 | Current | Hermes/Paperclip hybrid stack plan |
| `prd.md` | v1 | Superseded | n8n-heavy product requirements |
| `prd-v2.md` | v2 | Current | Hermes/Paperclip product requirements |
| `discussions.md` | — | Current | Architecture decisions and trade-off analysis |
| `README.md` | — | Current | Repository README |
| `feature-list.md` | — | Current | Comprehensive feature list (134 features, 45 MVP / 89 Full) |
| `requirements.md` | — | Current | Detailed requirements with user stories and acceptance criteria for all 12 categories |
| `activity.md` | — | Current | Iteration tracking (this file) |
