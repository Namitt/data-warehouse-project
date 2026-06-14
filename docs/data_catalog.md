# Gold Layer — Data Catalog

The Gold layer is the business-facing layer of the warehouse. It's modelled as a **star schema**: one central fact table (`fact_sales`) surrounded by two dimension tables (`dim_customers`, `dim_products`). Everything here is built on top of the cleansed Silver layer and is designed to be queried directly for reporting — no further joins back to raw source data needed.

## Model at a glance

```
        dim_customers
              │
              │ customer_key
              ▼
        fact_sales ──── product_key ────► dim_products
```

- `fact_sales` holds one row per **order line item**.
- Each line links to a customer and a product through surrogate keys (`customer_key`, `product_key`), not the original source IDs.
- The surrogate keys are generated in the Gold layer, which keeps the model insulated from changes or inconsistencies in the source systems (ERP and CRM).

---

## dim_customers

**Grain:** one row per unique customer.
**Source:** built by combining customer records from the CRM (core identity) and ERP (demographic/geographic enrichment) in the Silver layer.

| Column | Type | Description |
|---|---|---|
| customer_key | INT | Surrogate key — the primary key of this dimension, generated in Gold. Used by `fact_sales` to join. |
| customer_id | INT | The customer's numeric ID carried over from the source system. |
| customer_number | NVARCHAR(50) | Alphanumeric customer reference used for tracking. |
| first_name | NVARCHAR(50) | Customer's first name. |
| last_name | NVARCHAR(50) | Customer's last/family name. |
| country | NVARCHAR(50) | Country of residence (e.g. 'Australia'). Sourced from ERP location data. |
| marital_status | NVARCHAR(50) | Standardised marital status (e.g. 'Married', 'Single'). |
| gender | NVARCHAR(50) | Gender (e.g. 'Male', 'Female', 'n/a' where unknown). Reconciled between CRM and ERP in Silver. |
| birthdate | DATE | Date of birth (YYYY-MM-DD). |
| create_date | DATE | When the customer record was first created in the source system. |

**Note:** where the same customer appears in both source systems with conflicting gender values, the Silver layer resolves to the CRM value as the system of record; `n/a` is used only when neither source provides one.

---

## dim_products

**Grain:** one row per current product. Only the latest version of each product is kept — historical/expired product records are filtered out, since the project scope doesn't require historisation.
**Source:** product master from ERP, enriched with category data.

| Column | Type | Description |
|---|---|---|
| product_key | INT | Surrogate key — primary key of this dimension, generated in Gold. |
| product_id | INT | Product ID from the source system. |
| product_number | NVARCHAR(50) | Structured product code used for categorisation/inventory. |
| product_name | NVARCHAR(50) | Descriptive name including type, colour, size. |
| category_id | NVARCHAR(50) | Identifier linking the product to its high-level category. |
| category | NVARCHAR(50) | Top-level grouping (e.g. Bikes, Components). |
| subcategory | NVARCHAR(50) | More specific classification within the category. |
| maintenance_required | NVARCHAR(50) | Whether the product needs maintenance ('Yes'/'No'). |
| cost | INT | Base cost of the product in whole currency units. |
| product_line | NVARCHAR(50) | Product line/series (e.g. Road, Mountain). |
| start_date | DATE | When the product became available. |

---

## fact_sales

**Grain:** one row per **sales order line item**. A single order with three different products produces three rows here.
**Source:** sales transactions from CRM, with foreign keys resolved against the two dimensions during the Gold build.

| Column | Type | Description |
|---|---|---|
| order_number | NVARCHAR(50) | Alphanumeric order identifier (e.g. 'SO54496'). Repeats across line items of the same order. |
| product_key | INT | FK → `dim_products.product_key`. |
| customer_key | INT | FK → `dim_customers.customer_key`. |
| order_date | DATE | When the order was placed. |
| shipping_date | DATE | When the order shipped. |
| due_date | DATE | When payment was due. |
| sales_amount | INT | Total value of the line item in whole currency units. |
| quantity | INT | Units of the product ordered on this line. |
| price | INT | Price per unit in whole currency units. |

**Consistency check applied in Silver:** `sales_amount` should equal `quantity × price`. Rows where the source data violated this (nulls, negatives, or mismatches) were corrected or recalculated before reaching the Gold layer.

---

## How to read this model

- To analyse **sales over time**, group `fact_sales` by `order_date`.
- To analyse **by customer attribute** (country, gender, age band), join `fact_sales` → `dim_customers` on `customer_key`.
- To analyse **product performance**, join `fact_sales` → `dim_products` on `product_key`.

Because it's a clean star schema, every analytical question is one or two joins away from the fact table.
