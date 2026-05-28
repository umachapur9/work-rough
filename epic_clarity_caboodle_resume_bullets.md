# Resume Bullets — Epic Clarity & Caboodle / Healthcare Data Engineering

> Metrics verified from production pipelines (May 2026) — use scale numbers only, not internal table/dataset names

---

## Healthcare Data Engineering

- Engineered production ETL pipelines on GCP BigQuery supporting **4.6M+ TDC program members** across Aetna and Caremark (2020–2026), integrating Epic Clarity/Caboodle, ODS, and non-Epic sources into the ActiveHealth Data Platform for HIPAA-compliant PMPM billing

- Built and maintained care gap data pipelines processing **46M+ care gap records** (3.1 GB) across 197 gap categories spanning diabetes and hypertension programs, with source-to-target mapping, validation, and change data capture

- Led data model cutover for provider attribute restructuring (flat → nested `providers[]` array) across the CET bundle pipeline, coordinating schema changes across downstream consumers with zero pipeline downtime

- Partnered with Epic SMEs, product, and analytics stakeholders to translate TDC program engagement rules into validated, billable data pipelines supporting PMPM reporting across 459 enterprise clients/plans

---

## Enterprise Analytics / GCP / BigQuery

- Delivered end-to-end billing data pipeline with **2.9M+ delivery records** feeding PMPM invoicing across **459 enterprise clients/plans**, enabling client I&I reporting for the Transform Diabetes Care program

- Unified **16.3M+ A1C lab results** across two payer source systems into a single validated lab results dataset, supporting clinical metric calculation for diabetes management interventions

- Engineered multi-dataset BigQuery architecture with IAM-based access tiering for application vs. shared/consumption layers, encrypted at rest via AES-256 (KMS-managed)

- Applied BigQuery streaming buffer constraints and DDL discipline for Pub/Sub-fed production tables, preventing data loss during schema evolution on high-throughput pipelines

- Implemented dynamic caching and performance optimization for analytics reporting layer, tuning TTLCache sizing (2048) and query patterns to reduce latency for concurrent report consumers

---

## Consuming Clarity / Caboodle Data

- Integrated Epic AHM Clarity/Caboodle daily batch extracts as upstream sources for TDC historical engagement data, supporting **4.6M+ enrolled members** tracked across 6 years of program history

- Profiled and validated Epic Clarity-sourced encounter and gap data across **197 care gap categories**, developing rejection and reconciliation logic ensuring data integrity for PMPM-billable engagement metrics

- Supported cross-system data integration combining Epic Clarity/Caboodle, ODS real-time feeds, and non-Epic sources (WellDoc, Telephony, CCaaS) into a unified engagement metrics view for TDC PMPM billing

---

## Key Metrics Reference (internal use only — do not include table names externally)

| What | Scale | Context |
|---|---|---|
| TDC enrolled members | 4.6M+ | 2020–2026, Aetna + Caremark |
| Care gap records | 46M+ / 3.1 GB | DM + HTN programs |
| PMPM billing delivery records | 2.9M+ | Enterprise billing pipeline |
| A1C lab results unified | 16.3M+ | Across two payer systems |
| Enterprise clients/plans | 459 | TDC program |
| Care gap categories | 197 | DM + HTN gap types |

---

## Job Description Gap Note

This role requires Epic-**native** ETL development (DMCs, Caboodle ETL, Cogito components, Clarity ETL jobs).
CVS/Aetna's enterprise pattern is **consuming** Epic data into GCP — not building inside Epic.
Frame experience as: *"deep expertise consuming and validating Clarity/Caboodle-sourced pipelines"* — accurate and defensible.
If asked about Epic-native tooling, position it as a ramp item with strong adjacent foundation.


----------------------------------------------------------------------------------------

# Resume Bullets — Epic Clarity & Caboodle / Healthcare Data Engineering

> Metrics verified from production pipelines (May 2026) — use scale numbers only, not internal table/dataset names

