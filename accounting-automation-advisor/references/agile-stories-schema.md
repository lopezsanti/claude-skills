# Agile Stories — Schema, Sizing, and Epic Checklist

## JSON Schema

Save stories as `{ProcessName}_Agile_Stories.json` in the `docs/` folder of the repository.

```json
{
  "project": "{Process Name} Agent",
  "phase": "Phase {N}",
  "suite": "Multi-Tenant Accounting Agent Suite",
  "version": "1.0",
  "date": "YYYY-MM-DD",
  "status": "Ready for Development",
  "total_epics": 9,
  "total_stories": 37,
  "epics": [
    {
      "id": "EPIC-1",
      "title": "Epic Title",
      "description": "What this epic covers and why it matters.",
      "stories": [
        {
          "id": "US-01",
          "title": "Short imperative title",
          "story": "As a [Staff Accountant | Controller | Firm IT/Admin], I want [capability] so that [business value].",
          "acceptance_criteria": [
            "Observable, testable condition 1.",
            "Observable, testable condition 2.",
            "Observable, testable condition 3."
          ],
          "functional_requirements": ["FR-1", "FR-2"]
        }
      ]
    }
  ]
}
```

---

## Epic Checklist (9 standard epics)

Every accounting automation Phase 1 should include all nine epics. Adapt titles and story
content to the specific process. Each epic typically has 4–6 stories.

### EPIC-1: File Intake & Upload
Covers receiving, validating, and staging all input files.

Standard stories to include:
- Upload primary input file(s) (e.g., bank statements, vendor invoices, payroll export)
- Upload secondary input file (e.g., GL export, approved vendor list)
- Specify period and tenant metadata
- Validate uploaded files before processing (format, required columns, size limits)
- Upload or update tenant configuration (e.g., Chart of Accounts, vendor list, rules)

### EPIC-2: Data Normalisation
Converts heterogeneous source formats into the canonical schema.

Standard stories:
- Auto-detect source file layout (header mapping, rules-first + LLM fallback at T=0.3)
- Normalise primary input to canonical schema (with raw_ref traceability)
- Normalise secondary input to canonical schema
- Merge multiple input files into a consolidated view (with duplicate detection)

### EPIC-3: Classification / Categorization
Assigns Chart of Accounts codes, categories, or labels to each record.

Standard stories:
- Classify records against the tenant's CoA or category list (T=0.3, constrained)
- Prevent hallucinated assignments (validate every LLM output against the allowed list)
- Split records into meaningful groups (e.g., debits vs. credits, expense vs. income)

### EPIC-4: Core Processing (adapt title per process)
The heart of the automation — matching, calculation, extraction, or validation.

Examples by process:
- **Reconciliation:** exact match → subset-sum → fuzzy description cascade
- **AP Aging:** bucket invoices by age (0–30, 31–60, 61–90, 90+ days)
- **Expense categorization:** rule-based categorization + LLM fallback
- **Invoice processing:** extract fields (vendor, amount, date, PO number) from PDF/XLSX

Standard stories:
- Implement primary processing method (highest-confidence, deterministic)
- Implement secondary processing method (handles edge cases)
- Enforce confidence threshold for auto-acceptance vs. exception routing
- Handle {process-specific edge case, e.g., outstanding checks, partial payments}

### EPIC-5: Exception Detection & Recommendations
Identifies items that could not be auto-processed and explains each one.

Standard stories:
- Isolate unprocessed items type A (e.g., GL-only, unpaid invoices, uncategorized)
- Isolate unprocessed items type B (e.g., bank-only, duplicate invoices)
- Generate AI remediation recommendations (T=0.3, constrained to input data)
- Prevent hallucinated recommendations (validate against CoA and input raw_ref)

### EPIC-6: Output & Reporting
Produces the deliverable workbooks and ensures full traceability.

Standard stories:
- Generate primary output workbook (Sheet 1: processed, Sheet 2: exceptions, Sheet 3: recommendations)
- Generate secondary output workbook if applicable (e.g., breakdown by category)
- Ensure full output traceability via raw_ref (every cell traceable to source row)
- Deliver outputs to tenant output folder `/{tenant_id}/{period}/output/`

### EPIC-7: Audit, Observability & Idempotency
Every run must leave a complete, machine-readable audit trail.

