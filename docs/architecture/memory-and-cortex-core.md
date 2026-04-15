# Memory & Cortex Core

## Overview

The Memory & Cortex Core module manages agent memory across working, episodic, and semantic categories. It provides structured memory storage with compaction and briefing generation.

## Storage Architecture

### Redis - Working Memory

Fast, ephemeral storage for active conversation context.

```
working_memory
├── key: session_id:{tenant}:{employee}:{agent}
├── value: JSON array of messages
├── TTL: session timeout (configurable)
└── max_items: 50 (configurable)
```

### PostgreSQL - Episodic Memory

Persistent storage for conversation history and events.

```sql
CREATE TABLE episodic_memory (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL REFERENCES tenant(id),
    session_id UUID NOT NULL,
    employee_id UUID REFERENCES employee(id),
    agent_id UUID REFERENCES agent(id),
    memory_type VARCHAR(50) NOT NULL, -- 'conversation', 'action', 'tool_use', 'decision'
    content JSONB NOT NULL,
    importance_score INTEGER DEFAULT 0, -- 0-100
    embedding VECTOR(1536), -- for similarity search
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP
);

CREATE INDEX idx_episodic_session ON episodic_memory(session_id);
CREATE INDEX idx_episodic_employee ON episodic_memory(employee_id);
CREATE INDEX idx_episodic_importance ON episodic_memory(importance_score);
```

### SingleStore - Semantic Memory

Analytical storage for summaries, facts, and learned knowledge.

```sql
CREATE TABLE semantic_memory (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL,
    entity_type VARCHAR(100) NOT NULL, -- 'employee', 'project', 'policy', 'fact'
    entity_id UUID NOT NULL,
    memory_content TEXT NOT NULL,
    memory_embedding VECTOR(1536),
    confidence_score INTEGER DEFAULT 0, -- 0-100
    source_session_id UUID,
    source_type VARCHAR(50), -- 'compaction', 'explicit', 'inference'
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP,
    expires_at TIMESTAMP
);

CREATE INDEX idx_semantic_entity ON semantic_memory(entity_type, entity_id);
```

### Cortex Briefings

Generated summaries for agent context.

```sql
CREATE TABLE cortex_briefing (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL,
    employee_id UUID REFERENCES employee(id),
    agent_id UUID REFERENCES agent(id),
    briefing_type VARCHAR(50) NOT NULL, -- 'daily_summary', 'project_update', 'interaction_history'
    content TEXT NOT NULL,
    context_json JSONB,
    generated_at TIMESTAMP DEFAULT NOW(),
    valid_until TIMESTAMP,
    is_read BOOLEAN DEFAULT FALSE
);
```

## Memory Categories

### 1. Working Memory

Active conversation context, kept in Redis.

| Property | Value |
|----------|-------|
| Storage | Redis |
| Persistence | Session-only |
| TTL | 1 hour default |
| Max items | 50 messages |
| Access | O(1) read/write |

### 2. Episodic Memory

Historical events and conversation logs, kept in PostgreSQL.

| Property | Value |
|----------|-------|
| Storage | PostgreSQL |
| Persistence | 90 days default |
| Indexing | session, employee, importance |
| Search | Full-text + similarity |

### 3. Semantic Memory

Structured facts and learned knowledge, kept in SingleStore.

| Property | Value |
|----------|-------|
| Storage | SingleStore |
| Persistence | Permanent until expired |
| Indexing | entity type + ID |
| Search | Vector similarity |

## Compaction Policy

### Compaction Triggers

1. **Working Memory Overflow**
   - When working memory exceeds 50 items
   - Older items compacted to episodic memory

2. **Session End**
   - Full session compaction to episodic memory
   - Generate summary for semantic memory

3. **Scheduled Compaction**
   - Daily at configured time
   - Aggregate episodic → semantic memory

### Compaction Checkpoints

```python
class CompactionCheckpoint:
    id: UUID
    tenant_id: UUID
    checkpoint_type: enum('session', 'scheduled', 'manual')
    items_compacted: integer
    memory_before: JSON
    memory_after: JSON
    created_at: timestamp
```

### Compaction Rules

| Priority | Memory Type | Keep | Archive | Discard |
|----------|-------------|------|---------|---------|
| High importance | Episodic | 90 days | Permanent | After 1 year |
| Medium importance | Episodic | 30 days | 90 days | After 180 days |
| Low importance | Episodic | 7 days | 30 days | After 90 days |
| Facts/Entities | Semantic | Permanent | N/A | On expiration |

## Startup Scope vs Enterprise Governance

### Startup Scope (Phase 1)

- Simple Redis working memory
- Basic PostgreSQL episodic storage
- Manual compaction triggers
- Simple briefings (daily summary)

| Feature | Implementation |
|---------|----------------|
| Working memory | Redis, simple list |
| Episodic storage | PostgreSQL, basic tables |
| Semantic storage | SingleStore, basic |
| Compaction | Manual trigger |
| Briefings | Simple daily |

### Enterprise Governance Scope (Future Phase 2-3)

- Vector similarity search
- Importance scoring auto-classification
- Scheduled automatic compaction
- Rich briefings by type
- Cross-session memory linking
- Memory access audit

## Memory Write Policy

### Allowed Writes

1. **Working Memory**: Any during active session
2. **Episodic Memory**: Auto-save on session end, tool use, decisions
3. **Semantic Memory**: Verified facts only, approved by policy

### Restricted Writes

1. **PII/Sensitive Data**: Blocked by policy
2. **Credentials/Secrets**: Blocked always
3. **External Content**: Flagged for review

## Related Modules

- **Organization & Identity Core** (#1) - Employee/agent identity
- **Runtime Orchestration Core** (#4) - Runtime memory context
- **Context Window & Compaction Governance** (#32) - Context governance
- **Guardrails/Privacy/Compliance** (#24) - Privacy controls
- **Data Governance** (#15) - Data classification