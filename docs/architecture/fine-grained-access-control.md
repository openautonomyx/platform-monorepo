# Fine-Grained Access Control

## Overview

Fine-grained access control extends basic RBAC with ABAC, entity-level and field-level access, delegated and temporary access.

## Extensions

### ABAC Conditions

```sql
CREATE TABLE access_condition (
    id UUID PRIMARY KEY,
    policy_id UUID,
    attribute VARCHAR(100), -- 'time', 'location', 'device', 'department'
    operator VARCHAR(20), -- 'equals', 'in', 'between'
    value JSONB
);
```

### Entity-Level Access

```sql
CREATE TABLE entity_acl (
    id UUID PRIMARY KEY,
    resource_type VARCHAR(50),
    resource_id UUID,
    principal_type VARCHAR(50),
    principal_id UUID,
    permission VARCHAR(50), -- 'read', 'write', 'delete', 'admin'
    granted_by UUID,
    created_at TIMESTAMP
);
```

### Field-Level Access

```sql
CREATE TABLE field_acl (
    id UUID PRIMARY KEY,
    resource_type VARCHAR(50),
    resource_id UUID,
    field_name VARCHAR(100),
    principal_type VARCHAR(50),
    principal_id UUID,
    permission VARCHAR(20), -- 'read', 'write'
    created_at TIMESTAMP
);
```

### Temporary Access

```sql
CREATE TABLE temporary_access (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    principal_type VARCHAR(50),
    principal_id UUID,
    resource_type VARCHAR(50),
    resource_id UUID,
    permission VARCHAR(50),
    effective_from TIMESTAMP,
    effective_to TIMESTAMP,
    justification TEXT,
    is_extended BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP
);
```

## Related Modules

- **Organization & Identity Core** (#1) - Base RBAC
- **Runtime Registry** (#6) - Runtime access
- **Access Reviews** (#14) - Access certification