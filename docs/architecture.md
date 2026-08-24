# Architecture

## Component overview

```
                    ┌────────────────────────────────────────────────┐
                    │            ZeroQueries Platform                │
                    │                                                │
  User (Arabic/EN)  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │   ┌──────────────┐
 ───────────────────┼─▶│  Web UI  │─▶│  NQL     │─▶│  Query       │──┼──▶│  Your        │
  (RBAC-authenticated)│  │ + Chat   │  │  Engine  │  │  Executor    │  │   │  Database    │
                    │  └──────────┘  └──────────┘  └──────────────┘  │   │  (read-only) │
                    │        │              │              │         │   └──────────────┘
                    │        └──────┬───────┴──────┬───────┘         │
                    │               ▼              ▼                  │
                    │       ┌──────────────┐  ┌──────────────┐        │
                    │       │  RBAC        │  │  Audit Log   │        │
                    │       │  (role-based)│  │  (full trace)│        │
                    │       └──────────────┘  └──────────────┘        │
                    └────────────────────────────────────────────────┘
```

## Layer descriptions

### 1. Web UI (Arabic + English)
- Chat-style interface, bilingual (AR/EN), RTL-aware
- Question history, saved answers, chart rendering
- Role-scoped views — a user only sees the data sources their role permits

### 2. NQL Engine (Natural Language → SQL)
- Parses the question in Arabic or English
- Resolves entities against the actual database schema (tables, columns, joins)
- Validates intent before any SQL is generated
- Confidence scoring — ambiguous questions are clarified rather than guessed

### 3. Query Executor
- Executes generated SQL with **read-only credentials** (SELECT-only, no DDL/DML)
- Statement validation: rejects anything that is not a read query
- Row-level / column-level security applied via RBAC filters
- Result capped and shaped for answer + chart rendering

### 4. RBAC
- Roles map to data scopes (e.g. `branch_manager` → own branch only, `finance` → finance tables only)
- Enforced at both application and database levels

### 5. Audit Log
- Every question, generated SQL, execution result summary, and access attempt is recorded
- Immutable, exportable logs for compliance review

## Deployment model

| Model | Description |
|---|---|
| **On-premises** | Installed inside the customer's network / data center |
| **Private cloud (VPC)** | Dedicated isolated deployment in a private cloud tenant |
| **Hybrid** | Platform on-prem, optional managed monitoring by vendor (opt-in) |

Data never leaves the customer's boundary in any model.

## Compliance posture

- **PDPL (Saudi Personal Data Protection Law)** — data minimization, access control, auditability, purpose limitation built into the architecture
- **Read-only guarantee** — no write path exists at any layer
- **Role-based data scoping** — least-privilege by default
- **Full audit trail** — defensible evidence of who asked what, and what data was returned

## Security checklist for evaluation teams

- [ ] TLS in transit (end-to-end)
- [ ] Read-only DB credentials (SELECT-only role at DB level)
- [ ] Statement-level validation (no DDL/DML possible)
- [ ] RBAC scoping at application + DB level
- [ ] Full audit logging with export
- [ ] No data exfiltration path (on-prem boundary)
- [ ] PDPL data-handling review
