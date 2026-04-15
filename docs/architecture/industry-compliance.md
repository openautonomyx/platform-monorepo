# Industry Compliance

## Overview

Sector-specific compliance: industry frameworks, control mappings, data handling obligations, audit evidence, required approvals, restricted runtimes/tools.

## Entities

### IndustryFramework

```sql
CREATE TABLE industry_framework (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    framework_name VARCHAR(100), -- 'HIPAA', 'PCI-DSS', 'GDPR', 'SOX'
    industry VARCHAR(50),
    version VARCHAR(20),
    is_enabled BOOLEAN,
    created_at TIMESTAMP
);
```

### ControlMapping

```sql
CREATE TABLE control_mapping (
    id UUID PRIMARY KEY,
    framework_id UUID REFERENCES industry_framework(id),
    control_id VARCHAR(50),
    description TEXT,
    evidence_type VARCHAR(50),
    implemented_by VARCHAR(100), -- module/feature
    created_at TIMESTAMP
);
```

## Related

- **Policy Packs** (#17) - Compliance packs
- **Guardrails** (#24) - Compliance enforcement