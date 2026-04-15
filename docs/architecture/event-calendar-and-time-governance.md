# Event Calendar & Time Governance

## Overview

Time governance: deadlines, milestones, reminders, approval due dates, escalation due dates, demo/event calendars, blackout windows.

## Entities

### CalendarEvent

```sql
CREATE TABLE calendar_event (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    event_type VARCHAR(50), -- 'deadline', 'milestone', 'demo', 'blackout'
    title VARCHAR(200),
    description TEXT,
    start_time TIMESTAMP,
    end_time TIMESTAMP,
    parent_type VARCHAR(50),
    parent_id UUID,
    attendance_required JSONB,
    created_at TIMESTAMP
);
```

### TimeGovernanceRule

```sql
CREATE TABLE time_governance_rule (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    rule_type VARCHAR(50), -- 'working_hours', 'blackout', 'maintenance_window'
    applies_to JSONB, -- employee_ids, group_ids
    schedule JSONB,
    created_at TIMESTAMP
);
```

## Related

- **Workflow & Control Plane Core** (#3) - Deadlines, milestones
- **Policy Packs** (#17) - SLA policies