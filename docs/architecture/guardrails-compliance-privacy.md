# Guardrails, Privacy, and Compliance

## Overview

Runtime governance: prompt guardrails, action guardrails, tool-use guardrails, privacy controls, PII handling, runtime restrictions, memory write restrictions, approval triggers.

## Entities

### PromptGuardrail

```sql
CREATE TABLE prompt_guardrail (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    name VARCHAR(100),
    guardrail_type VARCHAR(50), -- 'content_filter', 'pii_filter', 'inject_prevention'
    rules JSONB,
    action VARCHAR(20), -- 'block', 'redact', 'alert'
    is_enabled BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP
);
```

### PrivacyControl

```sql
CREATE TABLE privacy_control (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    control_type VARCHAR(50), -- 'pii_masking', 'data_retention', 'access_control'
    target_fields JSONB,
    rules JSONB,
    created_at TIMESTAMP
);
```

## Related

- **Data Governance** (#15) - PII handling
- **Tool Registry** (#7) - Tool guardrails