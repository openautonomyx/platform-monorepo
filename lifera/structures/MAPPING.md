# Lifer Entity Mapping

## Core Lifer Entities → Custom Entities

| Lifer Core Entity | Maps To | Architecture Module | Custom Fields Needed |
|-----------------|--------|-------------------|---------------------|
| **User** | Employee | Org & Identity #1 | department, jobTitle, seniority, manager |
| **Organization** | Department | Org & Identity #1 | parentOrg, headcountTarget |
| **Role** | Basic Role | Org & Identity #1 | (use Lifer roles) |
| **Site** | Group/Channel | Collab & Work #2 | type, privacy |
| **Wiki** | Project | Collab & Work #2 | (custom fields) |
| **Blog** | Feedback | Feedback #9 | category, sentiment |
| **Document Library** | File | Collab & Work #2 | (custom fields) |
| **Web Content** | Announcement | Collab & Work #2 | channel, expiryDate |

## Simplified Structure Mapping

### Use Lifer Core (No Custom Structure)

```
✓ User → Employee (built-in, extend with custom fields)
✓ Organization → Department (built-in, add headcount)
✓ Site → Group/Channel (built-in)
✓ Role → Basic Role (built-in)
✓ Document Library → Files (built-in)
✗ Agent → Custom Structure (Lifer doesn't have)
✗ Task → Custom Structure (Lifer doesn't have)
✗ Deadline → Custom Structure
✗ Milestone → Custom Structure
✗ Feedback → Use Blog (builtin) + custom fields
```

### Custom Structures Only

```xml
agent.xml       → Agent (no Lifer equivalent)
task.xml        → Task (no Lifer equivalent)
deadline.xml    → Deadline (no Lifer equivalent)
milestone.xml   → Milestone (no Lifer equivalent)
```

### Use Lifer + Extensions

```xml
employee.xml    → Uses User (Lifer) + custom fields
department.xml → Uses Organization (Lifer) + headcount
group.xml      → Uses Site (Lifer) + type
feedback.xml  → Uses Blog (Lifer) + category
```

## Recommended Implementation

**Lifer Built-in (no structure needed)**:
- User
- Organization  
- Site
- Role
- User Group
- Document Library

**Custom Structures**:
- agent.xml
- task.xml
- deadline.xml
- milestone.xml
- employee.xml (extend User)
- department.xml (extend Organization)