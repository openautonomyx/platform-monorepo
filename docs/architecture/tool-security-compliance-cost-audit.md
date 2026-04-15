# Tool Security, Compliance, and Cost Audit

## Overview

Enterprise-grade tool governance: allowlists, security profiles, compliance audit, cost attribution, anomalies, approval-required tools.

## Entities

### ToolSecurityProfile

```sql
CREATE TABLE tool_security_profile (
    id UUID PRIMARY KEY,
    tool_id UUID REFERENCES tool(id),
    risk_score INTEGER, -- 0-100
    data_access VARCHAR(50), -- 'none', 'read', 'write', 'admin'
    network_access BOOLEAN,
    authentication_required BOOLEAN,
    compliance_flags JSONB,
    created_at TIMESTAMP
);
```

### ToolApproval

```sql
CREATE TABLE tool_approval (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    tool_id UUID REFERENCES tool(id),
    requester_id UUID REFERENCES employee(id),
    justification TEXT,
    status VARCHAR(20), -- 'pending', 'approved', 'rejected'
    approver_id UUID,
    created_at TIMESTAMP
);
```

### ToolCostAttribution

```sql
CREATE TABLE tool_cost (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    tool_id UUID REFERENCES tool(id),
    usage_date DATE,
    calls_count INTEGER,
    total_cost DECIMAL,
    cost_center VARCHAR(100),
    created_at TIMESTAMP
);
```

### ToolAnomaly

```sql
CREATE TABLE tool_anomaly (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    tool_id UUID REFERENCES tool(id),
    anomaly_type VARCHAR(50), -- 'volume', 'cost', 'duration'
    threshold_value DECIMAL,
    actual_value DECIMAL,
    detected_at TIMESTAMP
);
```

## Related

- **Tool Registry** (#7) - Tool definition
- **Basic Audit** (#12) - Tool usage logging