---

## Healthcare Data Engineering

- Engineered production ETL pipelines on GCP BigQuery supporting **4.6M+ TDC program members** across Aetna and Caremark (2020–2026), integrating Epic Clarity/Caboodle, ODS, and non-Epic sources into the ActiveHealth Data Platform for HIPAA-compliant PMPM billing

- Built and maintained care gap data pipelines processing **46M+ care gap records** (3.1 GB) across 197 gap categories spanning diabetes and hypertension programs, with source-to-target mapping, validation, and change data capture

- Led data model cutover for provider attribute restructuring (flat → nested `providers[]` array) across the CET bundle pipeline, coordinating schema changes across downstream consumers with zero pipeline downtime

- Partnered with Epic SMEs, product, and analytics stakeholders to translate TDC program engagement rules into validated, billable data pipelines supporting PMPM reporting across 459 enterprise clients/plans

---

## Enterprise Analytics / GCP / BigQuery

- Delivered end-to-end billing data pipeline with **2.9M+ delivery records** feeding PMPM invoicing across **459 enterprise clients/plans**, enabling client I&I reporting for the Transform Diabetes Care program

- Unified **16.3M+ A1C lab results** across Aetna and Caremark source systems into a single validated lab results dataset (`valid_a1c_lab_results_aetna_cmk_union`), supporting clinical metric calculation for diabetes management interventions

- Engineered multi-dataset BigQuery architecture (`cpa_de_ent_tdc_prod`, `cpa_de_ent_tdc_share_prod`) with IAM-based access tiering for application vs. shared/consumption layers, encrypted at rest via AES-256 (KMS-managed)

- Applied BigQuery streaming buffer constraints and DDL discipline for Pub/Sub-fed production tables, preventing data loss during schema evolution on high-throughput pipelines

- Implemented dynamic caching and performance optimization for analytics reporting layer, tuning TTLCache sizing (2048) and query patterns to reduce latency for concurrent report consumers

---

## Consuming Clarity / Caboodle Data

- Integrated Epic AHM Clarity/Caboodle daily batch extracts as upstream sources for TDC historical engagement data flowing into the ActiveHealth Data Platform (`anbc-prod`), supporting **4.6M+ enrolled members** tracked across 6 years of program history

- Profiled and validated Epic Clarity-sourced encounter and gap data across **197 care gap categories**, developing rejection and reconciliation logic ensuring data integrity for PMPM-billable engagement metrics

- Supported cross-system data integration combining Epic Clarity/Caboodle, ODS real-time feeds, and non-Epic sources (WellDoc, Telephony, CCaaS) into a unified engagement metrics view for TDC PMPM billing

---

## Key Metrics Reference

| Table | Rows | Size | Context |
|---|---|---|---|
| `tdc_welcomed_members` | 4,579,810 | 359 MB | TDC enrolled members 2020–2026 |
| `tdc_gaps_long` | 46,001,720 | 3.1 GB | Care gap records (DM + HTN) |
| `tdc_billing_dlvry` | 2,934,842 | 258 MB | PMPM billing delivery records |
| `valid_a1c_lab_results_aetna_cmk_union` | 16,344,121 | 468 MB | A1C lab results (Aetna + Caremark) |
| `hdp_eds_clients_prod` | 459 clients | — | Enterprise clients/plans served |
| `care_gap_info` | 197 gap categories | — | DM + HTN gap configuration |

---

## Job Description Gap Note

This role requires Epic-**native** ETL development (DMCs, Caboodle ETL, Cogito components, Clarity ETL jobs).
CVS/Aetna's enterprise pattern is **consuming** Epic data into GCP — not building inside Epic.
Frame experience as: *"deep expertise consuming and validating Clarity/Caboodle-sourced pipelines"* — accurate and defensible.
If asked about Epic-native tooling, position it as a ramp item with strong adjacent foundation.
