# ASG Airlines - End to End Data Engineering Case Study

A pipeline that ingests the ASG Airlines operational workbook, runs data quality checks,
cleans and standardises the data, protects passenger personal information, and models the
result into an analytical star schema with business KPIs. A Power BI report is built on
top of the modelled tables.

The whole pipeline is one notebook: `Angel_Blessy.ipynb`, covering ingestion (Task 1),
transformation and cleaning (Task 2), and modelling and KPIs (Task 3). The Power BI report
(Task 4), `Dashboard.pbix`, is built by hand from the notebook outputs.

## Repository structure

```
.
|-- README.md                         this file
|-- Angel_Blessy.ipynb                the full pipeline, Tasks 1 to 3
|-- Angel Blessy Document.docx        written answers to each brief requirement
|-- Dashboard.pbix                    the Power BI report, Task 4
|-- SUBMISSION_NOTES.md               short reviewer summary
|-- Airlines Use Case/
|   |-- UseCase - Airlines.xlsx       source workbook (4 sheets)
|   `-- Use_Case_Airlines Intructions.docx   the original brief
|-- data/                             all generated, safe to delete and rebuild
|   |-- bronze/                        raw rows read as text, with data quality flags
|   |   `-- flights|passengers|bookings|payments.parquet
|   |-- quarantine/                    rows rejected at ingestion, with a reason
|   |   `-- flights|passengers|bookings|payments.parquet
|   |-- silver/                        cleaned, standardised tables, PII hashed
|   |   `-- flights|passengers|bookings|payments.parquet
|   |-- gold/                          star schema + KPI tables, CSV, for Power BI
|   |   |-- dim_airline|dim_airport|dim_route|dim_date|dim_passenger.csv
|   |   |-- fact_flights|fact_bookings|fact_payments.csv
|   |   `-- kpi_*.csv                  16 pre-computed KPI tables (report cross-check)
|   |-- dq_report.csv                  every data quality issue and its row count
|   `-- cleaning_summary.csv           before and after row counts
|-- diagrams/                          architecture, data flow, data model (PNG)
|   `-- src/                           Mermaid sources + regenerate instructions
`-- docs/
    `-- powerbi/theme.json             Power BI theme
```

## Layers

| Layer | Folder | Contents |
|---|---|---|
| Landing | `Airlines Use Case/` | the source workbook, never modified |
| Bronze | `data/bronze/` | every row read as text, no business logic, data quality flags attached |
| Quarantine | `data/quarantine/` | rows with a missing key or an exact duplicate, kept with a reason |
| Silver | `data/silver/` | cleaned: standardised values, repaired airline, real timestamps, overnight handled, duration recomputed, duplicates removed, missing values imputed, personal data hashed |
| Gold | `data/gold/` | star schema (dimensions + facts) and KPI tables, written as CSV |

## Prerequisites

- Python 3.10 or newer
- Packages: `pandas`, `pyarrow`, `openpyxl`, and `jupyter` (or run the notebook in VS Code)
- Power BI Desktop (Windows) for the report

```
pip install pandas pyarrow openpyxl jupyter
```

## How to run

### 1. Run the pipeline

Open `Angel_Blessy.ipynb` and run all cells, top to bottom. Or from a shell:

```
jupyter nbconvert --to notebook --execute --inplace Angel_Blessy.ipynb
```

It reads `Airlines Use Case/UseCase - Airlines.xlsx` and rewrites `data/bronze`,
`data/quarantine`, `data/silver` and `data/gold`, plus `data/dq_report.csv` and
`data/cleaning_summary.csv`. The run is idempotent - it overwrites its own output, so
re-running is safe.

Expected result on the supplied data:

| Table | Source rows | Quarantined | Silver rows |
|---|---|---|---|
| flights | 1,020 | 15 (exact duplicate rows) | 1,004 (1 more duplicate key removed in cleaning) |
| passengers | 1,039 | 0 | 1,000 (39 duplicate ids removed) |
| bookings | 1,000 | 0 | 1,000 |
| payments | 1,000 | 0 | 1,000 |

### 2. The Power BI report

`Dashboard.pbix` loads the eight model tables from `data/gold`, builds the relationships,
adds the DAX measures, applies `docs/powerbi/theme.json`, and has four pages: Duration
Analysis, Route Performance, Airline Trends, and Delay and Anomaly.

Cross-check the visuals against the `data/gold/kpi_*.csv` files - with no slicer applied
they should match exactly.

## Documentation

- `Angel Blessy Document.docx` - written answers to each requirement in the brief:
  code implementation, dataset documentation, and the evaluation-criteria responses.
- `SUBMISSION_NOTES.md` - a short summary of the whole submission.
- `diagrams/` - architecture, data flow and data model diagrams (PNG, with Mermaid sources).
- Every code cell in the notebook has a markdown cell above it describing what it does.

## Notes on the data

- No scheduled times exist in the source, so "delay" is measured against the median
  duration of the same route: over 30 minutes above it is delayed, over 90 is severe.
- The airline is derived from the two letter flight code prefix (6F IndiGo, AI Air India,
  SJ SpiceJet, UK Vistara) where it is missing or UNKNOWN.
- Passenger names, email, phone, Aadhaar, passport and emergency contact are replaced with
  salted SHA-256 hashes before the Silver layer; `date_of_birth` is dropped. The Power BI
  model contains no readable personal data.
