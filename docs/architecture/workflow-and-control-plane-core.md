# Workflow & Control Plane Core

## Overview

The Workflow & Control Plane Core module manages tasks, deadlines, milestones, and workflow execution. It separates workflow definition from runtime execution and provides governance controls.

## Entity Map

### Task

Represents a work item that can be assigned to employees or agents.

```
Task
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── title: string
├── description: text
├── status: enum (pending, in_progress, blocked, completed, cancelled)
├── priority: enum (low, medium, high, urgent)
├── task_type: enum (action, review, approval, decision)
├── parent_type: enum (project, group, task, workflow)
├── parent_id: UUID
├── assignee_id: UUID (FK → Employee, nullable)
├── assignee_agent_id: UUID (FK → Agent, nullable)
├── due_date: timestamp (nullable)
├── estimated_hours: decimal (nullable)
├── actual_hours: decimal (nullable)
├── created_by: UUID (FK → Employee)
├── created_at: timestamp
├── updated_at: timestamp
└── completed_at: timestamp (nullable)
```

### Deadline

A time-bound commitment or deliverable.

```
Deadline
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── title: string
├── description: text
├── due_date: timestamp
├── reminder_sent: boolean
├── status: enum (upcoming, due, breached, completed, extended)
├── parent_type: enum (task, milestone, project)
├── parent_id: UUID
├── notify_employee_id: UUID (FK → Employee)
├── escalation_rule_id: UUID (FK → EscalationRule, nullable)
├── created_at: timestamp
└── updated_at: timestamp
```

### Milestone

A significant checkpoint within a project or workflow.

```
Milestone
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── name: string
├── description: text
├── due_date: timestamp
├── status: enum (pending, achieved, at_risk, missed)
├── parent_type: enum (project, workflow)
├── parent_id: UUID
├── sequence_order: integer
├── created_at: timestamp
└── updated_at: timestamp
```

### Reminder

An automated reminder for tasks, deadlines, or approvals.

```
Reminder
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── title: string
├── reminder_type: enum (deadline, approval, follow_up, escalation)
├── parent_type: enum (task, deadline, approval)
├── parent_id: UUID
├── sent_at: timestamp (nullable)
├── send_at: timestamp
├── recipient_type: enum (employee, agent, group)
├── recipient_id: UUID
├── message_template: text
├── created_at: timestamp
└── updated_at: timestamp
```

### Escalation

Defines escalation rules for overdue items.

```
Escalation
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── name: string
├── escalation_type: enum (deadline, approval, task, sla)
├── parent_type: enum (task, workflow, sla_policy)
├── parent_id: UUID
├── level: integer
├── escalate_to_employee_id: UUID (FK → Employee, nullable)
├── escalate_to_group_id: UUID (FK → Group, nullable)
├── escalate_to_agent_id: UUID (FK → Agent, nullable)
├── hours_until_escalation: integer
├── created_at: timestamp
└── updated_at: timestamp
```

### Approval

A governance step requiring authorization.

```
Approval
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── title: string
├── description: text
├── status: enum (pending, approved, rejected, override)
├── parent_type: enum (task, workflow, request)
├── parent_id: UUID
├── requester_id: UUID (FK → Employee)
├── approver_id: UUID (FK → Employee)
├── decision_note: text (nullable)
├── decided_at: timestamp (nullable)
├── due_date: timestamp (nullable)
├── created_at: timestamp
└── updated_at: timestamp
```

### Decision

A recorded decision with rationale.

```
Decision
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── title: string
├── description: text
├── decision_type: enum ( strategic, tactical, operational)
├── status: enum (draft, pending, finalized, superseded)
├── parent_type: enum (project, task, workflow)
├── parent_id: UUID
├── decided_by: UUID (FK → Employee)
├── rationale: text
├── alternatives_considered: JSON (nullable)
├── effective_from: date
├── effective_to: date (nullable)
├── created_at: timestamp
└── updated_at: timestamp
```

### Override

An override of standard workflow or policy.

```
Override
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── title: string
├── description: text
├── override_type: enum (policy, workflow, access, sla)
├── parent_type: enum (policy, workflow, task)
├── parent_id: UUID
├── requester_id: UUID (FK → Employee)
├── approver_id: UUID (FK → Employee)
├── justification: text
├── status: enum (pending, approved, rejected)
├── effective_from: timestamp
├── effective_to: timestamp (nullable)
├── created_at: timestamp
└── updated_at: timestamp
```

### Delegation

Temporary delegation of authority.

```
Delegation
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── delegator_id: UUID (FK → Employee)
├── delegate_id: UUID (FK → Employee)
├── delegation_type: enum (task, approval, decision)
├── parent_type: enum (task, approval, decision)
├── parent_id: UUID
├── effective_from: timestamp
├── effective_to: timestamp
├── reason: text
├── status: enum (active, revoked, expired)
├── created_at: timestamp
└── updated_at: timestamp
```

### ExecutionRequest

A request to execute a workflow or agent task.

```
ExecutionRequest
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── title: string
├── request_type: enum (workflow, agent_task, batch)
├── payload: JSON
├── status: enum (pending, dispatched, running, completed, failed, cancelled)
├── requested_by: UUID (FK → Employee)
├── runtime_id: UUID (FK → Runtime, nullable)
├── result: JSON (nullable)
├── error_detail: text (nullable)
├── created_at: timestamp
├── dispatched_at: timestamp (nullable)
├── completed_at: timestamp (nullable)
└── updated_at: timestamp
```

### ExecutionHistory

A record of workflow or task execution.

```
ExecutionHistory
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── execution_request_id: UUID (FK → ExecutionRequest)
├── step_name: string
├── status: enum (pending, running, completed, failed, skipped)
├── input_data: JSON
├── output_data: JSON (nullable)
├── error_detail: text (nullable)
├── started_at: timestamp
├── completed_at: timestamp (nullable)
└── duration_ms: integer (nullable)
```

## Lifecycle Summary

### Task Lifecycle

```
[Created] → [Assigned] → [In Progress] → [Blocked] → [In Progress] → [Completed]
                    ↓
              [Cancelled]
```

### Approval Lifecycle

```
[Requested] → [Pending] → [Approved] / [Rejected]
                              ↓
                        [Override Requested] → [Approved] / [Rejected]
```

### Execution Lifecycle

```
[Requested] → [Dispatched] → [Running] → [Completed] / [Failed]
                                                ↓
                                          [Retried]
```

## Startup vs Enterprise Responsibilities

### Startup Scope (Phase 1)

- Simple task assignment to employees
- Basic deadline tracking
- Simple approval workflows
- Manual escalation handling
- No complex SLA enforcement
- Basic execution history

| Feature | Scope |
|---------|-------|
| Tasks | Employee assignment only |
| Deadlines | Manual due dates |
| Approvals | Simple approve/reject |
| Escalation | Manual handling |
| Decisions | Manual recording |
| Execution | Basic dispatch |

### Enterprise Scope (Future Phase 2-3)

- Agent task assignment
- Automatic SLA monitoring
- Complex approval chains
- Automated escalation
- Decision tracking with alternatives
- Full execution analytics

## Related Modules

- **Organization & Identity Core** (#1) - Employee/agent assignment
- **Runtime Orchestration Core** (#4) - Execution runtimes
- **Runtime Registry & Selection** (#6) - Runtime selection for execution
- **Policy Packs & SLA Governance** (#17) - SLA policies
- **Event Calendar & Time Governance** (#22) - Time events
- **Basic Audit & Activity Log** (#12) - Approval/override logging