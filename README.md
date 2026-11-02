# CareFlow: Clinical Pathway Process Mining

Starter repository for the **CareFlow: Clinical Pathway Process Mining** project. It uses **synthetic/mock hospital event logs** to discover clinical pathways, calculate delays and bottlenecks between hospital activities, and check conformance against standard patient care guidelines.

> **Important Safety Note**: This project uses **100% fictional, synthetic sample data**. Never upload or commit real patient data, medical records, or identifiable personal health information (PHI).

---

## Quick Start on Windows (Step-by-Step)

You can run the entire Phase 1 pipeline directly from PowerShell or Command Prompt.

### Step 1: Generate Synthetic Patient Event Logs
```powershell
python src/generate_event_log.py
```
* **What it does**: Creates 200 synthetic patient journeys (180 standard clinical pathways and 20 edge-case/non-conformant journeys such as emergency bypasses, walkouts, and review loops).
* **Output**: `data/mock_hospital_event_log.csv`

---

### Step 2: Discover Clinical Process Maps (DFG & Petri Nets)
```powershell
python src/process_discovery.py
```
* **What it does**: Computes the **Directly-Follows Graph (DFG)** showing transition frequencies and average durations between hospital activities.
* **Outputs**:
  * `outputs/dfg_transitions.csv` — Full table of activity-to-activity transitions with counts and average durations.
  * `outputs/hospital_dfg_matrix.png` — High-resolution visual transition matrix heatmap (generated using Matplotlib).
* **PM4Py Support**:
  * If `pm4py` is installed on Python 3.10+, it will also discover and export an Inductive Petri Net and PM4Py DFG.
  * If PM4Py or Graphviz is not installed, the built-in Pandas + Matplotlib fallback automatically generates your process map and CSVs without crashing.

---

### Step 3: Run Bottleneck & Delay Analysis
```powershell
python src/bottleneck_analysis.py
```
* **What it does**: Calculates the time spent transitioning between activities (min, 25th percentile, median, mean, 75th percentile, max, and total delay hours). Also segments bottlenecks by hospital department.
* **Outputs**:
  * `outputs/average_transition_times.csv` — Ranked bottlenecks across the entire hospital.
  * `outputs/bottlenecks_by_department.csv` — Bottlenecks broken down by department (e.g., Emergency, Cardiology, Orthopedics).

---

### Step 4: Run Pathway Conformance Checking
```powershell
python src/conformance_check.py
```
* **What it does**: Compares each patient journey against standard reference rules (must start at Registration, undergo Triage and Doctor Consultation, and conclude with formal Discharge). Flags violations like emergency bypasses and incomplete walkouts.
* **Output**: `outputs/conformance_results.csv` (contains case ID, cycle time in hours, activity sequence, compliance status, violation category, and diagnostic reasons).

---

### Step 5: Run Data Modeling & Build Power BI Analytical Marts (Phase 2)
```powershell
python src/build_data_marts.py
```
* **What it does**: Runs the automated data modeling pipeline locally with **zero cloud setup, zero credentials, and zero cost**. It emulates the dbt transformation flow (`stg_event_log` -> `fct_case_summary`, `fct_transitions`), runs automated data quality checks, and generates clean analytical tables ready for Power BI.
* **Outputs**:
  * `outputs/fct_case_summary.csv` — Primary fact table for total length of stay, case completion status, and patient demographics.
  * `outputs/fct_transitions.csv` — Granular transition table detailing delay duration between every touchpoint.
  * `outputs/dim_department_summary.csv` — Departmental performance benchmarks and throughput KPIs.

---

## Google BigQuery & dbt (Optional Cloud Target)

The project includes production-ready dbt models in the `dbt/` folder for teams connecting to Google Cloud BigQuery:

