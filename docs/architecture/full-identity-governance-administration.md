# Full Identity Governance Administration

## Overview

Enterprise-grade IGA: access lifecycle, certification campaigns, SoD rules, privileged access governance, delegated authority windows, service and agent identity governance.

## Entities

### CertificationCampaign

```sql
CREATE TABLE certification_campaign (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    name VARCHAR(200),
    campaign_type VARCHAR(50), -- 'periodic', 'privileged', 'access'
    scope JSONB,
    schedule VARCHAR(50),
    status VARCHAR(20),
    created_at TIMESTAMP
);
```

### SoDRule

```sql
CREATE TABLE sod_rule (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    name VARCHAR(200),
    conflicting_roles JSONB,
    severity VARCHAR(20), -- 'high', 'medium', 'low'
    is_enabled BOOLEAN,
    created_at TIMESTAMP
);
```

### PrivilegedAccess

```sql
CREATE TABLE privileged_access (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    principal_type VARCHAR(50),
    principal_id UUID,
    access_right VARCHAR(100),
    grant_type VARCHAR(50), -- 'permanent', 'temporary', 'justified'
    effective_from TIMESTAMP,
    effective_to TIMESTAMP,
    created_at TIMESTAMP
);
```

## Related

- **Identity Governance Lite** (#16) - Base governance
- **Access Reviews** (#14) - Certifications