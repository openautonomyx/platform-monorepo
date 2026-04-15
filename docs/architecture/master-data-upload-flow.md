# Master Data Upload Flow

## Overview

The Master Data Upload Flow module governs bulk data uploads for departments, job titles, skills, tools, and policies. It supports validation, deduplication, and rollback.

## Entities

### UploadBatch

```sql
CREATE TABLE upload_batch (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    entity_type VARCHAR(50), -- 'department', 'job_title', 'skill', 'tool', 'policy'
    file_name VARCHAR(200),
    file_path VARCHAR(500),
    row_count INTEGER,
    valid_count INTEGER,
    invalid_count INTEGER,
    status VARCHAR(20), -- 'uploaded', 'validating', 'preview', 'approved', 'published', 'rolled_back'
    uploaded_by UUID REFERENCES employee(id),
    approved_by UUID REFERENCES employee(id),
    created_at TIMESTAMP DEFAULT NOW(),
    published_at TIMESTAMP
);
```

### UploadValidation

```sql
CREATE TABLE upload_validation (
    id UUID PRIMARY KEY,
    batch_id UUID REFERENCES upload_batch(id),
    row_number INTEGER,
    field_name VARCHAR(100),
    value TEXT,
    validation_rule VARCHAR(100),
    error_message TEXT,
    is_valid BOOLEAN
);
```

### UploadDeduplication

```sql
CREATE TABLE upload_deduplication (
    id UUID PRIMARY KEY,
    batch_id UUID REFERENCES upload_batch(id),
    source_key VARCHAR(200),
    target_id UUID,
    action VARCHAR(20) -- 'skip', 'update', 'create'
);
```

## Lifecycle

```
[Upload] → [Validate] → [Preview] → [Approve] → [Publish]
                                        ↓
                                   [Rollback]
```

## Validation Rules

| Rule | Description |
|------|-------------|
| required | Field cannot be empty |
| unique | Must be unique in tenant |
| reference | Must reference valid entity |
| format | Matches pattern/regex |

## Startup vs Enterprise Controls

### Startup

- Manual approval
- Basic validation
- No rollback complexity

### Enterprise

- Multi-stage approval
- Audit trail
- Rollback windows

## Related Modules

- **Organization & Identity Core** (#1) - Departments, job titles
- **Tool Registry** (#7) - Tools
- **Skill Lifecycle** (#8) - Skills