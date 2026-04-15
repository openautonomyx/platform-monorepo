# Policy Governance with OPA

## Overview

Centralized policy evaluation using OPA (Open Policy Agent): policy registry, authoring, versioning, decision history, replay/testing, tenant overrides.

## Entities

### PolicyRego

```sql
CREATE TABLE policy_rego (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    policy_name VARCHAR(100),
    rego_code TEXT,
    version VARCHAR(20),
    status VARCHAR(20), -- 'draft', 'active', 'deprecated'
    is_system BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP
);
```

### OPA Decision

```sql
CREATE TABLE opa_decision (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    policy_id UUID REFERENCES policy_rego(id),
    input_json JSONB,
    result_json JSONB,
    decided_at TIMESTAMP
);
```

## Related

- **Policy Packs** (#17) - Policy definitions
- **Guardrails** (#24) - Runtime enforcement