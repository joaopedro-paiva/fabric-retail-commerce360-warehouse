# Retail Commerce 360 on Fabric (Warehouse-first)

## What this is
- A Microsoft Fabric reference implementation aligned to DP-700.
- Pattern: Dataflow Gen2 → Lakehouse (staging) → Warehouse (serving) → Semantic model.
- Scenario: retail commerce analytics using the Olist public e-commerce dataset.

## Architecture (v1.0)
- Workspace: `ws-retail-commerce360`
- Lakehouse (staging): `lh_retail_staging`
- Warehouse (serving): `wh_retail_serving`
- Dataflow Gen2 (ingestion): `dfg2_olist_v1_ingest`
- Semantic model: `sm_retail_360`
- Serving contract (minimal): `wh_dim_customer`, `wh_dim_product`, `wh_dim_seller`, `wh_dim_date`, `wh_fact_order_items`
- Governance: seller-level RLS (role-simulated demo pattern)
- Ops: Monitoring hub + ops run log table

## Data
- Source: Olist Brazilian E-Commerce public dataset (7 CSVs in-scope; 2 CSVs optional).
- In-scope (loaded in v1.0): orders, order_items, customers, products, sellers, order_payments, category_translation.
- Optional (kept, not loaded in v1.0): geolocation, order_reviews.

## How to run
- Follow `docs/runbook.md`.
- Capture screenshots/query outputs under `docs/evidence/`.

## Repository layout
- `docs/architecture-design-session.md` — Architecture Design Session (decisions, trade-offs, evolution)
- `docs/source-mapping.md` — CSV → staging → serving mapping + join keys
- `docs/runbook.md` — UI checklist to run v1.0 end-to-end
- `sql/warehouse/` — serving layer + RLS + ops log build scripts
- `sql/validation/` — validation + RLS test scripts
- `docs/evidence/` — execution evidence (screenshots, query results)

## DP-700 coverage focus
- Ingestion (Dataflow Gen2), Lakehouse, Warehouse, semantic model basics
- Security (RLS pattern + sensitivity labeling)
- Monitoring/operations (Monitoring hub + minimal run logging)

## Out of scope (v1.0)
- Streaming/CDC, CI/CD, full identity integration, deep performance tuning, advanced dimensional modeling

## Notes
- The architecture is dataset-swappable (see evolution section in `docs/ads.md`).
- Company and dataset origin are not the point; the patterns are geography-agnostic.

References: :contentReference[oaicite:0]{index=0}, :contentReference[oaicite:1]{index=1}
