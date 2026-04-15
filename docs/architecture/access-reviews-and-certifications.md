# Access Reviews & Certifications

## Overview

Access reviews manage privileged and stale access, with reviewer assignments and certification decisions.

## Entities

```sql
-- Review Campaign
CREATE TABLE access_review_campaign (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    name VARCHAR(200),
    review_type VARCHAR(50), -- 'privileged', 'stale', 'periodic'
    target_resources JSONB, -- types to review
    reviewer_groups JSONB,
    due_date TIMESTAMP,
    status VARCHAR(20), -- 'draft', 'active', 'completed'
    created_at TIMESTAMP
);

-- Review Assignment
CREATE TABLE access_review_assignment (
    id UUID PRIMARY KEY,
    campaign_id UUID REFERENCES access_review_campaign(id),
    reviewer_id UUID REFERENCES employee(id),
    resource_type VARCHAR(50),
    resource_id UUID,
    status VARCHAR(20), -- 'pending', 'reviewed'
    decision VARCHAR(20), -- 'confirm', 'remove', 'reduce'
    justification TEXT,
    reviewed_at TIMESTAMP
);
```

## Review Types

| Type | Trigger | Scope |
|------|---------|-------|
| privileged | periodic | admin, elevated access |
| stale | 90 days inactive | all access |
| periodic | scheduled | role-based |

## Related Modules

- **Organization & Identity Core** (#1) - Employee roles
- **Fine-Grained Access Control** (#13) - Access granted