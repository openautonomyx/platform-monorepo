# Customer Expectations & Competitor Benchmarking

## Overview

Strategic product intelligence: customer expectations, competitor benchmarks, capability gaps, objection patterns, roadmap pressure.

## Entities

### CustomerExpectation

```sql
CREATE TABLE customer_expectation (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    source VARCHAR(50), -- 'support', 'sales', 'feedback', 'survey'
    category VARCHAR(50), -- 'feature', 'performance', 'integration', 'pricing'
    description TEXT,
    frequency INTEGER, -- times mentioned
    priority VARCHAR(20), -- 'low', 'medium', 'high'
    created_at TIMESTAMP
);
```

### CompetitorBenchmark

```sql
CREATE TABLE competitor_benchmark (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    competitor_name VARCHAR(100),
    benchmark_date DATE,
    capabilities JSONB,
    pricing JSONB,
    strengths JSONB,
    weaknesses JSONB,
    created_at TIMESTAMP
);
```

### ObjectionPattern

```sql
CREATE TABLE objection_pattern (
    id UUID PRIMARY KEY,
    tenant_id UUID REFERENCES tenant(id),
    objection_type VARCHAR(100),
    frequency INTEGER,
    overcoming_text TEXT,
    created_at TIMESTAMP
);
```

## Related

- **Feedback/Suggestions** (#9) - Customer input
- **Roadmap** - Pressure signals