* **Staging Model**: [`dbt/models/staging/stg_event_log.sql`](dbt/models/staging/stg_event_log.sql) — Cleans types, trims strings, and filters nulls.
* **Case Summary Fact**: [`dbt/models/marts/fct_case_summary.sql`](dbt/models/marts/fct_case_summary.sql) — Aggregates case-level metrics using BigQuery SQL window functions (`ROW_NUMBER()`, `TIMESTAMP_DIFF`).
* **Transitions Fact**: [`dbt/models/marts/fct_transitions.sql`](dbt/models/marts/fct_transitions.sql) — Calculates activity-to-activity latencies using BigQuery `LEAD()` and `TIMESTAMP_DIFF()`.
* **Project Configuration**: [`dbt/dbt_project.yml`](dbt/dbt_project.yml) and schema tests in [`dbt/models/marts/schema.yml`](dbt/models/marts/schema.yml).

### Differences between BigQuery Mode and Local Mode
* **Local Mode (`src/build_data_marts.py`)**: Uses Python and Pandas directly on local synthetic CSVs. Requires **no Google account, no billing, and no credentials**. Runs in under 2 seconds.
* **Cloud Mode (`dbt-bigquery`)**: Pushes SQL queries to Google Cloud BigQuery. Requires a Google Cloud Project ID, BigQuery dataset, and service account key or OAuth authentication.

---

## Summary of Generated Output Files

| File | Type | Generated By | Description |
| :--- | :--- | :--- | :--- |
| `data/mock_hospital_event_log.csv` | CSV | `generate_event_log.py` | Raw synthetic event log with timestamps and patient cohorts |
| `outputs/hospital_dfg_matrix.png` | Image | `process_discovery.py` | Visual Directly-Follows process transition heatmap |
| `outputs/dfg_transitions.csv` | CSV | `process_discovery.py` | Transition pairs with frequency and average duration |
| `outputs/average_transition_times.csv` | CSV | `bottleneck_analysis.py` | Ranked transition delays with detailed statistics |
| `outputs/bottlenecks_by_department.csv` | CSV | `bottleneck_analysis.py` | Delays segmented by department |
| `outputs/conformance_results.csv` | CSV | `conformance_check.py` | Per-case pathway compliance and diagnostic flags |
| `outputs/fct_case_summary.csv` | CSV | `build_data_marts.py` | Star-schema case fact table for Power BI (LOS, cohort, completion) |
| `outputs/fct_transitions.csv` | CSV | `build_data_marts.py` | Star-schema transition fact table for Power BI (step-by-step wait) |
| `outputs/dim_department_summary.csv` | CSV | `build_data_marts.py` | Department benchmark dimension table for Power BI |

---

## Python Environment & PM4Py Setup (Optional Upgrade)

Your project is built to run immediately with your existing Python setup. If you wish to install the full `pm4py` library for advanced Petri Net rendering:

