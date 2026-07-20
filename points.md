# Resume — Aetna Work Experience

## CVS Health / Aetna | Senior Data Engineer — Health Plan Advisory Platform
**Remote | Jun 2026 – Present**

Sole Data Engineer on a personalized health plan selection engine for commercial Aetna members, supporting an enterprise pilot ahead of a full-scale national launch.

---

### Clinical Data Modeling & Pipeline Architecture

- Designed **canonical healthcare data models** (single, authoritative schema) consolidating medical claims, pharmacy claims, plan benefit structures, and provider network data into a unified simulation-ready schema
- Built end-to-end ETL pipelines in Python and SQL transforming raw clinical claims into structured inputs for a multi-plan benefit simulation engine — waterfall logic covering deductible, coinsurance, and out-of-pocket maximum stages across 4 network scenarios
- Implemented configurable plan parameter logic supporting embedded, aggregated, and copay-first benefit designs, ensuring all cost-sharing thresholds are externally configurable for plan year flexibility

---

### Schema Mapping & Healthcare Data Transformation

- Performed detailed schema mapping across medical and pharmacy claim sources, resolving carve-in/carve-out pharmacy routing divergence and network ID mismatches requiring rate table lookups
- Standardized claim-line-level cost-sharing attributes — allowed amounts, member liability, OOP accumulators — into a canonical format consumed by downstream simulation and AI explainability layers
- Mapped FHIR-aligned clinical concepts (service dates, diagnosis categories, provider specialty) to proprietary fields for member utilizer segmentation across 9 behavioral sub-segments

---

### Data Quality & Validation

- Conducted backward validation of simulation accuracy against historical actuals, isolating root causes of accumulator drift — a data corruption issue in upstream benefit tables accounted for the majority of simulation error; drove resolution initiative
- Built automated data quality checks for eligibility filtering: enrollment tenure thresholds, household-level aggregation, utilization detection, and sensitivity classification across 5 member tiers
- Identified and quantified impact of a pharmacy data gap where a carve-out Rx source was not flowing into the analytics warehouse — documented business impact and escalated as a pipeline blocker to stakeholders

---

### Platform & Performance

- Developed and optimized large-scale SQL pipelines on Google Cloud / BigQuery, structuring query logic with performance-conscious partitioning and filter strategies for high-volume clinical data
- Pre-computed simulation payloads at annual snapshot, stored results in Google Cloud Spanner for sub-millisecond serving via API — fully decoupled heavy compute from member-facing request latency
- Authored mapping specifications, data flow documentation, and field-level lineage, establishing data engineering standards across the workstream

---

### Cross-Functional Collaboration

- Partnered with enterprise architects, data scientists, and product managers to resolve schema dependencies and align on simulation input contracts
- Contributed to design decisions on tiebreaker logic, family plan conflict surfacing, and live plan availability validation at render time

---

### Key Technologies

Python · SQL · BigQuery · Google Cloud Platform · Google Cloud Spanner · Healthcare Claims (Medical & Rx) · FHIR · ETL · Data Modeling · Schema Mapping · Data Quality · LLM Integration

---

## Terminology Notes

**Canonical** — the single, authoritative, agreed-upon standard form. In data engineering, a canonical data model is the one master schema that all source systems map to, so downstream consumers always get consistent field names, types, and formats regardless of the source.
