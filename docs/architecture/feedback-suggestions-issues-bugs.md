# Feedback, Suggestions, Issues, and Bugs

## Overview

This module captures user and product feedback, suggestions, issues, and bugs. Feedback is linked to runtime, tool, skill, workflow, or policy when possible.

## Entities

### FeedbackItem

```sql
CREATE TABLE feedback_item (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    category VARCHAR(50), -- 'product', 'feature', 'process', 'other'
    type VARCHAR(50), -- 'feedback', 'suggestion', 'issue', 'bug'
    title VARCHAR(200) NOT NULL,
    description TEXT,
    sentiment VARCHAR(20), -- 'positive', 'neutral', 'negative'
    status VARCHAR(20), -- 'open', 'acknowledged', 'in_progress', 'resolved', 'closed'
    priority VARCHAR(20), -- 'low', 'medium', 'high', 'critical'
    reporter_id UUID REFERENCES employee(id),
    assignee_id UUID REFERENCES employee(id),
    linked_type VARCHAR(50), -- 'runtime', 'tool', 'skill', 'workflow', 'policy'
    linked_id UUID,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);
```

### Incident (for bugs)

```sql
CREATE TABLE incident (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    feedback_id UUID REFERENCES feedback_item(id),
    incident_type VARCHAR(50), -- 'bug', 'outage', 'security', 'data'
    severity VARCHAR(20), -- 'sev1', 'sev2', 'sev3', 'sev4'
    status VARCHAR(20), -- 'investigating', 'identified', 'mitigating', 'resolved'
    impact_description TEXT,
    root_cause TEXT,
    resolution TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    resolved_at TIMESTAMP
);
```

## Lifecycle

```
[New] → [Acknowledged] → [In Progress] → [Resolved] → [Closed]
              ↓              ↓
         [Duplicate]    [Wont Fix]
```

## Priority/Severity Model

| Priority | Response Time | Description |
|----------|---------------|-------------|
| critical | 1 hour | System down |
| high | 4 hours | Major impact |
| medium | 24 hours | Workaround available |
| low | 1 week | Minor issue |

## Linkage to Roadmap and Operations

- Feedback status → Roadmap prioritization
- Issue trends → Technical debt tracking
- Bug frequency → Quality metrics

## Related Modules

- **Collaboration & Work Core** (#2) - Feature feedback
- **Advanced Evaluation** (#21) - Quality tracking