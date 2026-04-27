# AI-Powered Operational CRM — Database Schema

## Tables

### customers
| Column        | Type         | Notes                        |
|---------------|--------------|------------------------------|
| id            | SERIAL PK    |                              |
| name          | VARCHAR(255) | NOT NULL                     |
| email         | VARCHAR(255) | UNIQUE                       |
| phone         | VARCHAR(50)  |                              |
| segment       | VARCHAR(50)  | e.g. enterprise, smb, startup|
| created_at    | TIMESTAMPTZ  | DEFAULT now()                |
| updated_at    | TIMESTAMPTZ  |                              |

### contacts
| Column        | Type         | Notes                        |
|---------------|--------------|------------------------------|
| id            | SERIAL PK    |                              |
| customer_id   | INT FK       | → customers.id               |
| first_name    | VARCHAR(100) |                              |
| last_name     | VARCHAR(100) |                              |
| role          | VARCHAR(100) |                              |
| email         | VARCHAR(255) |                              |

### deals
| Column        | Type         | Notes                        |
|---------------|--------------|------------------------------|
| id            | SERIAL PK    |                              |
| customer_id   | INT FK       | → customers.id               |
| title         | VARCHAR(255) |                              |
| value         | NUMERIC(12,2)|                              |
| stage         | VARCHAR(50)  | prospecting/qualified/closed |
| probability   | SMALLINT     | 0–100 (AI-estimated)         |
| closed_at     | DATE         |                              |
| created_at    | TIMESTAMPTZ  | DEFAULT now()                |

### ai_recommendations
| Column        | Type         | Notes                            |
|---------------|--------------|----------------------------------|
| id            | SERIAL PK    |                                  |
| customer_id   | INT FK       | → customers.id                   |
| model_version | VARCHAR(50)  |                                  |
| recommendation| TEXT         | LLM output (validated)           |
| score         | NUMERIC(5,2) | confidence score                 |
| created_at    | TIMESTAMPTZ  | DEFAULT now()                    |

### audit_log
| Column        | Type         | Notes                            |
|---------------|--------------|----------------------------------|
| id            | SERIAL PK    |                                  |
| user_id       | INT          |                                  |
| action        | VARCHAR(100) | CREATE / UPDATE / DELETE / READ  |
| table_name    | VARCHAR(100) |                                  |
| record_id     | INT          |                                  |
| timestamp     | TIMESTAMPTZ  | DEFAULT now()                    |

## RBAC Roles
- **admin** — full read/write + audit log access
- **sales** — read/write customers, contacts, deals
- **analyst** — read-only on all tables + ai_recommendations write
- **readonly** — read-only on customers and deals

## Indexes
```sql
CREATE INDEX idx_deals_customer_id ON deals(customer_id);
CREATE INDEX idx_deals_stage ON deals(stage);
CREATE INDEX idx_ai_rec_customer_id ON ai_recommendations(customer_id);
CREATE INDEX idx_audit_log_timestamp ON audit_log(timestamp DESC);
```
