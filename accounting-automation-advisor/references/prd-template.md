# PRD Template — Accounting Automation Agent

Use this template for every new accounting process automation. Replace all `{placeholder}`
values. Sections 1–3 are written for non-technical business stakeholders. Sections 4–9
are for the development team.

File naming convention: `Phase{N}_{ProcessName}_PRD.md`
Example: `Phase1_Bank_Reconciliation_PRD.md`, `Phase2_AP_Aging_PRD.md`

---

# Phase {N} PRD — Multi-Tenant Accounting Agent Suite
## {Process Name} Agent

**Version:** 1.0
**Date:** {YYYY-MM-DD}
**Owner:** {Name or team}
**Status:** Draft for review

---

## 1. Purpose & Vision

{1–3 sentences describing the business problem this solves. Write for a CFO or firm partner,
not a developer. Mention how it fits into the broader Multi-Tenant Accounting Agent Suite.}

Example: "Build an AI-agentic Bank Reconciliation service that ingests bank statements and
General Ledger exports, classifies transactions against the firm's Chart of Accounts, and
produces a reconciled workbook that sums to $0.00. Phase 1 is the first building block of
a broader Multi-Tenant Accounting Agent Suite."

---

## 2. Goals & Non-Goals

**Goals**
- {Goal 1 — what the system will do}
- {Goal 2}
- {Goal 3}
- Minimum-maintenance architecture with a clear upgrade path to multi-tenant.

**Non-Goals (Phase {N})**
- {What this phase explicitly will NOT do}
- Full multi-tenant isolation (addressed in Phase 3)
- {Any out-of-scope advisory, UI polish, or adjacent processes}

---

## 3. Users & Personas

- **{Primary User Role}** — {what they do with the system, e.g., "uploads files, reviews results, resolves flagged items"}
- **{Approver Role}** — {e.g., "Controller / Firm Partner — reviews exceptions, signs off on period close"}
- **Firm IT / Admin (Phase 3)** — manages tenants, configuration, credentials

---

## 4. Scope — Phase {N}

### 4.1 Inputs
- {Input 1: file type, format, source system}
- {Input 2}
- Period metadata: tenant ID, period start/end, base currency

### 4.2 Processing
1. **Ingestion & normalisation** — parse heterogeneous inputs into canonical schema:
   `{date, description, amount, debit_credit, balance, source, raw_ref, tenant_id, period}`
2. **{Core processing step}** — {description}
3. **{Additional step}**
4. **Exception detection** — identify items that could not be auto-processed
5. **Recommendation generation** — for each exception, produce a proposed action

### 4.3 Outputs
- **{Output 1}** — {description, sheet structure, file name pattern}
- **{Output 2}**
- **JSON run log** — audit trail with inputs, decisions, confidence scores, model version

### 4.4 Rules & Governance
- **LLM temperature:** 0.3 across all classification, matching, and recommendation steps
- **Data integrity:** the agent is prohibited from fabricating amounts, dates, or accounts.
  Every output value must be traceable to a source row (`raw_ref`)
- **Auditability:** every run produces a JSON run log with inputs, decisions, confidence
  scores, and model version
- **Human-in-the-loop:** any result below the confidence threshold is routed to the
  exceptions sheet rather than the processed output

---

## 5. Functional Requirements

| ID | Requirement |
|----|-------------|
| FR-1 | {Requirement 1} |
| FR-2 | {Requirement 2} |
| FR-3 | {Requirement 3} |
| FR-4 | Emit a structured run log (JSON) for audit. |
| FR-5 | Expose a tenant identifier on every artifact (forward-compatible with Phase 3). |

---

## 6. Non-Functional Requirements

- **Reliability:** idempotent runs; same inputs → same outputs
- **Performance:** ≤ {N} minutes for {X} combined records
- **Security:** tenant data isolated at the storage layer; no cross-tenant prompt context
- **Observability:** run logs, token usage, and processing statistics per run
- **Maintainability:** low-code (n8n) or thin Python; upgrade path documented
- **Portability:** same container/workflow must run on-prem or cloud (Phase 3)

---

## 7. Forward Compatibility — Phases 2 & 3

- **Phase 2** processes ({list adjacent processes}) will plug into the same ingestion, CoA,
  and run-log services. Each process is an independent agent/workflow but shares the canonical
  schema and tenant model.
- **Phase 3** adds tenant onboarding, per-tenant secrets, row-level isolation, usage metering,
  and either multi-tenant SaaS or packaged on-prem deployment via containers. Phase {N} already
  uses tenant-scoped paths and a pluggable CoA store to make this a configuration change rather
  than a rewrite.

---

## 8. Success Metrics

- ≥ {X}% of {records} auto-processed on clean inputs
- 100% audit traceability (every output row linked to a source row)
- {Process-specific accuracy metric, e.g., "$0.00 control total on ≥ 98% of runs"}
- ≤ {N} minutes median end-to-end run time

---

## 9. Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| {Risk 1} | {Mitigation 1} |
| LLM hallucination of accounts or amounts | Constrained decoding against CoA list; schema validation; reject runs that violate invariants |
| Multi-tenant data leakage | Per-tenant namespaces; no shared vector store in Phase 1 |
| {Risk N} | {Mitigation N} |

---

## Agentic Solution — Phase {N}

### A. Open-Source Building Blocks

Before writing custom code, evaluate these libraries:

- **`pandas` + `openpyxl`** — CSV/XLSX ingestion and output (always use)
- **`beancount`** — double-entry canonical schema and `bean-check` reconciliation
- **`rapidfuzz`** — fuzzy string matching for descriptions
- **`ofxparse` / `ofxtools`** — OFX/QFX bank feed parsing
- **`pydantic`** — schema validation and structured agent output
- **`python-quickbooks`** — QuickBooks Online API (Phase 2+)
- **`streamlit`** — zero-frontend web UI for rapid deployment
- **MCP servers** — filesystem, QuickBooks, Postgres for live data access

### B. Agent Roles (adapt per process)

1. **Intake Agent** — validates files, detects format, assigns tenant + period
2. **Normalizer Agent** — converts inputs to canonical schema
3. **Classifier Agent** — assigns CoA accounts (T=0.3, constrained to CoA list)
4. **{Core Process} Agent** — deterministic processing (matching, calculation, extraction)
5. **Exception & Recommendation Agent** — explains anomalies, proposes actions (T=0.3)
6. **Reporter Agent** — writes output workbooks and JSON run log

### C. Platform Recommendation

{Choose from: recon-lite / n8n / CrewAI + FastAPI / n8n + MCP — and explain why}

See `references/platform-selection.md` for the full decision matrix.

### D. Recommended Claude Skills / Connectors

- **Claude Skills:** `xlsx` for workbook generation, `pdf` for statement PDFs
- **MCP connectors:** {list relevant connectors for this process}
- **Libraries:** {list the specific libraries selected for this process}

---

**End of Phase {N} PRD — ready for review.**
