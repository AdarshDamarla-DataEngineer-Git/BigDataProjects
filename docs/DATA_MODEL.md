# Data model and lineage

## Source relationships

```mermaid
erDiagram
    CUSTOMERS ||--o{ ORDERS : customer_id
    ORDERS ||--o{ ORDER_ITEMS : order_id
    ORDERS ||--o{ PAYMENTS : order_id
    ORDERS ||--o{ REVIEWS : order_id
    PRODUCTS ||--o{ ORDER_ITEMS : product_id
    SELLERS ||--o{ ORDER_ITEMS : seller_id
    CATEGORY_TRANSLATION ||--o{ PRODUCTS : product_category_name

    CUSTOMERS { string customer_id string customer_unique_id string customer_state }
    ORDERS { string order_id string customer_id string order_status timestamp order_purchase_timestamp }
    ORDER_ITEMS { string order_id int order_item_id string product_id string seller_id double price double freight_value }
    PAYMENTS { string order_id int payment_sequential string payment_type int payment_installments double payment_value }
    REVIEWS { string review_id string order_id int review_score }
    PRODUCTS { string product_id string product_category_name double product_weight_g }
    SELLERS { string seller_id string seller_city string seller_state }
    CATEGORY_TRANSLATION { string product_category_name string product_category_name_english }
```

## Final dataset grain

The transformation starts with orders, then joins potentially multiple payments and multiple order items. The practical grain is therefore close to:

```text
one row per order × payment sequence × order item
```

This means order-level measures can be repeated across line/payment combinations. Aggregations should use the appropriate key and avoid summing repeated values without deduplication.

## Derived delivery columns

| Column | Expression | Interpretation |
|---|---|---|
| `actual_delivery_time` | `datediff(delivered_customer_date, purchase_date)` | Actual delivery duration in days |
| `estimated_delivery_time` | `datediff(estimated_delivery_date, purchase_date)` | Promised delivery duration in days |
| `delay` | Actual duration greater than estimated duration | Late-delivery indicator |
| `Delay time` | Actual duration minus estimated duration | Days late or early |

## Current join participation

| Dataset | Read | Cleaned | Joined into `final_df` |
|---|---:|---:|---:|
| Customers | Yes | Yes | Yes |
| Orders | Yes | Yes | Yes |
| Order items | Yes | Yes | Yes |
| Payments | Yes | Yes | Yes |
| Products | Yes | Yes | Yes |
| Sellers | Yes | Yes | Yes |
| Reviews | Yes | Yes | No |
| Geolocation | Yes | Yes | No |
| Category translation | MongoDB | Not through shared function | Yes |

## Suggested analytical marts

- `fact_order_items`: item-level revenue, freight, seller, product, order, and customer keys
- `fact_payments`: payment value, type, installments, and order key
- `fact_reviews`: review score and response timing
- `dim_customer`: unique customer geography
- `dim_product`: product attributes and translated category
- `dim_seller`: seller geography
- `dim_date`: purchase, approval, shipping, delivery, and review dates

Separating these grains prevents payment and item multiplication and creates clearer BI semantics.

## Example query

```sql
SELECT
  product_category_name_english,
  COUNT(DISTINCT order_id) AS orders,
  SUM(CAST(price AS FLOAT)) AS item_revenue,
  AVG(CAST("Delay time" AS FLOAT)) AS avg_delay_days
FROM gold.finaltable
GROUP BY product_category_name_english
ORDER BY item_revenue DESC;
```

## Data-quality recommendations

- Assert non-null and unique primary keys at their source grains.
- Validate order status against accepted values.
- Validate non-negative prices, freight, and payments.
- Check delivery timestamps are chronologically consistent.
- Monitor join match rates for customers, products, sellers, and categories.
- Reconcile item revenue, payment value, and order counts separately by grain.
