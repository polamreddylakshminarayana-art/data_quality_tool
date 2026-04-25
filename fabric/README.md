# Data Quality Validation Notebooks (Microsoft Fabric / Synapse)

## 📋 Purpose

The three notebooks work together to ingest configs, automate data quality checks, capture issues, and prepare business-friendly metrics for reporting and monitoring in Microsoft Fabric or Synapse Lakehouse.

They act as the backbone of the data quality process: one notebook
**detects problems**, and the other **summarizes them for reporting**.

------------------------------------------------------------------------
## 🗃️ Notebooks at a Glance
| Notebook | What it does |
|--------|---------|
| `nb_dq_configs_ingestion.ipynb` | Ingests all the Config data from excel sheets to defined schema   |
| `nb_dg_dq_validation_engine.ipynb`| Scans your tables, applies data quality rules, and records every issue it finds |
| `nb_dq_error_counts.ipynb` | Takes those issues and converts them into easy-to-read table-level metrics |




These notebooks are typically scheduled and run through **Azure Data
Factory (ADF)** or **Microsoft Fabric Pipelines**.

------------------------------------------------------------------------

## 🔄 How the process works (simple view)

1.  Upload the Configuration Excel file to LH and run the **Configs Ingestion** → Stores the Configf from excel sheets to tables as per the defined schema and paths
2.  Run the **Validation Engine** → it checks the data and stores raw
    errors.
3.  Run the **Error Counts notebook** → it summarizes those errors for
    dashboards.
4.  Power BI reads the final results for governance reporting.

------------------------------------------------------------------------

# 1️⃣ Notebook: Data Quality Validation Engine

**File:** `nb_dg_dq_validation_engine.ipynb`

### Purpose

This notebook acts as the core validation engine. It checks data in
Lakehouse and Warehouse tables against centrally maintained data quality
rules and captures all violations in a structured way.

### ✅ What it validates (with real examples)

The notebook supports multiple types of checks, including:

| Rule Type | What it checks | Example |
|-----------|----------------|---------|
| **Null validation** | Critical fields must not be null | `customer_id IS NOT NULL` |
| **Accepted values** | Column must belong to a defined list | `status IN ('Active','Inactive')` |
| **Pattern check** | Text must follow a format | Email must contain `@` |
| **Prefix/Suffix** | Values must start or end correctly | Invoice ID starts with `INV_` |
| **Numeric range** | Values must be within bounds | `0 <= discount <= 100` |
| **Uniqueness** | No duplicate keys | `order_id` must be unique |
| **Date consistency** | Logical date relationships | `end_date >= start_date` |
| **Column length** | Text fields not too long | Country code = 2 letters |
| **Data type** | Correct schema | Amount must be numeric |
| **Percentage total** | Totals must equal 100% | Job split = 100% |

### Output

All detected issues are stored in:

    lh_utils.data_governance.data_quality_errors

Each record includes:
- Table name
- Column name
- Primary Key
- Rule defined
- Error Message
- Current existing value
- Timestamp of validation

This table serves as the central record of all data quality failures.


------------------------------------------------------------------------

# 2️⃣ Notebook: Data Quality Error Counts

**File:** `nb_dq_error_counts.ipynb`

### What the notebook does

1.  Reads all validated tables
2.  Calculates total row counts per table
3.  Aggregates validation failures by table and rule type
4.  Joins with governance metadata (owners and stewards)
5.  Writes curated results to Delta Lake

### Purpose

This notebook takes the raw validation results and converts them into
governance-ready metrics that are easier to interpret and act upon by
data owners and stakeholders.

### Final output table

Results are written to:

    lh_utils.data_governance.data_quality_error_counts (Customizable)

This table contains:

-   Source Name
-   Table Name
-   Total number of rows in the table
-   Total number of data quality errors
-   Quality Metric
-   Data owner
-   Data steward
-   Validation date

   ---

### 🔍 Example output (simplified)

| source | table | total_rows | dq_errors | quality_metric | owner | steward | date |
|--------|-------|------------|-----------|------------|-------|---------|------|
| Netsuite | invoices | 120000 | 350 | Completeness | Finance Lead | Data Steward | 2025-02-06 |
| CRM | customers | 85000 | 120 | Validity | Sales Lead | Data Steward | 2025-02-06 |

This table powers your Power BI dashboards. 

------------------------------------------------------------------------

## 📊 How this is used in Power BI

Using `data_quality_error_counts`, you can build dashboards showing:

- 🚩 **Top tables with issues**
- 📉 **Data quality trends over time**
- 🔍 **Tables needing attention from owners**
- ⚠️ **Most frequently failing rules**
- ✅ **Overall health of the data platform**

------------------------------------------------------------------------
## 🚀 Recommended Execution Flow

In a typical production pipeline:

1. Run:  
`nb_dg_dq_validation_engine.ipynb`

2. After it completes, run:  
`nb_dq_error_counts.ipynb`

3. Power BI then consumes:  
`data_quality_error_counts`

------------------------------------------------------------------------

## 🛠️ Technical Requirements

- Microsoft Fabric Lakehouse **or** Synapse Spark  
- Delta Lake enabled  
- Configuration tables in:
```
lh_utils/governance.dq_config
```

------------------------------------------------------------------------

## ✅ Business Benefits

This framework delivers:

- Automated data quality monitoring  
- Centralized issue tracking  
- Clear accountability via data owners  
- Higher trust in reports  
- Less manual data checking  
- Historical visibility of data problems  


