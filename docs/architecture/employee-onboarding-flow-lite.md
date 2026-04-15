# Employee Onboarding Flow Lite

## Overview

The Employee Onboarding Flow Lite module manages new employee onboarding including identity creation, role assignment, and policy acknowledgements.

## Entities

### OnboardingWorkflow

```sql
CREATE TABLE onboarding_workflow (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    employee_id UUID REFERENCES employee(id),
    status VARCHAR(20), -- 'pending', 'in_progress', 'completed', 'cancelled'
    current_step INTEGER,
    started_at TIMESTAMP,
    completed_at TIMESTAMP
);
```

### OnboardingStep

```sql
CREATE TABLE onboarding_step (
    id UUID PRIMARY KEY,
    workflow_id UUID REFERENCES onboarding_workflow(id),
    step_type VARCHAR(50), -- 'identity', 'role', 'policy', 'training', 'approval'
    step_order INTEGER,
    status VARCHAR(20), -- 'pending', 'completed', 'skipped'
    completed_by UUID,
    completed_at TIMESTAMP
);
```

### PolicyAcknowledgement

```sql
CREATE TABLE policy_acknowledgement (
    id UUID PRIMARY KEY,
    employee_id UUID REFERENCES employee(id),
    policy_id UUID,
    acknowledged_at TIMESTAMP,
    ip_address VARCHAR(50)
);
```

### TrainingRequirement

```sql
CREATE TABLE training_requirement (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    name VARCHAR(200),
    type VARCHAR(50), -- 'compliance', 'security', 'role_specific'
    duration_minutes INTEGER,
    is_required BOOLEAN DEFAULT TRUE
);
```

### TrainingCompletion

```sql
CREATE TABLE training_completion (
    id UUID PRIMARY KEY,
    employee_id UUID REFERENCES employee(id),
    training_id UUID REFERENCES training_requirement(id),
    completed_at TIMESTAMP,
    score INTEGER
);
```

## Onboarding Stages

```
[Account Created] → [Identity Verified] → [Role Assigned] 
        ↓                                    ↓
[Policies Acknowledged] ← [Training Completed] ← [Manager Approval]
        ↓
    [Onboarding Complete]
```

## Automation Opportunities

| Stage | Can Automate |
|-------|-------------|
| Account creation | Yes |
| Role assignment | Manual |
| Policy acknowledgement | Yes |
| Training | Yes (assign) |
| Manager approval | Manual |

## Future IGA Tie-in

- Identity verification workflows
- Access request integration
- Background check integration

## Related Modules

- **Organization & Identity Core** (#1) - Employee, role
- **Policy Packs** (#17) - Policies