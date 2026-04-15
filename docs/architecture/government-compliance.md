# Government Compliance

## Overview

Public sector compliance: sovereignty rules, residency controls, restricted model/runtime/tool lists, security classification handling.

## Entities

### GovernmentStandard

```sql
CREATE TABLE government_standard (
    id UUID PRIMARY KEY,
    standard_name VARCHAR(100), -- 'FedRAMP', 'FISMA', 'ITAR'
    jurisdiction VARCHAR(50), -- 'federal', 'state', 'local'
    impact_level VARCHAR(20), -- 'low', 'moderate', 'high'
    is_enabled BOOLEAN,
    created_at TIMESTAMP
);
```

### ResidencyRule

```sql
CREATE TABLE residency_rule (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    data_category VARCHAR(50),
    required_region VARCHAR(50),
    storage_requirement VARCHAR(50), -- 'domestic', 'specific_location'
    created_at TIMESTAMP
);
```

## Related

- **Industry Compliance** (#25) - Commercial frameworks
- **Guardrails** (#24) - Privacy controls