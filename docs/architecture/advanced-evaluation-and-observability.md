# Advanced Evaluation & Observability

## Overview

Enterprise evaluation: runtime, skill, tool evaluation, tracing, quality scoring, failure trends, drift/regression signals.

## Entities

```sql
-- Runtime Evaluation
CREATE TABLE runtime_evaluation (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    runtime_id UUID REFERENCES runtime(id),
    session_id UUID,
    quality_score DECIMAL,
    latency_ms INTEGER,
    success BOOLEAN,
    error_detail TEXT,
    created_at TIMESTAMP
);

-- Skill Evaluation
CREATE TABLE skill_evaluation (
    id UUID PRIMARY KEY,
    skill_id UUID REFERENCES skill(id),
    test_case_id UUID,
    input_data JSONB,
    expected_output JSONB,
    actual_output JSONB,
    passed BOOLEAN,
    created_at TIMESTAMP
);

-- Trace Event
CREATE TABLE trace_event (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    trace_id UUID,
    span_name VARCHAR(100),
    parent_span_id UUID,
    attributes JSONB,
    created_at TIMESTAMP
);
```

## Related

- **Skill Lifecycle** (#8) - Skill evaluation
- **Tool Registry** (#7) - Tool evaluation