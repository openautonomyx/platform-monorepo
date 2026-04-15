# Tool Registry & Basic Governance

## Overview

The Tool Registry module manages available tools, their categories, and basic allow/deny governance. It supports startup-friendly allowlists while deferring deep compliance to later stages.

## Tool Registry Schema

```sql
CREATE TABLE tool (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    name VARCHAR(100) NOT NULL,
    description TEXT,
    tool_type VARCHAR(50) NOT NULL, -- 'client', 'server', 'mcp'
    provider VARCHAR(50), -- 'openai', 'anthropic', 'custom', 'mcp_server'
    execution_category VARCHAR(50), -- 'read', 'write', 'delete', 'admin'
    functional_category VARCHAR(50), -- 'communication', 'productivity', 'development', 'data', 'integration'
    is_enabled BOOLEAN DEFAULT TRUE,
    is_allowlisted BOOLEAN DEFAULT FALSE, -- require explicit allow
    runtime_compatibility JSONB, -- array of runtime IDs
    config JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);
```

## Client vs Server vs MCP Tools

### Client Tools

Tools that execute on the client side (browser, mobile app).

```
Client Tool
├── executes_in: client_application
├── examples: ui_interaction, local_storage, native_apis
└── security: sandboxed by client app
```

### Server Tools

Tools that execute on the server side.

```
Server Tool
├── executes_in: server_runtime
├── examples: api_call, database_query, file_operations
└── security: server-side validation
```

### MCP Tools

Tools from Model Context Protocol servers.

```
MCP Tool
├── executes_in: mcp_server
├── protocol: MCP (JSON-RPC 2.0)
├── examples: filesystem, search, custom_integrations
└── security: MCP server authentication
```

## Category Model

### Execution Categories

| Category | Description | Risk Level |
|----------|-------------|------------|
| read | Read-only operations | Low |
| write | Data modification | Medium |
| delete | Data deletion | High |
| admin | Administrative operations | High |

### Functional Categories

| Category | Description | Examples |
|----------|-------------|----------|
| communication | Messaging, notifications | email, slack, chat |
| productivity | Office, documents | docs, sheets, calendar |
| development | Code, APIs | git, github, jira |
| data | Database, storage | postgres, s3, redis |
| integration | External services | stripe, salesforce |
| ai | AI operations | embeddings, generation |
| system | Platform operations | users, roles, config |

## Startup Tool Policy Model

### Basic Allowlist

```sql
CREATE TABLE tool_policy (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    tool_id UUID REFERENCES tool(id),
    policy_type VARCHAR(20), -- 'allow', 'deny'
    policy_scope VARCHAR(20), -- 'tool', 'category', 'global'
    role_restrictions JSONB, -- array of role IDs
    status enum('active', 'suspended')
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Default Policy at Startup

| Functional Category | Default | Override Allowed |
|-------------------|---------|------------------|
| communication | allow | yes |
| productivity | allow | yes |
| development | deny | yes |
| data | allow | yes |
| integration | allow | yes |
| ai | allow | yes |
| system | deny | no |

### Startup Allowlist Rules

1. Default allow for low-risk categories
2. Require explicit allow for development/tools
3. System tools blocked by default
4. Role-based restrictions allowed

## Future Enterprise Tool Governance Path

### Phase 2-3 Features

- Tool security profiles
- Compliance audit requirements
- Cost attribution
- Approval-required tool lists
- Tool usage anomaly detection

### Enterprise Governance

| Feature | Description |
|---------|-------------|
| Security profiles | Tool risk scoring |
| Compliance audit | SOC 2, DPA requirements |
| Cost tracking | Per-tool cost attribution |
| Approval workflow | Multi-level approval |
| Anomaly detection | Unusual tool usage |

## Related Modules

- **Organization & Identity Core** (#1) - Role mapping
- **Workflow & Control Plane Core** (#3) - Tool usage in tasks
- **Runtime Orchestration Core** (#4) - Tool execution
- **Runtime Registry & Selection** (#6) - Runtime compatibility
- **Tool Security/Compliance/Cost Audit** (#20) - Enterprise tool governance
- **Basic Audit & Activity Log** (#12) - Tool usage logging