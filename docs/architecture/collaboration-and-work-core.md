# Collaboration & Work Core

## Overview

The Collaboration & Work Core module manages products, projects, groups, channels, files, and user interactions. It maintains clear separation between organizational units (departments), work containers (projects), and collaboration spaces (groups/channels).

## Entity Model

### Product

A product represents a major offering or service line. Products contain multiple projects.

```
Product
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── name: string
├── description: text
├── status: enum (draft, active, archived, deprecated)
├── owner_id: UUID (FK → Employee)
├── created_at: timestamp
└── updated_at: timestamp
```

### Project

A project is a time-bounded work initiative. Projects belong to a product and are owned by employees.

```
Project
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── product_id: UUID (FK → Product)
├── name: string
├── description: text
├── status: enum (planning, active, on_hold, completed, archived)
├── owner_id: UUID (FK → Employee)
├── start_date: date
├── end_date: date (nullable)
├── budget: decimal (nullable)
├── created_at: timestamp
└── updated_at: timestamp
```

### Group

A group is a persistent collaboration container for ongoing team communication. Groups can be project-related or standalone.

```
Group
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── name: string
├── description: text
├── type: enum (team, project, department, company, external)
├── status: enum (active, archived)
├── privacy: enum (public, private, internal)
├── owner_id: UUID (FK → Employee)
├── parent_group_id: UUID (FK → Group, nullable)
├── created_at: timestamp
└── updated_at: timestamp
```

### Channel

A channel is a focused discussion space within a group or standalone. Channels support real-time and async communication.

```
Channel
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── group_id: UUID (FK → Group, nullable)
├── name: string
├── description: text
├── type: enum (chat, announcement, threaded, support)
├── status: enum (active, archived)
├── privacy: enum (public, private, direct)
├── owner_id: UUID (FK → Employee)
├── created_at: timestamp
└── updated_at: timestamp
```

### File

Represents a shared file or document.

```
File
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── name: string
├── mime_type: string
├── size_bytes: integer
├── storage_key: string (S3/local path)
├── version: integer
├── parent_type: enum (group, channel, project, product)
├── parent_id: UUID (FK)
├── owner_id: UUID (FK → Employee)
├── created_at: timestamp
└── updated_at: timestamp
```

### Comment

User-generated comments on files, channels, or other entities.

```
Comment
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── parent_type: enum (file, channel, project, product, task)
├── parent_id: UUID
├── author_id: UUID (FK → Employee)
├── content: text
├── mentions: JSON (array of employee IDs)
├── reply_to_id: UUID (FK → Comment, nullable)
├── created_at: timestamp
└── updated_at: timestamp
```

### Feedback

User feedback on products, projects, or platform features.

```
Feedback
├── id: UUID (PK)
├── tenant_id: UUID (FK → Tenant)
├── category: enum (product, feature, process, other)
├── parent_type: enum (product, project, channel, platform)
├── parent_id: UUID
├── author_id: UUID (FK → Employee)
├── title: string
├── content: text
├── sentiment: enum (positive, neutral, negative)
├── status: enum (open, acknowledged, resolved, closed)
├── created_at: timestamp
└── updated_at: timestamp
```

## Ownership Model

### Product Ownership

```
Product.owner_id → Employee
     ↓
     └── (N) Project.owner_id → Employee
```

### Group/Channel Ownership

```
Group.owner_id → Employee
     ↓
     └── Channel.owner_id → Employee (optional override)
```

### File Ownership

```
File.owner_id → Employee (uploader/maintainer)
     ↓
     └── parent_type + parent_id → Group/Channel/Project/Product
```

## Channel Roles

Basic role model for channel access:

```
ChannelRole
├── id: UUID (PK)
├── channel_id: UUID (FK → Channel)
├── principal_type: enum (employee, agent, group)
├── principal_id: UUID
├── role: enum (owner, moderator, member, guest)
├── created_at: timestamp
└── updated_at: timestamp
```

| Role | Permissions |
|------|-------------|
| owner | Full control, manage members, delete channel |
| moderator | Manage messages, manage members, archive |
| member | Post messages, upload files, react |
| guest | View only, no posting |

## Startup Scope

### Phase 1 (Startup)

- Products and projects for work organization
- Groups for team collaboration
- Channels for focused discussions
- Basic file sharing within groups/channels
- Simple comments on files and channels
- Basic feedback collection

### Feature Set

| Feature | Scope |
|---------|-------|
| Products | Limited (manual creation) |
| Projects | Time-bounded, simple tracking |
| Groups | Team-level, basic privacy |
| Channels | Chat + threaded, basic roles |
| Files | Upload/download, versioned |
| Comments | File + channel comments |
| Feedback | Simple product feedback |

## Future Enterprise Expansion

### Phase 2 (Growth)

- Project templates and workflows
- External guest access
- File templates
- Rich feedback categorization
- Basic analytics

### Phase 3 (Enterprise)

- Complex project hierarchies
- Cross-tenant collaboration
- Compliance-locked files
- Advanced analytics
- Integration with external systems

## Related Modules

- **Organization & Identity Core** (#1) - Employee ownership
- **Workflow & Control Plane Core** (#3) - Task/project integration
- **Product** entity here → Links to **Customer Expectations** (#18)
- **Feedback** here → Links to **Feedback/Suggestions/Issues/Bugs** (#9)
- Files → Links to **Data Governance** (#15)