# Agent Frosty — Architecture Discussions

**File Purpose**: Track key architectural decisions, trade-off analyses, and resolution rationales.

---

## Discussion 1: Workflow Engine — n8n vs Agentic Stack

**Date**: 2026-05-24  
**Participants**: Architect (opencode)  
**Status**: Resolved — Hybrid agentic stack selected over n8n-heavy approach

### Context

The original architecture assumed n8n as the workflow/pipeline engine for Agent Frosty. The implementation plan and PRD were drafted with n8n as the orchestration backbone. The user challenged this assumption and suggested evaluating four alternative tools:

1. [Paperclip](https://github.com/paperclipai/paperclip) — 67K stars
2. [Pi](https://github.com/earendil-works/pi) — 53.5K stars
3. [Hermes-Agent](https://github.com/nousresearch/hermes-agent) — 165K stars
4. [OpenCode](https://github.com/anomalyco/opencode) — 165K stars

### Initial Position (n8n-heavy)

n8n was assessed as the workflow engine with:

- **Strengths**: Built-in AI Agent node with 8 connection types, visual workflow builder, any node as an AI tool via `ai_tool` port, sub-workflow support, error handling patterns
- **Key Gaps**:
  - Controller meta-agent (dynamic planning) — n8n is DAG-only, can't dynamically branch/loop based on intermediate LLM output
  - Cyclic reasoning (re-query on insufficient results) — requires `SplitInBatches` hacks
  - AgentDB integration — no native AgentDB node
  - ReasoningBank trajectory tracking — no built-in learning loop
  - LangGraph-style state graphs — not supported in n8n's DAG model

**Recommendation then**: Split architecture — Controller Agent as external service calling n8n sub-workflows for each pipeline stage.

### Tool Analysis

#### Paperclip
- **What it is**: Open-source orchestration platform for managing teams of AI agents. Node.js server + React UI dashboard.
- **Key capabilities**: Org charts, goal alignment, budgets per agent, heartbeat-based activation, ticket system with full audit trail, multi-company support, governance (board-level approve/pause/terminate)
- **Core metaphor**: "If OpenClaw is an employee, Paperclip is the company"
- **Relevance to Agent Frosty**: Fills the governance/dashboard/management layer — budgets, org charts, mission alignment, audit

#### Pi
- **What it is**: AI agent toolkit with coding agent CLI, agent core runtime, and unified multi-provider LLM API.
- **Key packages**:
  - `@earendil-works/pi-coding-agent`: Interactive coding agent CLI
  - `@earendil-works/pi-agent-core`: Agent runtime with tool calling and state management
  - `@earendil-works/pi-ai`: Unified multi-provider LLM API (OpenAI, Anthropic, Google, etc.)
- **Relevance to Agent Frosty**: LLM abstraction layer and agent runtime plumbing

#### Hermes-Agent
- **What it is**: Self-improving AI agent by Nous Research with built-in learning loop, subagent spawning, MCP integration, cron, and multi-platform delivery.
- **Key capabilities**:
  - Built-in learning loop — creates skills from experience, improves them during use
  - Autonomous skill creation after complex tasks
  - Subagent spawning for parallel workstreams
  - 40+ tools, toolset system, MCP integration
  - FTS5 session search with LLM summarization for cross-session recall
  - Honcho dialectic user modeling
  - Cron scheduler with delivery to any platform
  - Trajectory generation and compression (research-ready)
  - Multi-platform gateway: Telegram, Discord, Slack, WhatsApp, Signal, CLI
  - Seven terminal backends: local, Docker, SSH, Singularity, Modal, Daytona, Vercel Sandbox
- **Relevance to Agent Frosty**: This is the **runtime backbone** — replaces ReasoningBank, Controller Agent, Swarm Orchestration, multi-platform delivery, and evaluation pipeline all in one

#### OpenCode
- **What it is**: Open source AI coding agent with build and plan modes.
- **Key capabilities**: Two built-in agents (build full-access, plan read-only), general subagent for complex tasks, CLI and desktop app
- **Relevance to Agent Frosty**: The development tool for building Agent Frosty itself (SPARC methodology, plan mode for analysis, build mode for implementation)

### Revised Position (Hybrid Agentic Stack)

After reviewing all four tools, the n8n-heavy recommendation was revised. The key insight: **Hermes-Agent already has most of what we were building from scratch.**

| What we were building | What Hermes already provides |
|---|---|
| ReasoningBank (trajectory tracking + learning) | Built-in learning loop: skill creation, trajectory compression, self-improvement |
| Controller Agent + Swarm Orchestration | Subagent spawning, parallel workstreams, MCP tool integration |
| Multi-platform report delivery | Telegram, Discord, Slack, WhatsApp, Signal, Email — built-in |
| Cron-based scheduled reports | Built-in cron scheduler with platform delivery |
| AgentDB memory layer | Cross-session FTS5 search + Honcho user modeling |
| Evaluation pipeline | Batch trajectory generation and compression |
| Agent specialization | Skills system (agentskills.io compatible) |

### Final Recommendation

```
Paperclip (governance, budgets, org chart, dashboard)
  └─ Hermes-Agent (runtime, learning loop, subagents, cron, delivery)
       ├─ Pi (LLM abstraction — optional, Hermes has this built-in)
       ├─ AgentDB (vector/knowledge store via MCP)
       ├─ n8n (pipeline stages as MCP-connected tools, not orchestrator)
       └─ OpenCode (building Agent Frosty itself via SPARC methodology)
```

**n8n's role shrinks** from orchestrator to one tool among many — called via MCP by Hermes-Agent for specific pipeline stages that benefit from visual workflow construction (e.g., document processing chains).

### Impact on Existing Documents

| Document | Impact |
|---|---|
| `implementation-plan.md` | Needs updating — replace Swarm Orchestration references with Hermes subagent spawning |
| `prd.md` | Needs updating — Epic 3 (Agent Swarm) and Epic 6 (Orchestration) architecture changes |

### Open Questions

1. **Hermes skill vs external microservice**: Should each specialist agent (Diagram Analyst, Research Analyst, Reasoning Agent) be a Hermes skill or an external service called via MCP?
2. **Paperclip integration depth**: How much Paperclip governance is needed for MVP? Can Paperclip be added later?
3. **Pi redundancy**: Hermes already has multi-provider LLM support via Nous Portal and OpenRouter. Is Pi needed at all, or is it extra abstraction?

---

## Discussion 2: AgentDB vs Hermes Built-in Memory

**Date**: 2026-05-24  
**Status**: Open — needs investigation

### Context

Hermes-Agent has built-in FTS5 session search, LLM summarization for cross-session recall, and Honcho user modeling. The original architecture relied on AgentDB for all knowledge storage.

### Consideration

Should AgentDB remain as the primary knowledge store (for domain ontology, documents, diagrams, formulas), with Hermes memory handling only session/agent-level context? Or should we consolidate onto Hermes' built-in memory?

### Tentative Decision

AgentDB remains for structured domain knowledge (high-performance vector search, hybrid search, quantization, versioned provenance). Hermes memory handles agent session context, user modeling, and conversation history. The two layers serve different purposes — AgentDB is the "library," Hermes memory is the "notebook."

---

## Discussion 3: MVP Scope with Hermes

**Date**: 2026-05-24  
**Status**: Open — needs scoping

### Context

The PRD defines MVP as: AgentDB multi-database + text ingestion + single Analyst Agent + 20 seed ontology entities + 20 test queries. With Hermes-Agent as the runtime, the MVP scope changes.

### Revised MVP Options

**Option A — Hermes-native MVP** (recommended):
- Install Hermes-Agent, configure with Nous Portal or OpenRouter
- Create Agent Frosty skills as Hermes skills (Diagram Analyst, Research Analyst, Reasoning Agent, Report Writer)
- Add AgentDB as MCP-connected tool
- MVP is: upload documents via Hermes CLI → Agent Frosty skill processes them → report output via CLI or Telegram
- Timeline: potentially faster since Hermes provides subagent spawning, memory, and delivery out of the box

**Option B — Traditional MVP** (original plan):
- Build custom Python/Node.js backend with AgentDB + custom agent implementation
- No Hermes dependency in MVP
- Add Hermes integration later for production delivery
- Timeline: slower but more portable

### Unresolved

Which MVP path provides faster time-to-value without compromising the architecture?

---

## Discussion 4: Foundation Model Strategy

**Date**: 2026-05-24  
**Status**: Open — needs domain clarification

### Context

The original agent-frosty.md identifies the need for vision models (GPT-4V, Claude Sonnet) for diagram parsing and reasoning models for report generation.

### Question

- Which model(s) for reasoning? GPT-4o vs Claude Sonnet vs local Llama/Mistral?
- Which for vision? GPT-4V vs Claude Sonnet structured output?
- Hermes supports model switching via `/model` — does this solve the multi-model problem?
- What about Nous Portal vs OpenRouter vs direct API keys?

### Dependency

Answer depends on the user's domain (civil engineering, aerospace, biotech, financial compliance — currently unknown).

---

## Decision Log

| ID | Decision | Rationale | Date |
|---|---|---|---|
| D001 | Hermes-Agent replaces ReasoningBank + Swarm Orchestration + Controller Agent | Hermes has all three built-in (learning loop, subagent spawning, MCP) | 2026-05-24 |
| D002 | Paperclip added for governance layer | Provides org charts, budgets, audit logs that the original architecture lacked | 2026-05-24 |
| D003 | n8n demoted to MCP tool | No longer the orchestrator, but useful for specific pipeline stages | 2026-05-24 |
| D004 | AgentDB retained for domain knowledge | Hermes memory handles session context; AgentDB handles structured domain knowledge | 2026-05-24 (tentative) |
| D005 | OpenCode used for development | Build + Plan agents for SPARC methodology | 2026-05-24 |
