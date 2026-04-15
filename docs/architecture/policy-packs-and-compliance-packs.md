# Policy Packs & Compliance Packs

## Overview

Preloaded compliance packs: DPA, SOC 2, internal control packs with evidence requirements, applicability rules, review cycles, exception handling.

## Entities

### CompliancePack

```sql
CREATE TABLE compliance_pack (
    id UUID PRIMARY KEY,
    pack_type VARCHAR(50), -- 'dpa', 'soc2', 'internal'
    name VARCHAR(200),
    version VARCHAR(20),
    is_enabled BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP
);
```

### ComplianceControl

```sql
CREATE TABLE compliance_control (
    id UUID PRIMARY KEY,
    pack_id UUID REFERENCES compliance_pack(id),
    control_id VARCHAR(50),
    title VARCHAR(200),
    description TEXT,
    evidence_requirements JSONB,
    created_at TIMESTAMP
);
```

## Related

- **Policy Packs** (#17) - Policy definitions
- **Audit & Evidence** (#28) - Evidence generation