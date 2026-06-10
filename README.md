# Air Quality ETL Pipeline & Dashboard

A multi-city air quality monitoring pipeline that extracts hourly forecast data
from the Open-Meteo Air Quality API, transforms and validates it, loads it into
a PostgreSQL database (Supabase), and prepares analytics-ready datasets for
visualization in Microsoft Power BI.

---

## Project Overview

This project compares 7-day air quality forecasts across seven U.S. cities
representing a range of geographic, climatic, and demographic profiles:

| City | State | Coordinates |
|---|---|---|
| Louisville | KY | 38.2542°N, 85.7594°W |
| Bakersfield | CA | 35.3733°N, 119.0187°W |
| Brownsville | TX | 25.9017°N, 97.4975°W |
| Eugene | OR | 44.0521°N, 123.0867°W |
| Bozeman | MT | 45.6797°N, 111.0386°W |
| Casper | WY | 42.8666°N, 106.3131°W |
| Kahului | HI | 20.8893°N, 156.4729°W |

Variables are grouped into four analytical categories matching the dashboard layout:

- **Particulates & AQI** — PM2.5, PM10, US AQI and sub-AQI components
- **Gaseous pollutants** — CO, NO₂, SO₂, Ozone
- **Greenhouse gases** — CO₂, Methane
- **Atmospheric / radiation** — UV index, aerosol optical depth, dust

---

## Project Structure

```
air_quality_project/
├── data                      # CSV files
├── notebooks                 # loadscript, python script, SQL for Power BI analytics layer 
├── requirements.txt          # Python dependencies
├── README.md                 
├── .env                      # Database credentials (hidden)
├── .gitignore                # Excludes .env and cache files
└── output/                   # CSV exports (generated on each run)
    ├── readings_clean.csv            # Full cleaned and enriched hourly dataset
    ├── analytics_daily_summary.csv   # One row per city per day — mean/max aggregates
    ├── analytics_aqi_wide.csv        # AQI pivoted wide — one column per city
    ├── analytics_who_exceedances.csv # Rows where at least one WHO limit is breached
    └── qa_report.csv                 # Quality check results from each pipeline run
```

---

## Scripts

### air_quality_pipeline.py

**Purpose**: Production ETL pipeline — extracts live data from the Open-Meteo
Air Quality API and loads it into PostgreSQL via a six-step process.

**Pipeline steps:**

1. **Extraction** — fetches 7-day hourly forecasts for all 7 cities simultaneously
   using the `openmeteo-requests` library with 5-retry logic and a 1-hour local
   request cache to avoid redundant API calls. Validates that the response contains
   data for all expected locations before proceeding.

2. **Cleaning & normalisation** — parses timestamps to UTC, enforces numeric types,
   clips negative concentrations to zero, applies hard upper caps (AQI ≤ 500,
   UV ≤ 20), and interpolates short gaps of up to 3 consecutive missing hours
   per location using linear interpolation.

3. **Transformation & derived metrics** — adds the following engineered columns:
   - `aqi_category` — EPA label (Good / Moderate / Unhealthy etc.)
   - `dominant_pollutant` — which sub-AQI component is driving the overall AQI
   - `pm_ratio` — PM2.5 / PM10 ratio (near 1.0 = combustion; near 0.0 = dust)
   - `uv_cloud_attenuation_pct` — % of clear-sky UV blocked by cloud cover
   - `who_exceedance_*` — boolean flags per pollutant vs WHO 24-hour guideline
   - Time features: `date`, `hour`, `week`, `day_of_week`

4. **Validation & quality checks** — runs 9 automated checks including null
   timestamps, duplicate detection, referential integrity, range validation
   (AQI 0–500, UV 0–20), negative concentration checks, high null rate warnings
   (> 20%), temporal gap detection, and row count verification. All results are
   logged and exported to `output/qa_report.csv`.

5. **Database loading** — writes to PostgreSQL using an incremental append-with-
   deduplication strategy. Each run stages data, deletes matching keys, then
   inserts fresh rows — preventing duplicates while preserving historical data
   not present in the current API window. A `pipeline_runs` audit table records
   every execution's timestamp, row counts, and success/failure status.