Standard stories:
- Emit structured JSON run log (run_id, tenant_id, period, input hashes, model version,
  temperature, decisions, confidence scores, run status)
- Track token usage and processing statistics per run
- Guarantee idempotent runs (same inputs → same outputs; re-runs overwrite, not duplicate)

### EPIC-8: Tenant Isolation & Forward Compatibility
Ensures Phase 3 multi-tenant upgrade is a configuration change, not a redesign.

Standard stories:
- Stamp all artifacts with tenant identifier (file names, folder paths, JSON logs)
- Isolate tenant data at the storage layer (no cross-tenant path traversal, no shared prompts)
- Support pluggable additional processes (shared services: canonical schema, CoA store, run log)
- Run in containerised environment (Docker image, environment variables for all config)

### EPIC-9: Performance & Reliability
Validates the system meets its SLA and handles real-world edge cases.

Standard stories:
- Complete processing within {N} minutes for {X} records (performance test with known dataset)
- Achieve ≥ {X}% auto-processing rate on clean inputs (acceptance test with known outcome)
- Handle {process-specific edge case 1} gracefully (e.g., outstanding checks, partial payments)
- Handle {process-specific edge case 2} gracefully (e.g., heterogeneous bank layouts, encoding errors)

---

## Story Sizing for Solo / 2-Person Teams

Story points at this scale:
- **1 point** — < 2 hours: single-function implementation, config change, minor UI tweak
- **2 points** — 2–4 hours: a complete module method + unit test
- **3 points** — half a day: a new agent role or a core algorithm
- **5 points** — 1 day: a full epic's worth of deterministic logic
- **8 points** — split this story

Sprint cadence for 1–2 developers: 1-week sprints, target 20–30 points per sprint per developer.
A 37-story, 9-epic Phase 1 at average 2.5 points per story = ~90 points = 3–4 sprints for
one developer working full-time (6–8 weeks elapsed with buffer).

---

## Persona Vocabulary

Use exactly these personas in story "As a..." clauses for consistency:

- **Staff Accountant** — day-to-day operator, uploads files, reviews results
- **Controller / Firm Partner** — approver, reviews exceptions, signs off on close
- **Firm IT / Admin** — configures tenants, manages CoA, handles infrastructure
- **Developer** — internal tooling and CI/CD (use sparingly in Phase 1 stories)

---

## Acceptance Criteria Writing Rules

Good acceptance criteria are:
1. **Observable** — you can see or measure the outcome without running the code
2. **Testable** — a QA person could write a test case from it
3. **Specific** — includes thresholds, formats, file names, error messages

Avoid:
- "The system should work correctly" (not testable)
- "The system should be fast" (not specific)
- "The agent processes the file" (not observable)

Good example:
> "Every output row includes a `raw_ref` column in the format `{filename}#{row_number}`
>  (e.g., `bank_statement.xlsx#42`). No output cell contains a value that cannot be traced
>  to a raw_ref in the input files."

---

## Template Story Bank

Copy and adapt these for common recurring stories:

**Tenant-scoped output delivery:**
```
As a Staff Accountant, I want both output workbooks to be automatically saved to
/{tenant_id}/{period}/output/ so that I can retrieve results without navigating a
complex file system.
AC:
- Both workbooks are written to /{tenant_id}/{period}/output/ on run completion.
- File names include tenant_id, period, and run timestamp for uniqueness.
- Delivery confirmation (file paths and sizes) is returned to the caller.
```

**Idempotency:**
```
As a Firm IT/Admin, I want the system to produce identical outputs when given the same
inputs so that I can safely reprocess files after a failure without inconsistent results.
AC:
- Given the same input files and metadata, two consecutive runs produce byte-identical output.
- LLM calls use a fixed random seed or deterministic prompt hashing.
- Re-runs overwrite the previous run's output files rather than creating duplicates.
```

**CoA-constrained classification:**
```
As a Controller/Firm Partner, I want the system to be technically prohibited from assigning
an account that does not exist in the tenant's CoA so that I can trust every classified
transaction is valid and auditable.
AC:
- Classifier output is validated against the CoA list before acceptance.
- Any LLM response proposing an account not in the CoA is rejected and flagged.
- Rejected classifications are routed to the unprocessed sheet with reason "CoA validation failed".
```
