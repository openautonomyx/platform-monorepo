# Policy Packs & SLA Governance

## Overview

Policy packs manage HR, IT, finance, organization process packs, and SLA policies with breach detection.

## Entities

### PolicyPack

```sql
CREATE TABLE policy_pack (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    name VARCHAR(200),
    category VARCHAR(50), -- 'hr', 'it', 'finance', 'org', 'sla'
    version VARCHAR(20),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP
);
```

### Policy

```sql
CREATE TABLE policy (
    id UUID PRIMARY KEY,
    pack_id UUID REFERENCES policy_pack(id),
    name VARCHAR(200),
    description TEXT,
    policy_type VARCHAR(50), -- 'hr_policy', 'it_policy', 'finance_policy', 'sla_policy'
    rules JSONB,
    effective_from DATE,
    effective_to DATE,
    created_at TIMESTAMP
);
```

### SLAPolicy

```sql
CREATE TABLE sla_policy (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    name VARCHAR(200),
    target_type VARCHAR(50), -- 'task', 'approval', 'escalation'
    target_id UUID,
    sla_hours INTEGER,
    breach_action VARCHAR(50), -- 'notify', 'escalate', 'auto_close'
    created_at TIMESTAMP
);
```

### SLABreach

```sql
CREATE TABLE sla_breach (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    sla_policy_id UUID REFERENCES sla_policy(id),
    entity_type VARCHAR(50),
    entity_id UUID,
    breach_at TIMESTAMP,
    resolved_at TIMESTAMP,
    created_at TIMESTAMP
);
```

## Startup Scope

- Basic policies (simple JSON rules)
- Simple SLA (time-based)
- Manual breach handling

## Future: Enterprise Compliance Packs

- Pre-built compliance packs (DPA, SOC 2)
- Automated enforcement
- Complex rules (OPA)

## Related

- **Workflow & Control Plane Core** (#3) - Deadlines, escalations
- **Event Calendar** (#22) - Time governance