6. **Analytics dataset preparation** — exports three analytics-ready CSV files
   optimised for Power BI and Plotly Dash consumption (see Output Files below).

**Usage:**
```bash
python air_quality_pipeline.py
```

---

### supabase_views.sql

**Purpose**: Creates five SQL views in the Supabase PostgreSQL database that
join the hub-and-spoke tables back to readable city names and timestamps,
making them directly consumable by Power BI without further transformation.

**Views created:**

| View | Source tables | Primary use |
|---|---|---|
| `vw_particulates` | particulates + readings + locations | AQI and PM trend charts |
| `vw_gaseous_pollutants` | gaseous_pollutants + readings + locations | Pollutant comparison charts |
| `vw_greenhouse_gases` | greenhouse_gases + readings + locations | CO₂ and methane trends |
| `vw_atmospheric` | atmospheric + readings + locations | UV and aerosol visuals |
| `vw_daily_summary` | all tables joined | Executive overview page |

**Usage:** paste into Supabase SQL Editor and run.

---

## Database Schema

The database uses a hub-and-spoke design where `readings` is the central fact
table and each variable group has its own spoke table joined by `reading_id`.

```
locations ──< readings >──── particulates
                    │
                    ├──────── gaseous_pollutants
                    │
                    ├──────── greenhouse_gases
                    │
                    ├──────── atmospheric
                    │
                    └──────── pipeline_runs (audit log)
```

This structure allows querying only the variable group needed for a given
analysis without scanning the full dataset, and makes adding new variable
groups in future a non-breaking change.

---

## Data Source

