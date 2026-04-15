# Data Governance

## Overview

Data governance manages classification, stewardship, lineage, retention, deletion, quality, and access policies.

## Entities

### DataClassification

```sql
CREATE TABLE data_classification (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    name VARCHAR(100), -- 'public', 'internal', 'confidential', 'restricted'
    description TEXT,
    color_code VARCHAR(20),
    created_at TIMESTAMP
);
```

### DataSteward

```sql
CREATE TABLE data_steward (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    employee_id UUID REFERENCES employee(id),
    data_category VARCHAR(50),
    created_at TIMESTAMP
);
```

### DataLineage

```sql
CREATE TABLE data_lineage (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    source_type VARCHAR(50),
    source_id UUID,
    target_type VARCHAR(50),
    target_id UUID,
    transform_type VARCHAR(50),
    created_at TIMESTAMP
);
```

### DataRetention

```sql
CREATE TABLE data_retention_policy (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    data_type VARCHAR(50),
    retention_days INTEGER,
    archive_after_days INTEGER,
    delete_after_days INTEGER,
    created_at TIMESTAMP
);
```

## Related Modules

- **Basic Audit & Activity Log** (#12) - Data access logs
- **Guardrails/Privacy/Compliance** (#24) - PII handling