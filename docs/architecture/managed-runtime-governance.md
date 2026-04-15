# Managed Runtime Governance

## Overview

Managed runtimes: managed runtime registry, managed session lifecycle, hosted environment profiles, runtime policy controls, usage tracking, approval controls.

## Entities

### ManagedRuntime

```sql
CREATE TABLE managed_runtime (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    runtime_type VARCHAR(50),
    environment_profile VARCHAR(50), -- 'development', 'staging', 'production'
    instance_count INTEGER,
    max_sessions INTEGER,
    cost_tier VARCHAR(20),
    is_enabled BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP
);
```

### ManagedSession

```sql
CREATE TABLE managed_session (
    id UUID PRIMARY KEY,
    managed_runtime_id UUID REFERENCES managed_runtime(id),
    session_id UUID,
    employee_id UUID REFERENCES employee(id),
    status VARCHAR(20), -- 'initializing', 'active', 'terminating'
    started_at TIMESTAMP,
    terminated_at TIMESTAMP
);
```

## Related

- **Runtime Orchestration Core** (#4) - Base runtime
- **Runtime Registry** (#6) - Runtime selection