**Open-Meteo Air Quality API** — [https://air-quality-api.open-meteo.com](https://air-quality-api.open-meteo.com)

- Free, no authentication required
- Hourly resolution, 7-day forecast window
- 19 variables retrieved per location per run
- Data sourced from CAMS (Copernicus Atmosphere Monitoring Service)

**WHO guideline thresholds used for exceedance flagging:**

| Pollutant | WHO 24-h limit | Unit |
|---|---|---|
| PM2.5 | 15.0 | µg/m³ |
| PM10 | 45.0 | µg/m³ |
| Ozone | 100.0 | µg/m³ |
| NO₂ | 25.0 | µg/m³ |
| SO₂ | 40.0 | µg/m³ |
| CO | 4.0 | mg/m³ |

---

## Output Files

| File | Description | Best used for |
|---|---|---|
| `readings_clean.csv` | Full hourly dataset, all cities, all variables | Raw analysis, additional modelling |
| `analytics_daily_summary.csv` | Mean/max per city per day | Power BI trend charts, daily comparisons |
| `analytics_aqi_wide.csv` | AQI pivoted wide, one column per city | Multi-city time-series in a single chart |
| `analytics_who_exceedances.csv` | Rows where any WHO limit is breached | Compliance reporting, alert filtering |
| `qa_report.csv` | Results of all 9 QA checks | Pipeline monitoring, data quality review |

---

## Requirements

Install all dependencies:
```bash
pip install -r requirements.txt
```

**requirements.txt:**
```
openmeteo-requests
requests-cache
retry-requests
pandas
numpy
sqlalchemy
psycopg2-binary
python-dotenv
```

---

## Setup and Run

### 1. Clone the repository
```bash
git clone <your-repo-url>
cd air_quality_project
```

### 2. Create and activate a virtual environment

macOS / Linux:
```bash
python3 -m venv .venv
source .venv/bin/activate
```

Windows:
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure database credentials

Create a `.env` file in the project root:

```env
DB_HOST=db.xxxxxxxxxxxx.supabase.co
DB_PORT=5432
DB_NAME=postgres
DB_USER=postgres
DB_PASSWORD=your_supabase_password
```

The pipeline reads these automatically on startup via `python-dotenv`.
Never commit `.env` to version control — it is listed in `.gitignore`.

### 5. Create the Supabase views

Open Supabase → SQL Editor → paste and run `supabase_views.sql`.
This only needs to be done once.

### 6. Run the pipeline
```bash
python air_quality_pipeline.py
```

The pipeline will log each step to the terminal and write output files
to the `output/` directory. A successful run looks like:

```
2026-06-10 12:00:00 [INFO] ============================================================
2026-06-10 12:00:00 [INFO] Air Quality Pipeline — START
2026-06-10 12:00:01 [INFO] Extracting data from Open-Meteo API …
2026-06-10 12:00:03 [INFO]   → 1176 hourly rows across 7 locations
2026-06-10 12:00:03 [INFO] Cleaning and normalising …
2026-06-10 12:00:04 [INFO] Transforming and deriving metrics …
2026-06-10 12:00:04 [INFO] Running data validation …
2026-06-10 12:00:04 [INFO]   → QA complete: 9 checks passed, 0 with issues
2026-06-10 12:00:05 [INFO] Loading to PostgreSQL (incremental append with deduplication) …
2026-06-10 12:00:07 [INFO] Building analytics-ready datasets …
2026-06-10 12:00:07 [INFO] Pipeline complete.
```

### 7. Connect Power BI to Supabase

1. Power BI Desktop → Get Data → PostgreSQL database
2. Enter your Supabase host and database name
3. Load the five `vw_*` views
4. Build visuals directly from the views — city, timestamp, and all
   measurement columns are already joined and ready to use

---

## Power BI Dashboard

The report is structured across four pages:

| Page | Views used | Key visuals |
|---|---|---|
| 1 — Overview | `vw_daily_summary` | KPI cards, Azure Maps bubble map, AQI bar chart, summary table |
| 2 — Air quality & AQI | `vw_particulates`, `vw_daily_summary` | AQI trend, PM2.5 vs PM10 bar, AQI category hours stacked bar |
| 3 — Pollutants | `vw_gaseous_pollutants`, `vw_greenhouse_gases` | % of WHO guideline bar, ozone trend, NO₂ trend, CO₂ vs methane dual-axis |
| 4 — Atmospheric | `vw_atmospheric` | Peak UV by city, hourly UV profile, aerosol vs dust scatter, UV vs clear-sky |

A city slicer on each page filters all visuals simultaneously. An
`Is Daytime` calculated column filters UV visuals to hours 6am–8pm,
removing nighttime zeros that would otherwise flatten the charts.

---

## Key Findings

Across the 7-day forecast window, the fleet average AQI was **37.06** (Good),
though Bakersfield, CA recorded the highest average AQI of **68.08** (Moderate)
and the highest PM2.5 average of **16.73 µg/m³** — exceeding the WHO 24-hour
guideline of 15 µg/m³. Kahului, HI recorded the lowest average AQI of **24.70**
but the highest peak UV index of **11.85** (Extreme). A total of **314 WHO
exceedance hours** were flagged across all cities, with ozone and PM2.5 as the
dominant drivers. Full findings are documented in the executive summary.

---

## Notes

- The pipeline is designed to be run on a schedule (e.g. daily cron job) —
  the incremental loading strategy means re-running it will update forecast
  values without duplicating historical rows.
- The 7-day forecast window means data from previous runs that falls outside
  the current API window is preserved in the database untouched.
- All timestamps are stored in UTC in the database and converted to
  America/New_York timezone for display in Power BI.

---

## API Reference

- Open-Meteo Air Quality API: [https://open-meteo.com/en/docs/air-quality-api](https://open-meteo.com/en/docs/air-quality-api)
- WHO Air Quality Guidelines: [https://www.who.int/publications/i/item/9789240034228](https://www.who.int/publications/i/item/9789240034228)
- EPA AQI Technical Assistance Document: [https://www.airnow.gov/sites/default/files/2020-05/aqi-technical-assistance-document-sept2018.pdf](https://www.airnow.gov/sites/default/files/2020-05/aqi-technical-assistance-document-sept2018.pdf)
