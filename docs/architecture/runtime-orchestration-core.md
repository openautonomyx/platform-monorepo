# Runtime Orchestration Core

## Overview

The Runtime Orchestration Core module manages the execution layer using LangGraph as the top-level orchestrator and OpenAI Agents SDK as the human-facing runtime. It defines runtime roles and separates channel, branch, and worker runtimes.

## Runtime Roles

### Orchestrator (LangGraph)

LangGraph serves as the top-level workflow orchestrator.

```
┌─────────────────────────────────────────────┐
│           LangGraph Orchestrator            │
├─────────────────────────────────────────────┤
│ - Workflow definition and state management │
│ - Task routing to runtimes                 │
│ - Multi-step execution coordination        │
│ - Branch/merge logic                      │
│ - Error handling and retry               │
└─────────────────────────────────────────────┘
```

### Human-Facing Runtime (OpenAI Agents SDK)

OpenAI Agents SDK handles direct human interactions.

```
┌─────────────────────────────────────────────┐
│       OpenAI Agents SDK (Human-Facing)       │
├─────────────────────────────────────────────┤
│ - Direct employee conversations            │
│ - Interactive task handling               │
│ - Context preservation                  │
│ - Natural language interface            │
└─────────────────────────────────────────────┘
```

### Worker Runtimes

Additional runtimes for specialized tasks.

| Runtime | Role | Tasks |
|---------|------|-------|
| Claude Agent SDK | Code worker | Code generation, debugging |
| Deep Agents | Research worker | Research, analysis |
| CrewAI | Crew worker | Multi-agent workflows |
| LangChain | Pipeline worker | Chain executions |

## Channel / Branch / Worker Split

### Channel Runtime

Each channel (chat, announcement, threaded, support) can have an assigned runtime.

```sql
CREATE TABLE channel_runtime (
    id UUID PRIMARY KEY,
    channel_id UUID REFERENCES channel(id),
    runtime_id UUID REFERENCES runtime(id),
    is_default BOOLEAN DEFAULT FALSE,
    config JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);
```

| Channel Type | Default Runtime |
|---------------|-----------------|
| chat | openai-agents-sdk |
| announcement | openai-agents-sdk |
| threaded | openai-agents-sdk |
| support | openai-agents-sdk |

### Branch Runtime

Branches (feature branches, task divisions) can route to different runtimes.

```sql
CREATE TABLE branch_runtime (
    id UUID PRIMARY KEY,
    branch_id UUID NOT NULL,
    parent_workflow_id UUID,
    runtime_id UUID REFERENCES runtime(id),
    execution_mode VARCHAR(20), -- 'sequential', 'parallel', 'hybrid'
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Worker Runtime

Workers execute specific task types.

```sql
CREATE TABLE worker_runtime (
    id UUID PRIMARY KEY,
    worker_type VARCHAR(50) NOT NULL, -- 'code', 'research', 'crew', 'chain'
    runtime_id UUID REFERENCES runtime(id),
    config JSONB,
    max_concurrent_tasks INTEGER DEFAULT 5,
    timeout_seconds INTEGER DEFAULT 300,
    created_at TIMESTAMP DEFAULT NOW()
);
```

## Runtime Selection Summary

```
Task Received
    ↓
LangGraph (Orchestrator)
    ↓
Determine: Human-facing or Worker?
    ├── Human-facing → OpenAI Agents SDK
    │       ↓
    │   Channel assignment
    │       ↓
    │   Employee interaction
    │
    └── Worker → Determine worker type
            ├── Code → Claude Agent SDK
            ├── Research → Deep Agents
            ├── Crew → CrewAI
            └── Chain → LangChain
    ↓
Execution complete → Return to orchestrator
```

## Startup vs Later Managed-Runtime Scope

### Startup Stage (Phase 1)

- LangGraph as single orchestrator
- OpenAI Agents SDK as only human-facing runtime
- Workers enabled via opt-in only
- No managed runtime environment
- Simple runtime configuration

| Runtime | Status | Config |
|---------|--------|--------|
| LangGraph | Required | Managed |
| OpenAI Agents SDK | Default | Configurable |
| Claude Agent SDK | Opt-in | Manual config |
| Deep Agents | Opt-in | Manual config |
| CrewAI | Opt-in | Manual config |
| LangChain | Opt-in | Manual config |

### Later Managed-Runtime Scope (Phase 2-3)

- Managed runtime environments
- Runtime performance monitoring
- Auto-scaling based on load
- Runtime cost tracking
- Custom runtime registration

### Managed Runtime Features

| Feature | Description |
|---------|-------------|
| Managed sessions | Pre-warmed runtime environments |
| Environment profiles | Pre-configured runtime configs |
| Usage tracking | Per-runtime cost and performance |
| Approval controls | Require approval for premium runtimes |

## Related Modules

- **Workflow & Control Plane Core** (#3) - Task/execution
- **Runtime Registry & Selection** (#6) - Runtime selection
- **Tool Registry & Basic Governance** (#7) - Tool compatibility
- **Managed Runtime Governance** (#31) - Managed runtimes
- **Memory & Cortex Core** (#5) - Runtime memory context