# Naming Conventions

These are the naming standards I follow across every object in the warehouse — schemas, tables, views, columns, and stored procedures. The goal is that anyone (including future me) can look at an object name and immediately know which layer it lives in, what it represents, and where its data came from.

## Core rules

These apply everywhere, no exceptions:

- **snake_case** — lowercase words separated by underscores (`customer_info`, not `CustomerInfo` or `customerInfo`). It's consistent and avoids case-sensitivity headaches.
- **English** for all object names.
- **No SQL reserved words** as names — avoids quoting and parser ambiguity.

The naming pattern deliberately changes between layers, and that change is meaningful: it tells you how far through the pipeline the data has travelled.

---

## Tables

### Bronze and Silver — keep the source's shape

In Bronze and Silver, table names stay tied to the source system and keep the original entity name. The point of these layers is traceability — you should be able to look at any Silver table and know exactly which raw source file it maps back to.

**Pattern:** `<source_system>_<entity>`

- `<source_system>` — `crm` or `erp`
- `<entity>` — the original table/file name from that system, unchanged

Example: `crm_customer_info` → customer data straight from the CRM.

Bronze and Silver share the same naming because they describe the same entities; the difference between them is the *state* of the data (raw vs cleansed), not its identity.

### Gold — switch to business language

Gold is where naming stops following the source and starts following the *business*. Stakeholders querying this layer don't care that something came from `crm` — they care whether it's a customer or a sale. So names become business-aligned and prefixed by their modelling role.

**Pattern:** `<type>_<entity>`

- `<type>` — the table's role in the star schema
- `<entity>` — a clean business name (`customers`, `products`, `sales`)

| Prefix | Meaning | Examples |
|---|---|---|
| `dim_` | Dimension table | `dim_customers`, `dim_products` |
| `fact_` | Fact table | `fact_sales` |
| `report_` | Pre-aggregated reporting table | `report_sales_monthly` |

Examples: `dim_customers`, `fact_sales`.

---

## Columns

### Surrogate keys

Surrogate keys (the warehouse-generated keys that join facts to dimensions) always end in `_key`, named after the table they identify.

**Pattern:** `<entity>_key` → e.g. `customer_key` is the surrogate key of `dim_customers`.

This makes joins self-documenting: seeing `customer_key` in a fact table tells you instantly which dimension it points to.

### Technical / metadata columns

Any column the warehouse generates itself (not present in the source data) is prefixed `dwh_`. This cleanly separates real business data from pipeline bookkeeping, so no one mistakes a load timestamp for a business field.

**Pattern:** `dwh_<name>` → e.g. `dwh_load_date` records when a row was loaded.

---

## Stored procedures

Load procedures are named after the layer they populate.

**Pattern:** `load_<layer>` → `load_bronze`, `load_silver`, `load_gold`.
