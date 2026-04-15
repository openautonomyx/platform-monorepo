# Identity Governance Lite

## Overview

Identity governance lite adds lightweight governance before full IGA. Includes joiner/mover/leaver, temporary access, delegated access, access exceptions, and privileged access requests.

## Entities

### IdentityLifecycle

```sql
CREATE TABLE identity_lifecycle (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    employee_id UUID REFERENCES employee(id),
    lifecycle_type VARCHAR(20), -- 'joiner', 'mover', 'leaver'
    status VARCHAR(20), -- 'pending', 'in_progress', 'completed'
    triggered_by UUID,
    effective_date DATE,
    completed_at TIMESTAMP,
    created_at TIMESTAMP
);
```

### TemporaryAccess

```sql
CREATE TABLE temporary_access_request (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    employee_id UUID REFERENCES employee(id),
    requested_resource VARCHAR(100),
    justification TEXT,
    duration_days INTEGER,
    status VARCHAR(20), -- 'pending', 'approved', 'rejected'
    approver_id UUID,
    created_at TIMESTAMP
);
```

### AccessException

```sql
CREATE TABLE access_exception (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    employee_id UUID REFERENCES employee(id),
    policy_id UUID,
    justification TEXT,
    effective_from DATE,
    effective_to DATE,
    status VARCHAR(20),
    created_at TIMESTAMP
);
```

## Future: Full IGA

- Full SoD rules
- Certification campaigns
- Privileged access governance

## Related

- **Organization & Identity Core** (#1) - Employee
- **Access Reviews** (#14) - Certifications