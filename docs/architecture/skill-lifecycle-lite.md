# Skill Lifecycle Lite

## Overview

The Skill Lifecycle Lite module manages skills as first-class assets with basic versioning, evaluation, and lifecycle states. Evaluation remains lightweight for startup.

## Schema

```sql
-- Skill Registry
CREATE TABLE skill (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    name VARCHAR(100) NOT NULL,
    description TEXT,
    version VARCHAR(20) NOT NULL,
    status VARCHAR(20) NOT NULL, -- 'draft', 'published', 'deprecated', 'archived'
    evaluation_score DECIMAL, -- 0-100
    runtime_compatibility JSONB,
    skill_type VARCHAR(50), -- 'prompt', 'plugin', 'chain', 'agent'
    config JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);

-- Skill Versioning
CREATE TABLE skill_version (
    id UUID PRIMARY KEY,
    skill_id UUID REFERENCES skill(id),
    version VARCHAR(20) NOT NULL,
    changelog TEXT,
    is_active BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Skill Usage
CREATE TABLE skill_usage (
    id UUID PRIMARY KEY,
    skill_id UUID REFERENCES skill(id),
    employee_id UUID REFERENCES employee(id),
    session_id UUID,
    feedback_score INTEGER,
    feedback_note TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

## Lifecycle States

```
[Draft] → [Published] → [Deprecated] → [Archived]
    ↓              ↓            ↓
 [Edit]      [Use]        [Migrate]    [View Only]
```

| State | Description | Allowed Actions |
|-------|-------------|----------------|
| draft | In development | read, update, delete |
| published | Active in use | read, update, deprecate |
| deprecated | Migrate away | read, migrate |
| archived | Retired | read only |

## Basic Evaluation Model

| Metric | Description |
|--------|------------|
| usage_count | Times executed |
| success_rate | % successful |
| avg_duration | Execution time |
| user_rating | 1-5 score |

## Startup Scope

- Basic version tracking
- Simple usage counts
- Manual publishing
- No approval workflow

## Future Enterprise Path

- Formal skill approval
- Rollout rings (alpha/beta/stable)
- Automated testing
- Usage analytics

## Related Modules

- **Tool Registry** (#7) - Tool vs skill distinction
- **Runtime Registry** (#6) - Runtime compatibility
- **Advanced Evaluation** (#21) - Enterprise evaluation