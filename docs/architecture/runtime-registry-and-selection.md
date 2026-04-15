# Runtime Registry & Selection

## Overview

The Runtime Registry & Selection module manages available runtimes, their configurations, and selection logic for task execution. LangGraph remains the orchestrator, OpenAI Agents SDK is the default human-facing runtime.

## Runtime Types

### Available Runtimes

| Runtime | Type | Use Case |
|---------|------|---------|
| langgraph | orchestrator | Workflow orchestration |
| openai-agents-sdk | human-facing | Human interactions |
| claude-agent-sdk | worker | Claude Code tasks |
| deep-agents | worker | Deep research tasks |
| crewai | worker | Crew workflows |
| langchain | worker | LangChain pipelines |

### Runtime Registry Schema

```sql
CREATE TABLE runtime (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL REFERENCES tenant(id),
    name VARCHAR(100) NOT NULL,
    runtime_type VARCHAR(50) NOT NULL, -- 'orchestrator', 'human_facing', 'worker'
    provider VARCHAR(50) NOT NULL, -- 'langgraph', 'openai', 'claude', 'deep', 'crewai', 'langchain'
    version VARCHAR(20),
    config JSONB, -- provider-specific config
    is_enabled BOOLEAN DEFAULT TRUE,
    is_default BOOLEAN DEFAULT FALSE,
    cost_tier VARCHAR(20), -- 'free', 'standard', 'premium'
    capabilities JSONB, -- array of capability tags
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);
```

## Selection Logic

### Default Selection Rules

```
1. Task type → Preferred runtime from mapping
2. Check runtime enabled → Use if yes
3. Check runtime allowed → Use if yes
4. Fallback to tenant default runtime
5. Fallback to platform default (openai-agents-sdk for human-facing)
```

### Runtime Selection Mapping

```sql
CREATE TABLE runtime_selection (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    task_type VARCHAR(50) NOT NULL,
    preferred_runtime_id UUID REFERENCES runtime(id),
    allowed_runtime_ids JSONB, -- array of runtime IDs
    fallback_runtime_id UUID REFERENCES runtime(id),
    priority_order JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);
```

| Task Type | Preferred Runtime | Allowed Runtimes |
|----------|------------------|-----------------|
| simple_conversation | openai-agents-sdk | [openai-agents-sdk, claude-agent-sdk] |
| complex_reasoning | openai-agents-sdk | [openai-agents-sdk, deep-agents] |
| research | deep-agents | [deep-agents] |
| code_generation | claude-agent-sdk | [claude-agent-sdk, openai-agents-sdk] |
| crew_workflow | crewai | [crewai] |
| chain_execution | langchain | [langchain] |

### Tenant Override Hook

```sql
CREATE TABLE runtime_tenant_override (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    runtime_id UUID REFERENCES runtime(id),
    override_type VARCHAR(50), -- 'preferred', 'allowed', 'disabled'
    task_type VARCHAR(50) NULL,
    effective_from TIMESTAMP,
    effective_to TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);
```

## Fallback Logic

### Runtime Fallback Chain

```
Primary Runtime (down/unavailable)
    ↓
Allowed Runtime 1 → Allowed Runtime 2 → ... → Platform Default
    ↓
    If all fail → Queue task for retry (max 3 attempts)
```

### Failure Recovery

| Scenario | Action |
|----------|--------|
| Runtime unavailable | Fall to next allowed |
| Token limit exceeded | Try lighter runtime |
| Timeout | Fall to standard tier |
| All failed | Create incident, alert ops |

## Startup Defaults

### Default Runtimes at Startup

| Runtime | Role | Enabled |
|--------|------|---------|
| langgraph | orchestrator | Yes (required) |
| openai-agents-sdk | human-facing | Yes (default) |
| claude-agent-sdk | worker | No (opt-in) |
| deep-agents | worker | No (opt-in) |
| crewai | worker | No (opt-in) |
| langchain | worker | No (opt-in) |

### Platform Configuration

```json
{
  "platform": {
    "default_orchestrator": "langgraph",
    "default_human_facing": "openai-agents-sdk"
  },
  "startup": {
    "runtimes_enabled": ["langgraph", "openai-agents-sdk"],
    "worker_opt_in": true
  }
}
```

## Enterprise Override Path

### Phase 2-3 Enterprise Features

- Custom runtime registration
- Runtime cost controls
- Runtime performance tracking
- Auto-scaling policies
- Multi-runtime fallback optimization

### Enterprise Override Rules

1. Tenant can override preferred runtime per task type
2. Tenant can disable specific runtimes
3. Tenant can add custom runtimes (with approval)
4. Platform can enforce specific runtimes for compliance

## Related Modules

- **Runtime Orchestration Core** (#4) - Runtime execution
- **Workflow & Control Plane Core** (#3) - Execution requests
- **Tool Registry & Basic Governance** (#7) - Tool runtime compatibility
- **Advanced Evaluation & Observability** (#21) - Runtime evaluation
- **Managed Runtime Governance** (#31) - Managed runtimes