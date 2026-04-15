# Continuous Governance & Improvement

## Overview

Recurring governance: recurring checks, drift detection, regression detection, continuous compliance, continuous access review, continuous SLA monitoring.

## Entities

```sql
-- Governance Check
CREATE TABLE governance_check (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    check_type VARCHAR(50), -- 'drift', 'regression', 'compliance'
    entity_type VARCHAR(50),
    entity_id UUID,
    status VARCHAR(20), -- 'passed', 'warning', 'failed'
    details JSONB,
    created_at TIMESTAMP
);

-- Improvement Recommendation
CREATE TABLE improvement_recommendation (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    source_type VARCHAR(50), -- 'check', 'feedback', 'incident'
    recommendation TEXT,
    priority VARCHAR(20),
    status VARCHAR(20), -- 'pending', 'accepted', 'rejected'
    created_at TIMESTAMP
);
```

## Related

- **Access Reviews** (#14) - Continuous review
- **Event Calendar** (#22) - SLA monitoring