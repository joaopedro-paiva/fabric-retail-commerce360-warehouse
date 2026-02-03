# Runbook — v1.0 (UI steps + exact object names)

## 0) Naming (use exactly)
- Workspace: `ws-retail-commerce360`
- Lakehouse (staging): `lh_retail_staging`
- Warehouse (serving): `wh_retail_serving`
- Dataflow Gen2: `dfg2_olist_v1_ingest`
- Semantic model: `sm_retail_360`

## 1) Create Fabric workspace
- Fabric portal → Workspaces → New workspace
- Name: `ws-retail-commerce360`
- Assign capacity (if prompted / available)

## 2) Create Lakehouse (staging)
- Workspace `ws-retail-commerce360` → New → Lakehouse
- Name: `lh_retail_staging`
- In Lakehouse, create tables by ingestion (next step)

## 3) Create Warehouse (serving)
- Workspace `ws-retail-commerce360` → New → Warehouse
- Name: `wh_retail_serving`

## 4) Create Dataflow Gen2 (ingestion)
- Workspace `ws-retail-commerce360` → New → Dataflow Gen2
- Name: `dfg2_olist_v1_ingest`
- Add 7 sources (CSV files)
  - `olist_orders_dataset.csv`
  - `olist_order_items_dataset.csv`
  - `olist_customers_dataset.csv`
  - `olist_products_dataset.csv`
  - `olist_sellers_dataset.csv`
  - `olist_order_payments_dataset.csv`
  - `product_category_name_translation.csv`
- For each query, set destination
  - Destination type: Lakehouse table
  - Target Lakehouse: `lh_retail_staging`
  - Target table names (exact):
    - `lh_stg_orders`
    - `lh_stg_order_items`
    - `lh_stg_customers`
    - `lh_stg_products`
    - `lh_stg_sellers`
    - `lh_stg_order_payments`
    - `lh_stg_category_translation`
- Save → Publish
- Configure schedule (batch)
  - Set daily (or manual for first run)
- Run now (first load)

## 5) Validate staging load (Lakehouse)
- Open `lh_retail_staging` → Tables
- Confirm all 7 tables exist
- Spot-check: preview rows for each table (schema + sample rows)

## 6) Build serving layer (Warehouse)
- Open `wh_retail_serving` → New SQL query
- Run scripts from repo folder `sql/warehouse/` in this order:
  1) `00_schema.sql`
  2) `01_dims.sql`
  3) `02_fact.sql`
  4) `03_rls.sql`
  5) `04_ops_log.sql` (if separated)

## 7) Validate serving layer (Warehouse)
- Open `wh_retail_serving` → Tables / Views
- Confirm serving objects exist (expected names):
  - `wh_dim_customer`
  - `wh_dim_product`
  - `wh_dim_seller`
  - `wh_dim_date`
  - `wh_fact_order_items`

## 8) Run validation queries
- In `wh_retail_serving`, run scripts from `sql/validation/`:
  - `10_row_counts.sql`
  - `20_join_sanity.sql`
  - `30_duplicate_keys.sql`
  - `40_rls_test.sql`

## 9) Configure RLS test (role-simulated)
- In `wh_retail_serving`, ensure the mapping table exists:
  - `sec_seller_access(role_name, seller_id)`
- Populate with two demo roles (via `sql/validation/40_rls_test.sql`)
- Execute RLS tests:
  - Role A returns rows for Seller A only
  - Role B returns rows for Seller B only
- Capture evidence (screenshots or query outputs) into `docs/evidence/`

## 10) Create semantic model
- In workspace `ws-retail-commerce360` → New → Semantic model (or from Warehouse)
- Name: `sm_retail_360`
- Source: `wh_retail_serving`
- Add tables: dims + fact
- Create 3–5 measures (keep generic, no field guessing beyond what exists)

## 11) Apply sensitivity label
- Apply label to `sm_retail_360` (and serving objects if applicable)
- Capture evidence into `docs/evidence/`

## 12) Monitor runs
- Monitoring hub → confirm:
  - `dfg2_olist_v1_ingest` refresh status
  - Warehouse query history for build scripts
- Optional: write a row to ops log per run (if implemented)

## 13) Completion criteria (v1.0)
- Dataflow refresh succeeds
- 7 staging tables exist and are queryable
- Serving objects exist and return results
- RLS test proves filtered results for two roles
- Semantic model is created and queries work
- Evidence captured in `docs/evidence/`

