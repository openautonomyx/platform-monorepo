# Organization & Identity Core

## Overview

The Organization & Identity Core module manages tenants, employees, agents, and their relationships within the platform. It provides the foundation for identity management while keeping employee and agent concerns separate.

## Entity Model

### Tenant

Represents a logical organization boundary. Each tenant operates independently with isolated data.

```
Tenant
├── id: UUID (PK)
├── name: string
├── domain: string (unique)
├── status: enum (active, suspended, pending)
├── created_at: timestamp
├── updated_at: timestamp
└── settings: JSON
```

### Employee

Represents a human user within a tenant. Employees are the primary identity holders.

```
Employee
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── email: string (unique per tenant)
├── display_name: string
├── status: enum (active, inactive, suspended)
├── department_id: UUID (FK → Department, nullable)
├── job_title_id: UUID (FK → JobTitle, nullable)
├── seniority_id: UUID (FK → Seniority, nullable)
├── manager_id: UUID (FK → Employee, nullable)
├── hire_date: date
├── termination_date: date (nullable)
├── created_at: timestamp
└── updated_at: timestamp
```

### Agent

Represents an AI agent identity associated with an employee. Agents act on behalf of employees.

```
Agent
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── name: string
├── agent_type: enum (human_facing, task_worker, hybrid)
├── primary_employee_id: UUID (FK → Employee)
├── status: enum (active, inactive, suspended)
├── capabilities: JSON (array of capability refs)
├── runtime_config: JSON
├── created_at: timestamp
└── updated_at: timestamp
```

### Employee_Agent_Assignment

Maps the relationship between employees and their assigned agents. Supports one employee to many agents.

```
Employee_Agent_Assignment
├── id: UUID (PK)
├── employee_id: UUID (FK → Employee)
├── agent_id: UUID (FK → Agent)
├── assignment_type: enum (primary, secondary, temporary)
├── is_primary: boolean (only one primary per employee)
├── access_level: enum (full, limited, restricted)
├── effective_from: date
├── effective_to: date (nullable)
├── created_at: timestamp
└── updated_at: timestamp
```

### Department

Organizational unit for grouping employees.

```
Department
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── name: string
├── code: string
├── parent_id: UUID (FK → Department, nullable)
├── manager_id: UUID (FK → Employee, nullable)
├── headcount_target: integer (nullable)
├── created_at: timestamp
└── updated_at: timestamp
```

### JobTitle

Defines job roles within the organization.

```
JobTitle
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── title: string
├── code: string
├── department_id: UUID (FK → Department)
├── level_min: integer
├── level_max: integer
├── created_at: timestamp
└── updated_at: timestamp
```

### Seniority

Career level progression within job titles.

```
Seniority
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── job_title_id: UUID (FK → JobTitle)
├── level: integer
├── label: string (e.g., "Junior", "Mid", "Senior", "Lead", "Principal")
├── created_at: timestamp
└── updated_at: timestamp
```

## Relationship Model

### Employee ↔ Agent Relationships

```
Employee (1) ────── (N) Employee_Agent_Assignment (N) ────── (1) Agent
     │
     └── Primary Agent (one per employee)
     └── Secondary Agents (many allowed)
     └── Temporary Agents (time-bounded)
```

### Organizational Hierarchy

```
Tenant (1) ────── (N) Department
     │
     └── (recursive) Department.parent_id → Department
     │
     └── (N) Employee
     │
     └── (N) JobTitle
```

### Role Mapping (Basic)

Basic role assignment at startup:

```
Employee (1) ────── (N) EmployeeRole
     │
     └── role: enum (user, admin, manager, viewer)
```

## Basic Lifecycle

### Employee Lifecycle

```
[Join] → [Active] → [Inactive] → [Terminated]
              ↓
         [Rehire] → [Active]
```

### Agent Lifecycle

```
[Provision] → [Active] → [Inactive] → [Retired]
              ↓
         [Update Config]
```

### Assignment Lifecycle

```
[Assign] → [Active] → [Modify] → [Revoke]
                    ↓
              [Extend/Temp]
```

## Startup Scope vs Later Enterprise Scope

### Startup Scope (Phase 1)

- Single tenant or small multi-tenant
- Basic employee/agent separation maintained
- One primary human-facing agent per employee
- Simple role mapping (user/admin)
- Manual department and job title management
- No complex reporting chains

### Enterprise Scope (Future Phase 2-3)

- Complex organizational hierarchies
- Full IGA (Identity Governance & Administration)
- SoD (Segregation of Duties) rules
- Access certifications
- Privileged access management
- Temporary access workflows
- Delegated authority windows

## Open Questions for Future IGA Integration

1. **Should agents have their own identity lifecycle separate from employees?**
   - Currently: Agent lifecycle tied to employee
   - Future: Independent agent identity governance?

2. **How to handle agent-to-agent delegation?**
   - Need path for agents to delegate to other agents
   - Chain-of-authority tracking

3. **Should departments have agent quotas or capacity?**
   - Headcount vs agent capacity
   - Cost governance

4. **What audit trail is needed for agent actions?**
   - Agent as actor vs employee as actor
   - Attribution model

5. **How to handle cross-tenant agent access?**
   - Shared services agents
   - Cross-tenant delegation

## Related Modules

- **Identity Governance Lite** (#16) - Future governance layer
- **Access Reviews & Certifications** (#14) - Access certification
- **Fine-Grained Access Control** (#13) - Extended access model
- **Employee Onboarding Flow Lite** (#11) - Onboarding process
- **Basic Audit & Activity Log** (#12) - Audit trail