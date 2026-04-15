# Audit & Evidence Export

## Overview

Upgrade from activity logs to evidence-grade audit with compliance exports and review.

## Entities

### EvidenceBundle

```sql
CREATE TABLE evidence_bundle (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    bundle_type VARCHAR(50), -- 'soc2', 'dpa', 'hipaa'
    date_from DATE,
    date_to DATE,
    status VARCHAR(20), -- 'generating', 'ready', 'exported'
    file_path VARCHAR(500),
    created_at TIMESTAMP
);
```

### AuditEvidence

```sql
CREATE TABLE audit_evidence (
    id UUID PRIMARY KEY,
    bundle_id UUID REFERENCES evidence_bundle(id),
    evidence_type VARCHAR(50),
    control_id VARCHAR(50),
    event_count INTEGER,
    summary TEXT,
    created_at TIMESTAMP
);
```

## Related

- **Basic Audit & Activity Log** (#12) - Activity logs
- **Compliance Packs** (#29) - Pack definitions