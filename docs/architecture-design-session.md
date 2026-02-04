# Architecture Design Session (ADS) — Retail Commerce 360 on Microsoft Fabric (v1.0)

## 0. Document control
- Version: v1.0
- Status: design-first (execution deferred to Fabric trial capacity)
- Audience: engineering leadership, senior engineers, technical recruiters
- Scenario type: hypothetical customer engagement (constraints-driven)
- Customer standard: Azure + Microsoft 365 standardized environment
- Roles: Data Architect (author), Customer Sponsor (simulated), BI Lead (simulated), Platform Lead (simulated)

## 1. Executive summary
- Goal: Warehouse-first Fabric reference architecture aligned to DP-700, optimized for fast delivery under realistic constraints.
- Pattern: Dataflow Gen2 → Lakehouse staging → Warehouse serving → Semantic model.
- Data slice (loaded): orders, order_items, customers, products, sellers, order_payments, category_translation (7 CSVs).
- Optional-only: geolocation, order_reviews (documented; not ingested in v1.0).
- Serving contract: 4 dimensions + 1 fact; fact grain = order-item; seller-level governance anchor.
- Governance: seller-level RLS demonstrated via role-simulation; production identity path documented.
- Ops: monitoring evidence + minimal run log contract + deterministic rerun strategy.
- Out of scope (v1.0): streaming/CDC, CI/CD, deep performance tuning, enterprise identity hardening, exhaustive modeling.
- Success criteria: staging loaded, serving queryable, RLS provable, validations pass, evidence captured during trial.

## 2. Session 0 — Discovery (Architect Q&A)

### Business outcomes
- Q: What outcomes must be delivered in the first iteration (4–8 weeks)?
- A: A reliable Commerce 360 view with consistent KPIs and reduced manual reporting effort.
- Q: Who are the primary consumers?
- A: BI analysts and business stakeholders; limited data science scope in the short term.

### Data and latency
- Q: What are the source systems and refresh expectations?
- A: Batch CSV extracts; daily refresh is acceptable for v1.0.
- Q: What latency is acceptable?
- A: T+1 day is sufficient; near-real-time is not required.
- Q: What domains are in scope for the first delivery?
- A: Orders, items, customers, products, sellers, and payments. Reviews/geo can be added later.

### Platform standards (Azure/M365)
- Q: Are you standardized on a cloud/vendor stack?
- A: Yes. Azure and Microsoft 365 are default; preference is managed services and minimal integration overhead.
- Q: Any procurement/tooling constraints?
- A: Prefer staying within Microsoft ecosystem for v1.0.

### Team maturity and operating model
- Q: What is the skill profile (SQL, Spark, Python, DevOps)?
- A: Strong SQL and Power BI; limited Spark; limited DevOps capacity for multi-tool orchestration.
- Q: Who will operate the platform?
- A: A small central platform team; BI teams consume curated data via a governed interface.

### Governance and security
- Q: What access boundaries are required?
- A: Seller/partner-level separation is required for some reporting scenarios.
- Q: What is the governance expectation in v1.0?
- A: Basic labels and access control now; hardening and auditing later.
- Q: Is PII (Personally Identifiable Information) the primary focus?
- A: Not for v1.0; keep scope minimal and avoid compliance claims.

### Operations and reliability
- Q: What reliability and alerting is expected?
- A: Basic monitoring and failure visibility; no 24/7 on-call for v1.0.
- Q: Is CI/CD required for the first iteration?
- A: Manual is acceptable for v1.0; document a path to CI/CD.

### Cost and performance
- Q: What cost posture do you want?
- A: Predictable; optimize for BI workloads; avoid complex always-on compute.
- Q: Performance expectations?
- A: Interactive dashboards; start minimal and iterate.

### Definition of done (v1.0)
- A: Automated batch ingestion, curated query layer, one semantic model, seller-level access control, and basic monitoring evidence.

## 3. Constraints distilled
- Delivery: time-to-value in weeks; minimal moving parts in v1.0.
- Platform: Microsoft-first tenant; prioritize integrated operating model.
- Skills: SQL-first delivery; minimize Spark dependency in v1.0.
- Governance: seller boundary required now; identity hardening later.
- Ops: basic monitoring + rerun discipline; no 24/7 on-call in v1.0.
- Budget (assumption): platform spend ≤ USD 2,000/month for v1.0 workloads (excludes existing M365/Power BI commitments).

## 4. Architect proposal (target architecture)
- Proposal: Warehouse-first Fabric implementation optimized for governed BI consumption.
- Data flow: Dataflow Gen2 (batch) → Lakehouse staging → Warehouse serving → Semantic model.
- Serving principle: one curated SQL surface for BI consumers.

### Architecture diagram (Mermaid)
    flowchart LR
      S[CSV batch files] --> DF[Dataflow Gen2]
      DF --> LH[(Lakehouse: staging tables)]
      LH --> WH[(Warehouse: serving layer)]
      WH --> SM[Semantic model]
      WH --- RLS[Seller-level RLS]
      DF --- MON[Monitoring hub]
      WH --- OPS[ops.run_log]