1. **Install Python 3.10 or newer**: Download from [python.org](https://www.python.org/downloads/) (ensure **"Add python.exe to PATH"** is checked).
2. **Install Graphviz for Windows**: Download the installer from [graphviz.org/download](https://graphviz.org/download/) (ensure **"Add Graphviz to system PATH"** is selected).
3. **Install dependencies**:
   ```powershell
   pip install -r requirements.txt
   ```

---

## Repository Structure

```text
CareFlow_Clinical_Pathway_Process_Mining/
├── data/
│   └── mock_hospital_event_log.csv       # Synthetic event log
├── dbt/
│   ├── dbt_project.yml                   # Root dbt project configuration
│   ├── profiles.yml.example              # BigQuery dbt profile template
│   └── models/
│       ├── staging/
│       │   ├── sources.yml               # Source table definition
│       │   └── stg_event_log.sql         # SQL staging model
│       └── marts/
│           ├── schema.yml                # Mart model tests and documentation
│           ├── fct_case_summary.sql      # Case-level fact model
│           └── fct_transitions.sql       # Transition-level fact model
├── docs/
│   ├── data_dictionary.md                # Schema descriptions
│   ├── project_plan.md                   # Project milestones
│   └── power_bi_dashboard_guide.md       # Complete Power BI setup, DAX & layout guide
├── outputs/
│   ├── average_transition_times.csv      # Bottleneck statistics
│   ├── bottlenecks_by_department.csv     # Department-level delays
│   ├── conformance_results.csv           # Conformance audit
│   ├── dfg_transitions.csv               # Directly-Follows Graph table
│   ├── dim_department_summary.csv        # Department KPI benchmarks
│   ├── fct_case_summary.csv              # Power BI case fact table
│   ├── fct_transitions.csv               # Power BI transition fact table
│   └── hospital_dfg_matrix.png           # Visual process heatmap
├── sql/
│   └── create_bigquery_table.sql         # BigQuery table DDL
├── src/
│   ├── generate_event_log.py             # Synthetic data generator
│   ├── process_discovery.py              # DFG & Petri Net process discovery
│   ├── bottleneck_analysis.py            # Bottleneck calculation
│   ├── conformance_check.py              # Clinical pathway conformance check
│   └── build_data_marts.py               # Local data modeling & QA runner
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Power BI Dashboard & Visualizations (Phase 3)

A complete implementation guide is provided in [`docs/power_bi_dashboard_guide.md`](docs/power_bi_dashboard_guide.md).

### 1. Data Modeling Architecture (Star Schema)
To prevent double-counting cases when analyzing transitions:
* **Merge in Power Query (Recommended)**: Merge `fct_case_summary.csv` with `conformance_results.csv` on `case_id` into a single unified table.
* **Relationship**: Connect `fct_case_summary[case_id]` to `fct_transitions[case_id]` with a **One-to-Many (1:*)** single-direction relationship.

### 2. Key DAX Measures
* **Total Cases**: `DISTINCTCOUNT(fct_case_summary[case_id])`
* **Average Stay (LOS)**: `AVERAGE(fct_case_summary[cycle_time_hours])`
* **Pathway Conformance Rate**: `DIVIDE([Conformant Cases], [Total Cases], 0)` (Formatted as `%`)
* **Completion Rate**: `DIVIDE([Completed Cases], [Total Cases], 0)` (Formatted as `%`)
* **Average Transition Latency**: `AVERAGE(fct_transitions[transition_minutes])`

### 3. Two-Page Dashboard Structure
* **Page 1: Clinical Pathway Executive Overview**:
  * 5 Top KPI Cards (`Total Cases`, `Conformance Rate %`, `Completion Rate %`, `Avg Stay`, `Avg Step Delay`).
  * Conformance Donut Chart (90% Conformant vs 10% Non-Conformant).
  * Violation Category Bar Chart (Walkouts, Emergency Bypasses, Loops).
  * Department Comparison Chart (using secondary Y-axis for stay hours vs. completion percentage).
  * Slicers for *Department*, *Patient Age Group*, and *Doctor ID*.
* **Page 2: Process Bottlenecks & Transition Analysis**:
  * High-Delay Transitions KPI card.
  * Ranked Bottleneck Horizontal Bar Chart (`Admission -> Treatment`, `Consultation -> Lab`, etc.).
  * Directly-Follows Flow Grid (Matrix with heat map gradient).
  * Detailed Patient Journey Audit Table.

### 4. Mentor Presentation Talking Points
* **Pipeline Integration**: Demonstrates an end-to-end data pipeline from raw synthetic event logs to dbt star-schema modeling and Power BI reporting.
* **Process Discovery**: Highlights how process mining uncovers actual clinical activity transitions rather than assumed workflows.
* **Operational Impact**: Pinpoints exact transition bottlenecks (e.g., inpatient admission to treatment averaging 129.2 minutes) and identifies safety drop-offs (11 triage walkouts) for targeted hospital improvement.

---

## Next Project Phase

* **Phase 4: Final Presentation & Review** — Packaging end-to-end code execution and final review documentation.

