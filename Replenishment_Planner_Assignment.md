**REPLENISHMENT PLANNER — TAKE-HOME ASSIGNMENT**

You are given a dataset of SKU-level inventory and supply data for a fulfillment center.
Build a replenishment planner that computes purchase order (PO) suggestions for each row
and populates the output columns described below.

AI use is expected. Focus is on our ability to iterate fast by using this tool as an enabler. Attach the AI Chat link with public view access enabled.

**ASSUME THE SUGGESTION GENERATION DATE (effectively the start_date for calculation purposes) AS 2026-03-16**


1. DATASET

File: assignment_data.csv (1,419 rows, 36 columns)

The data is pre-cleaned. Numeric columns have no comma formatting. JSON columns
(inventory_breakup, open_po_details) are valid JSON strings that you will need to parse.


2. COLUMN REFERENCE

Identity
  facility_id          Fulfillment center identifier
  facility_name        Fulfillment center name
  jpin                 Product identifier (SKU)
  title                Product name
  category_name        Product category
  manufacturername     Manufacturer
  pvname               Product variant / sub-category

Vendor
  vendor_id            Vendor identifier
  vendor_name          Vendor name
  vendor_lead_time     Days from PO to delivery
  current_vendor_mov   Vendor minimum order value
  minimum_order_criteria  How MOV is enforced (VALUE or CASES)

Demand
  max_drr              Daily run rate (units/day)
  sales_band           Demand classification (Band A / B / C)

Inventory norms
  inv_norm             Target days of inventory to maintain
  safety_stock         Minimum buffer stock (days)

Current position
  current_inventory    On-hand inventory (units)
  inventory_breakup    JSON: breakdown by sellable, contingency, pending_putaway, under_transfer
  deadweight           weight of the product

Space
  max_allocated_space  Maximum units that can be stored for this SKU
  cases_allocated      max_allocated_space in case_size multiples
  space_value          Value of allocated space

Ordering
  case_size            Units per case (for rounding)

Open purchase orders
  open_po_details      JSON: details of existing open POs (promise dates, quantities, values)
  orderedquantity      Total units already on order
  open_po_value        Total value of open POs
  open_po_cases        Total cases on order
  earliest_promise_date  Earliest expected delivery date of open POs

Commercials
  mrp                  Maximum retail price
  cp                   Cost price (purchase price per unit)

Output columns (you populate these)
  final_suggestion          Units to order
  final_cases_suggestion    Cases to order
  final_value               Purchase value of the suggested order
  final_days_of_inventory   Projected days of inventory after the suggested order
  final_tonnage             Weight/tonnage of the suggested order
  mov_check                 MOV compliance flag


3. WHAT TO BUILD

Using Python:

  a) Compute optimal inventory to order for each SKU to satisfy inventory norms
     while maintaining safety stock and preventing out-of-stock risk.

  b) Account for existing on-hand inventory and any open purchase orders in the pipeline.

  c) Determine how many days of inventory the current and projected position covers.

  d) Round order quantities to whole cases and ensure orders do not exceed available space.

  e) Compute the projected days of inventory after the suggested order is placed.

  f) Populate all five output columns for every row.

Required constraints:
  - Case rounding: orders must be in whole cases (use case_size).
  - Space cap: suggested quantity must not exceed max_allocated_space.
  - MOV enforcement using current_vendor_mov and minimum_order_criteria.

Bonus constraints (not required, but valued):
  - Implementation of Inventory Health objectives, sales_band-based prioritization, or other logic you identify.


4. SQL REQUIREMENT

Load the base data into a SQL database (SQLite, DuckDB, or Postgres — your choice).

Starter schema for SQLite:

  CREATE TABLE replenishment_data (
      facility_id TEXT,
      category_name TEXT,
      manufacturername TEXT,
      facility_name TEXT,
      jpin TEXT,
      title TEXT,
      pvname TEXT,
      vendor_lead_time INTEGER,
      inv_norm INTEGER,
      safety_stock INTEGER,
      vendor_id TEXT,
      vendor_name TEXT,
      current_vendor_mov REAL,
      minimum_order_criteria TEXT,
      max_allocated_space INTEGER,
      case_size INTEGER,
      cases_allocated REAL,
      space_value REAL,
      current_inventory INTEGER,
      inventory_breakup TEXT,
      max_drr INTEGER,
      deadweight REAL,
      earliest_promise_date TEXT,
      open_po_details TEXT,
      orderedquantity INTEGER,
      open_po_value REAL,
      open_po_cases REAL,
      final_suggestion INTEGER,
      final_days_of_inventory INTEGER,
      final_cases_suggestion INTEGER,
      final_value INTEGER,
      final_tonnage REAL,
      mov_check REAL,
      mrp REAL,
      cp REAL,
      sales_band TEXT
  );

  -- To import:
  -- sqlite3 replenishment.db
  -- .mode csv
  -- .import assignment_data.csv replenishment_data

Write and include at least two SQL queries (bonus for visualisation included):

  Query 1: Vendor-level summary showing total suggested value, total cases and
           count of SKUs and other relevant metrics that would be useful for a consumer of this data as a dashboard.

  Query 2: Top 10 riskiest SKUs — lowest days of inventory where max_drr > 0,
           showing facility, vendor, title, and current days of inventory.


5. DELIVERABLES

Submit a repository or zipped folder containing:

  - A Jupyter / Google Co-Lab Notebook OR a Python script with your planner logic and results.
  - SQL file(s) or embedded queries (schema + the two required queries, minimum).
  - A short README covering:
      - How to run your code.
      - Key assumptions you made and why.
      - Where AI helped and what you verified manually.
      - What you would improve with more time.
  - Any AI Bot Chat Link.

Bonus (not required):
  - A small test suite for your core computations.
  - A Streamlit or similar UI for interacting with the planner.
  - Additional SQL queries or dashboards beyond the two required.


6. EVALUATION

  Logical correctness (40%)
    Suggestions are coherent, constraints are respected, edge cases handled.

  Data engineering (25%)
    JSON parsing, null handling, type correctness, reproducible pipeline.

  SQL competence (20%)
    Useful queries with proper joins, grouping, and aggregation.

  Code quality and documentation (15%)
    Readable code, clear assumptions, honest documentation of AI usage.
