# Basic Audit & Activity Log

## Overview

The Basic Audit & Activity Log module provides a usable audit trail without enterprise complexity.

## Entities

### ActivityLog

```sql
CREATE TABLE activity_log (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    actor_type VARCHAR(50), -- 'employee', 'agent', 'system'
    actor_id UUID,
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    details JSONB,
    ip_address VARCHAR(50),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### SensitiveActionLog

```sql
CREATE TABLE sensitive_action_log (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    action VARCHAR(100) NOT NULL,
    actor_type VARCHAR(50),
    actor_id UUID,
    target_type VARCHAR(50),
    target_id UUID,
    justification TEXT,
    status VARCHAR(20), -- 'pending', 'approved', 'rejected'
    approver_id UUID,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### ApprovalOverrideLog

```sql
CREATE TABLE approval_override_log (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    original_approval_id UUID,
    override_approver_id UUID REFERENCES employee(id),
    reason TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### RuntimeToolUsageLog

```sql
CREATE TABLE runtime_tool_usage_log (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    session_id UUID,
    runtime_id UUID,
    tool_name VARCHAR(100),
    execution_status VARCHAR(20),
    duration_ms INTEGER,
    created_at TIMESTAMP DEFAULT NOW()
);
```

## Minimum Required Audit Events

| Event Category | Events |
|----------------|--------|
| Authentication | login, logout, failed_login |
| Authorization | role_change, access_grant, access_revoke |
| Data | create, update, delete, export |
| Configuration | setting_change, policy_update |
| Runtime | tool_execution, agent_action |

## Storage Model

- **Primary**: PostgreSQL (activity_log, sensitive_action_log)
- **Archive**: S3/cold storage for events > 90 days

## Retention Basics

| Data Type | Retention |
|-----------|-----------|
| Activity logs | 1 year |
| Sensitive actions | 7 years |
| Runtime logs | 90 days |
| Export events | 1 year |

## Future Evidence/Export Path

- SOC 2 evidence export
- DPA access logs
- Compliance report generation

## Related Modules

- **Organization & Identity Core** (#1) - Actor tracking
- **Workflow & Control Plane Core** (#3) - Approval logging
- **Tool Registry** (#7) - Tool usage