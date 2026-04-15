# Context Window & Compaction Governance

## Overview

Treat context handling as governance: context budget policy, compaction triggers, briefing generation, checkpoint policy, memory write policy, context risk controls.

## Entities

### ContextBudgetPolicy

```sql
CREATE TABLE context_budget_policy (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    max_tokens INTEGER,
    max_messages INTEGER,
    compaction_trigger_percent INTEGER, -- trigger at X% of budget
    policy_type VARCHAR(50), -- 'strict', 'flexible'
    created_at TIMESTAMP
);
```

### CompactionRule

```sql
CREATE TABLE compaction_rule (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    compaction_type VARCHAR(50), -- 'summarize', 'drop', 'archive'
    priority_order INTEGER,
    conditions JSONB,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP
);
```

### MemoryWritePolicy

```sql
CREATE TABLE memory_write_policy (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    memory_type VARCHAR(50), -- 'semantic', 'episodic', 'working'
    allow_pii BOOLEAN DEFAULT FALSE,
    require_verification BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP
);
```

## Related

- **Memory & Cortex Core** (#5) - Memory storage
- **Guardrails** (#24) - Privacy controls