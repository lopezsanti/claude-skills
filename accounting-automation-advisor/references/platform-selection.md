# Platform Selection Guide — Accounting Automation

## Decision Matrix

| Dimension | recon-lite (Streamlit + SQLite) | n8n Workflow | CrewAI + FastAPI | n8n + MCP |
|-----------|--------------------------------|--------------|------------------|-----------|
| Build speed | Fastest | Fast | Moderate | Moderate |
| Developer skill required | Python basics | No-code | Python + API | Low-code + MCP config |
| Ops maintenance | Lowest | Low | Low (container) | Low |
| Testability | Good (pytest on tools) | Workflow-level | Strong (unit tests) | Workflow-level |
| Multi-tenant upgrade path | SQLite → PostgreSQL | Workflow vars or per-tenant instance | Auth + queue in front of FastAPI | Same + per-tenant credentials |
| Live data integration | Manual file upload | Native HTTP/webhook | Custom API clients | Native via MCP servers |
| Observability | Run log JSON | n8n built-in audit | OpenTelemetry | n8n built-in |
| Best for | Solo dev, fastest time-to-value | Firm IT, visual audit | Python team, deep tests | Live QuickBooks/bank feeds |

---

## Option A — recon-lite Pattern (Recommended starting point)

**What it is:** A single-file Streamlit application (~950 lines across 5 Python files) with
SQLite for persistence, deployed to Railway with one click.

**Stack:**
- UI: Streamlit (full web app in pure Python — no frontend code)
- Matching: rapidfuzz
- Data parsing: pandas (handles CSV / XLSX / XLS / TSV)
- Excel output: openpyxl
- Persistence: SQLite (zero config, handles 500+ customers easily)

**Why it wins for solo developers:**
- No Docker required to start
- No separate database server
- One-click Railway deployment (~$5/month)
- Upgrade path is documented inline (SQLite → Postgres, email intake via n8n, multi-user auth)

**Upgrade triggers:**
- > 500 customers or > 10,000 transactions/run → move matching to a background job queue
- Need email/automated file intake → add n8n workflow calling a thin API wrapper
- Multi-user login needed → add `streamlit-authenticator`
- Postgres needed → swap `sqlite3.connect()` for `psycopg2`; interface is identical

---

## Option B — n8n Workflow

**What it is:** A low-code visual workflow engine, self-hostable, with native AI Agent nodes,
HTTP triggers, file watchers, schedulers, and per-workflow credential management.

**When to choose:**
- The firm's IT team prefers visual tools and a point-and-click audit
- The solution needs to integrate with many external systems (email, Slack, cloud storage)
- You want scheduling and retry logic without writing infrastructure code

**Key n8n patterns for accounting:**
- Use AI Agent nodes with structured JSON output schemas — never Code nodes with inline JS
- System prompts must explicitly list the allowed Chart of Accounts — the model must never
  propose an account outside this list
- Use Execute Command nodes to call small Python CLIs for deterministic tasks (parsing, matching)
  rather than embedding logic in Function nodes
- Version the workflow JSON in git — it is your audit trail

**Multi-tenant upgrade path:**
- Single instance: use workflow variables keyed by `tenant_id`
- Multiple instances: one n8n instance per tenant (simplest isolation)
- Phase 3: use n8n's enterprise credential scoping per workspace

---

## Option C — CrewAI + FastAPI

**What it is:** Python-native multi-agent orchestration with explicit agent roles, tasks, and
tool definitions. Exposed via a FastAPI endpoint and packaged as a Docker container.

**When to choose:**
- The team is Python-comfortable and wants strong unit tests on all tools
- You need clean separation between deterministic tool logic and LLM reasoning
- The solution will eventually serve as a backend API for multiple frontends

**Key patterns:**
- All heavy logic (parsing, matching, output writing) lives in `@tool`-decorated Python
  functions that are unit-testable in isolation
- Only three agents should actually call the LLM: Classifier, Reconciliation tiebreaker,
  Recommender — everything else is deterministic
- Package as a container from day 1 with a thin FastAPI wrapper exposing `POST /reconcile`
- Phase 3 upgrade: add a tenant-aware auth middleware and a job queue (Celery or RQ)
  in front of the existing FastAPI entry point — core Crew is unchanged

---

## Option D — n8n + MCP Servers

**What it is:** n8n orchestration with live data extraction via Model Context Protocol (MCP)
servers, enabling agents to query QuickBooks and bank systems in real time rather than
working from exported files.

**When to choose:**
- The firm wants zero-touch file export — data is pulled automatically
- Live QuickBooks integration is required (not just XLSX/CSV exports)
- Phase 2 processes need real-time GL access

**Critical implementation note:**
- **Always use SSE transport for cloud and browser-based MCP integrations.**
- STDIO transport is only appropriate for local developer tooling (runs on the same machine).
- For cloud/hosted n8n, the MCP server must expose an SSE endpoint.

**Relevant MCP servers:**
- `mcp-server-filesystem` — file intake from S3, local, or cloud storage
- `quickbooks-mcp` (community, emerging) — live QuickBooks GL access
- `beancount-mcp` — exposes beancount double-entry primitives to agents
- `mcp-server-postgres` — CoA and tenant config storage
- Box/Drive/S3 MCP connectors — tenant file intake

---

## Platform-Process Fit

| Accounting Process | Recommended Path |
|-------------------|-----------------|
| Bank reconciliation (first automation) | recon-lite → n8n when scaling |
| AP aging report | n8n (file trigger → LLM → Excel output) |
| AR aging report | n8n (same pattern as AP) |
| Expense categorization | n8n with AI Agent node (CoA-constrained) |
| Invoice processing | n8n + OCR MCP or CrewAI + PDF parsing |
| Payroll validation | CrewAI (deterministic rules dominate) |
| Sales-tax review | CrewAI or n8n (rule-heavy, low LLM need) |
| Period close checklist | n8n (orchestration of multiple sub-processes) |
| Full multi-tenant SaaS | CrewAI + FastAPI + Docker + Phase 3 auth layer |
