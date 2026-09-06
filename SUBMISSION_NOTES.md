# ASG Airlines - Submission Notes

## Overview

The full pipeline is in one notebook, `Angel_Blessy.ipynb`. It reads the supplied Excel
workbook, checks data quality, cleans and standardises the data, hashes the passenger PII,
and builds a star schema with the KPI tables the report needs. On top of the gold tables I
built a 4-page Power BI report, `Dashboard.pbix`.

I ran the notebook end to end before submitting and it completes with no errors. It also
overwrites its own output, so it is safe to re-run.

## How the data moves through the layers

| Layer | Folder | What is in it |
|---|---|---|
| Landing | `Airlines Use Case/` | the original workbook, left untouched |
| Bronze | `data/bronze/` | every row read as text, no business rules yet, data quality flags added |
| Quarantine | `data/quarantine/` | rows dropped at ingestion (missing key or exact duplicate), kept with the reason |
| Silver | `data/silver/` | cleaned data: standardised values, airline repaired, real timestamps, overnight flights handled, duration recomputed, duplicates removed, missing values imputed, PII hashed |
| Gold | `data/gold/` | 5 dimensions, 3 facts and 16 KPI tables, written as CSV so Power BI can pick them up |

Row counts on the supplied data:

- flights: 1,020 in the source, 1,004 in silver (15 exact-duplicate rows quarantined, 1 more
  duplicate key removed during cleaning)
- passengers: 1,039 in the source, 1,000 in silver (39 duplicate passenger ids removed)
- bookings and payments: 1,000 each, unchanged

## Decisions and assumptions worth calling out

**Delay.** The source has no scheduled departure or arrival times, so there is nothing to
measure lateness against directly. I compare each flight to the median duration of the same
route: more than 30 minutes over is "delayed", more than 90 is "severe". One thing this
surfaced is that flights actually run about an hour early on average, so the reliability
issue is inconsistency rather than flights simply running late.

**Airline.** Where the airline is blank or UNKNOWN I fill it from the two-letter flight code
prefix (6F IndiGo, AI Air India, SJ SpiceJet, UK Vistara).

**PII.** Passenger name, email, phone, Aadhaar, passport and emergency contact are replaced
with salted SHA-256 hashes before the silver layer, and date of birth is dropped entirely.
Nothing readable about a passenger reaches the Power BI model.

**Imputation.** 78 payment rows had a missing or non-numeric amount and were set to the
median fare. 75 booking rows had a status value that could not be parsed and were set to
UNKNOWN instead of being thrown away.

## The report

With no slicer applied the visuals line up with the `data/gold/kpi_*.csv` files, for example
Total Flights 1,004, Avg Duration 164.8, Total Revenue about 8.01M, Cancellation Rate 31.4%.

- **Duration Analysis.** Flights average 165 minutes and run roughly an hour early. Duration
  barely moves between airlines (164 to 165 minutes) but does vary by route (174 to 187).
- **Route Performance.** Traffic is spread fairly evenly across the 30 routes with no single
  dominant one. Bookings work out to about one per flight and revenue per flight is similar
  everywhere.
- **Airline Trends.** The four carriers are close on flight count, revenue (around 2M each)
  and average fare (around 8k). Vistara earns the most despite flying the fewest flights.
  Cancellations sit near 31% for all of them.
- **Delay & Anomaly.** 37% of flights are delayed and 27% are flagged as anomalies, both
  well over the 20% target. IndiGo is the worst airline at about 41%, and the Delhi routes
  (BLR-DEL 47%, DEL-BOM 46%) are the worst corridors. Delays bunch up in the 15:00 to 19:00
  departure window.

## Running it

```
pip install pandas pyarrow openpyxl jupyter
jupyter nbconvert --to notebook --execute --inplace Angel_Blessy.ipynb
```

Then open `Dashboard.pbix`.

## What is in the submission

- `Angel_Blessy.ipynb` - the pipeline, Tasks 1 to 3
- `Angel Blessy Document.docx` - written answers to each requirement in the brief
- `data/bronze`, `quarantine`, `silver`, `gold` - the generated layers
- `data/dq_report.csv` - every data quality issue with its row count
- `data/cleaning_summary.csv` - before and after row counts
- `Dashboard.pbix` - the report, Task 4
- `diagrams/` - architecture, data flow and data model diagrams (with Mermaid sources)
- `docs/powerbi/theme.json` - the Power BI theme
