# Agent Capability Management

## Overview

Manages agent capabilities: planning, reasoning, training, testing, checkpointing, event listeners, processes, crews/flows.

## Entities

### AgentCapability

```sql
CREATE TABLE agent_capability (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    agent_id UUID REFERENCES agent(id),
    capability_type VARCHAR(50), -- 'planning', 'reasoning', 'training', 'testing'
    name VARCHAR(100),
    config JSONB,
    is_enabled BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP
);
```

### AgentCheckpoint

```sql
CREATE TABLE agent_checkpoint (
    id UUID PRIMARY KEY,
    agent_id UUID REFERENCES agent(id),
    checkpoint_name VARCHAR(100),
    snapshot_data JSONB,
    created_at TIMESTAMP
);
```

### AgentEventListener

```sql
CREATE TABLE agent_event_listener (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    agent_id UUID REFERENCES agent(id),
    event_type VARCHAR(50),
    handler_config JSONB,
    created_at TIMESTAMP
);
```

## Related

- **Organization & Identity Core** (#1) - Agent definition
- **Runtime Registry** (#6) - Runtime compatibility