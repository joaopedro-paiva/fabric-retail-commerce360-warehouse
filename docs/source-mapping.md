# Source Mapping — Retail Commerce 360 (v1.0)

## Purpose
- Single reference for: source files → staging tables → serving objects → join keys.
- Any schema drift: update this file + rerun validation gates.

## In-scope files (loaded in v1.0)
| Source file | Staging table (Lakehouse) | Primary key / grain | Used by serving objects |
|---|---|---|---|
| `olist_orders_dataset.csv` | `lh_stg_orders` | `order_id` | `serving.wh_fact_order_items`, `serving.wh_dim_date` |
| `olist_order_items_dataset.csv` | `lh_stg_order_items` | `(order_id, order_item_id)` | `serving.wh_fact_order_items` |
| `olist_customers_dataset.csv` | `lh_stg_customers` | `customer_id` | `serving.wh_dim_customer` |
| `olist_products_dataset.csv` | `lh_stg_products` | `product_id` | `serving.wh_dim_product` |
| `olist_sellers_dataset.csv` | `lh_stg_sellers` | `seller_id` | `serving.wh_dim_seller`, `serving.wh_fact_order_items` |
| `olist_order_payments_dataset.csv` | `lh_stg_order_payments` | `(order_id, payment_sequential)` | `serving.wh_fact_order_items` (enrichment) |
| `product_category_name_translation.csv` | `lh_stg_category_translation` | `product_category_name` | `serving.wh_dim_product` (enrichment) |

## Optional-only files (documented, not loaded in v1.0)
| Source file | Planned use (v1.1+) | Rationale |
|---|---|---|
| `olist_geolocation_dataset.csv` | geo enrichment and location marts | adds regional analysis without changing core grain |
| `olist_order_reviews_dataset.csv` | customer satisfaction mart | adds reviews without changing core serving contract |

## Join keys (v1.0)
- `lh_stg_orders.customer_id = lh_stg_customers.customer_id`
- `lh_stg_order_items.order_id = lh_stg_orders.order_id`
- `lh_stg_order_payments.order_id = lh_stg_orders.order_id`
- `lh_stg_order_items.product_id = lh_stg_products.product_id`
- `lh_stg_order_items.seller_id = lh_stg_sellers.seller_id`
- `lh_stg_products.product_category_name = lh_stg_category_translation.product_category_name`

## Notes
- Column-level schemas are intentionally not duplicated here; v1.0 uses the pinned dataset version as the source of truth.