- Physical names:
  - Workspace `ws-retail-commerce360`
  - Lakehouse `lh_retail_staging`
  - Warehouse `wh_retail_serving` (schemas: `serving`, `security`, `ops`)
  - Dataflow `dfg2_olist_v1_ingest`
  - Semantic model `sm_retail_360`
- Alternatives considered (contextual):
  - Databricks: better for Spark-heavy engineering/ML; not selected due to SQL-first + minimal ops surface constraint.
  - Snowflake: better for DWH-first multi-cloud posture; not selected due to integrated Fabric operating model constraint.
- Decision review cadence: reassess platform fit after v1.0 evidence and usage patterns (cost + performance).

## 5. Contracts (source, staging, serving)
- Source: batch CSV extracts (daily T+1 acceptable).
- In-scope inputs (loaded): 7 CSVs listed in `docs/source-mapping.md`.
- Optional-only inputs: geolocation and reviews (document-only in v1.0).
- Staging (Lakehouse): 1 file → 1 table; no business transforms; mapping lives in `docs/source-mapping.md`.
- Serving (Warehouse, minimal):
  - `serving.wh_dim_customer` (key: `customer_id`)
  - `serving.wh_dim_seller` (key: `seller_id`)
  - `serving.wh_dim_product` (key: `product_id` + optional translation)
  - `serving.wh_dim_date` (key: `date`)
  - `serving.wh_fact_order_items` (grain: `(order_id, order_item_id)`; includes `seller_id`)
- Validation gates (minimum): row counts, duplicate keys, join coverage, RLS sanity.

## 6. Governance and security
- Access model: least privilege; separate build access from consume access; serving is the consumption surface.
- RLS: seller boundary anchored at `serving.wh_fact_order_items` using `seller_id`.
- Mapping concept: `security.sec_seller_access(principal_or_role, seller_id)`; deterministic and testable.
- v1.0 test approach: role-simulation (session context) to prove policy behavior; evolve to Entra principals/groups + auditing.
- Classification: apply sensitivity label to `sm_retail_360`; document intent; avoid compliance claims.

## 7. Operations and observability
- Monitoring (v1.0): Dataflow refresh status/duration; Warehouse build + validation query history.
- Run log contract: `ops.run_log(run_id, component, status, started_at, ended_at, rows_in, rows_out, watermark_date, error_message)`.
- Rerun strategy: full refresh overwrite → rebuild serving → rerun validations; block promotion if gates fail.

## 8. Decision log (constraints → decisions)
- D1 SQL-first + many consumers → Warehouse-first serving surface.
- D2 Fast delivery + low orchestration capacity → Dataflow Gen2 ingestion in v1.0.
- D3 Reduce semantic drift → 1:1 staging; mapping controlled in `docs/source-mapping.md`.
- D4 Seller boundary required → order-item fact grain; RLS anchored at fact.
- D5 Governance now, hardening later → role-simulated RLS now; Entra evolution documented.
- D6 Budget + minimal tooling → single-platform operating model in v1.0.
- D7 Basic reliability → monitoring evidence + minimal run log + deterministic rerun.
- D8 Dataset flexibility → portability plan keeps serving contract stable.

## 9. Risks and mitigations
- Schema drift → version pinning + mapping update + rerun gates.
- Key integrity issues → duplicate-key checks + join coverage checks block promotion.
- Lakehouse-to-Warehouse access constraints → documented fallback (stage directly in Warehouse / enforce shortcuts).
- RLS mis-scope → anchor at fact grain + A/B tests + evidence.
- Semantic drift → define measures only after field validation during trial.
- Cost creep → budget assumption + scoped workloads + usage discipline.

## 10. Evolution plan (v1.1+)
- Identity hardening: Entra principals/groups + audit expectations.
- Incremental loads: partitioning/incremental refresh after baseline stability.
- Automated quality gates: freshness + null thresholds + schema drift detection.
- CI/CD: repeatable promotion strategy (dev → test → prod).
- Domain expansion: ingest reviews/geolocation; add marts/aggregations as justified.
- Performance: tune based on measured usage patterns.

## 11. Evidence checklist (to capture during Fabric trial)
- Workspace/Lakehouse/Warehouse created with exact names (evidence artifacts).
- Dataflow refresh history shows at least one successful run.
- Staging tables exist + row counts captured.
- Serving objects exist + row counts captured.
- Validation outputs saved (duplicates, join coverage, counts).
- RLS proof saved (A vs B returns different row counts on fact).
- Semantic model created + label applied + usable query/visual evidence.

## 12. What this project is / isn’t

- This is: a constraints-driven reference architecture + minimal v1.0 blueprint for a Warehouse-first Fabric implementation.
- This is: a reproducible structure with explicit contracts, governance intent, ops posture, and an evidence plan to validate during Fabric trial.
- This is not: a full production implementation (no enterprise identity hardening, CI/CD, deep tuning, or exhaustive marts in v1.0).
- This is not: a claim that Fabric is universally “best”; the platform choice is justified by the simulated customer constraints.
- This is not: a compliance attestation; security and labeling are documented and require end-to-end enforcement + evidence.
