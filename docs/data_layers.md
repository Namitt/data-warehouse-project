# Medallion Layers — Reference

This doc captures how the three layers of the warehouse differ — what each one is for, how data lands in it, and who's meant to use it. It's the mental model I work from when deciding what belongs where: if I'm not sure whether a transformation goes in Silver or Gold, this is the table I check.

## The three layers at a glance

| | Bronze | Silver | Gold |
|---|---|---|---|
| **What it is** | Raw, unprocessed data exactly as it came from the source | Cleaned and standardised data | Business-ready data |
| **Why it exists** | Traceability and debugging — a faithful copy of the source to fall back on | Intermediate layer that prepares data for analysis | The layer reporting and analytics actually consume |
| **Object type** | Tables | Tables | Views |
| **Load method** | Full load (truncate & insert) | Full load (truncate & insert) | None — built on top of Silver at query time |
| **Transformations** | None (as-is) | Cleaning, standardisation, normalisation, derived columns, enrichment | Integration, aggregation, business logic and rules |
| **Data modelling** | None (as-is) | None (as-is) | Star schema, aggregated objects, flat tables |
| **Who uses it** | Data engineers | Data engineers, data analysts | Data analysts, business users |

## How I think about each layer

**Bronze** is deliberately dumb. Nothing gets fixed here — the value is that it's an exact mirror of the source files, so if something looks wrong three layers up, I can always trace it back to what actually arrived. Loading is full truncate-and-insert because there's no history to preserve; each run replaces the last.

**Silver** is where the cleaning work happens but the shape of the data still mirrors the source. Tables here still map roughly one-to-one to source files (`crm_cust_info`, `erp_loc_a101`, etc.), but the values inside are now trustworthy: standardised formats, resolved nulls, normalised categories, a few derived columns. It's the bridge between "what the source gave us" and "what the business wants."

**Gold** is where source structure disappears and business structure takes over. These are views, not tables — they assemble the dimensions and facts on the fly from Silver, applying integration and business logic. This is the only layer most analysts and stakeholders ever touch, so it's modelled for *them* (star schema, clean business names) rather than for traceability.

## Why the load methods differ

Bronze and Silver both physically store data (Tables) and both fully reload each run. Gold doesn't load at all — it's a layer of views, so it always reflects the current Silver data without a separate copy step. That keeps Gold cheap to maintain and guarantees it never drifts out of sync with the cleaned data underneath it.

## Per-layer working pattern

For each layer I follow the same four-step loop, which is also roughly the order I commit things to Git:

1. **Analyse** — understand the data before touching it (interview the source / explore the tables / understand the business objects).
2. **Code** — the actual ingestion, cleansing, or integration logic.
3. **Validate** — completeness and schema checks (Bronze), correctness checks (Silver), integration checks (Gold).
4. **Document & version** — update the relevant docs and commit to Git.

The documentation that comes out of each layer differs: Silver produces the data flow and integration diagrams; Gold produces the data model and data catalog.

## Notes on planning a source system (for reference)

Before ingesting from any new source, these are the questions worth answering up front — captured here so the approach is repeatable beyond this project:

- **Business context & ownership** — who owns the data, what process it supports, where its own documentation lives.
- **Architecture & tech** — how it's stored (SQL Server, Oracle, cloud, etc.) and how you can pull from it (API, file extract, direct DB, streaming).
- **Extract & load** — incremental vs full loads, expected extract size, volume limits, how to avoid hammering the source system, and how authentication/authorisation is handled (tokens, keys, VPN, IP allow-listing).

For this project the answers are simple — flat CSV extracts, full loads, no live connection — but the checklist is the part worth keeping.
