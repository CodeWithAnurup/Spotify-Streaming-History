# 🎵 Spotify Streaming History — Data Cleaning & Preprocessing

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg?logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/pandas-2.0%2B-150458.svg?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626.svg?logo=jupyter&logoColor=white)](https://jupyter.org/)
[![Internship](https://img.shields.io/badge/Internship-SoftNexis-0052CC.svg)](https://github.com/CodeWithAnurup)
[![Status](https://img.shields.io/badge/Status-Completed-success.svg)]()

> **Project:** SoftNexis Data Analytics Internship — Project 1  
> **Milestone:** Task 1 — Comprehensive Data Health Assessment, Cleaning & Feature Transformation  
> **Author:** [Anurup Samanta](https://github.com/CodeWithAnurup)  
> **Dataset Period:** July 8, 2013 – December 15, 2024 (~11.5 Years)

---

## 📌 Executive Summary

Real-world streaming logs contain noise, app telemetry drops, timestamp collisions, and irregular event definitions that corrupt downstream business intelligence and analytical models.

This repository presents the end-to-end execution of **Task 1: Data Cleaning & Preprocessing** on an extensive **11.5-year Spotify personal streaming history dataset** containing **149,860 interaction records**. Through disciplined diagnostic profiling, deduplication, missingness strategy, chronological normalization, outlier catalog cross-referencing, and telemetry anomaly flagging, the raw log was transformed into an **analysis-ready dataset** of **148,463 records × 13 attributes** stored in `Cleaned_data_spotify_history.csv`.

---

## 📁 Repository Structure

```text
Spotify-Streaming-History/
├── Cleaned_data_spotify_history.csv     # Transformed, analysis-ready dataset (148,463 rows × 13 cols)
├── Guide_pdf.pdf                        # Internship task guidelines & requirement specifications
├── README.md                            # Comprehensive technical documentation & methodology
├── data_cleaning.ipynb                  # Interactive Jupyter Notebook detailing full cleaning steps
├── spotify_data_dictionary.csv          # Feature definitions and semantic descriptions
└── spotify_history.csv                  # Raw telemetry export (149,860 rows × 11 cols)
```

---

## 📊 Data Architecture & Dictionary

| Column Name | Raw Type | Clean Type | Description | Cleaning Action / Business Logic |
| :--- | :---: | :---: | :--- | :--- |
| `Uniform_Resource_Identifier` | `object` | `string` | Base-62 Spotify track identifier | Renamed from `spotify_track_uri` for standardized naming. |
| `Time_Stamp` | `object` | `datetime64[us]` | Play completion timestamp in UTC | Converted to datetime object; spans 2013-07-08 to 2024-12-15. |
| `platform` | `object` | `string` | Client operating system or playback host | Validated 6 categorical values (`android`, `iOS`, `web player`, etc.). |
| `Actually_Played(ms)` | `int64` | `int64` | Exact playback duration in milliseconds | Renamed from `ms_played`. Validated range: `0` to `1,561,125 ms`. |
| `track_name` | `object` | `string` | Track / song title | Trimmed leading/trailing whitespace (`str.strip()`). |
| `artist_name` | `object` | `string` | Performing artist / band | Trimmed leading/trailing whitespace (`str.strip()`). |
| `album_name` | `object` | `string` | Parent album / collection name | Trimmed leading/trailing whitespace (`str.strip()`). |
| `reason_start` | `object` | `string` | Trigger for playback initiation | Removed 143 null rows; preserved valid telemetry categories. |
| `reason_end` | `object` | `string` | Reason playback terminated | Removed 117 null rows; analyzed against user skip actions. |
| `shuffle` | `bool` | `bool` | Playback shuffle state | Preserved boolean flag. |
| `skipped` | `bool` | `bool` | User skip flag | Preserved boolean flag; cross-tabulated against `reason_end`. |
| `was_played` | *New* | `bool` | Playback integrity flag (`ms > 0`) | **Engineered Feature:** Flags zero-millisecond app load events. |
| `minutes_played` | *New* | `float64` | Duration in minutes (`ms / 60,000`) | **Engineered Feature:** Standardized unit for human-readable EDA. |

---

## 🛠️ Data Health Diagnosis & Cleaning Methodology

The cleaning pipeline strictly follows the industrial standard defined in the project specification:

```
[ Raw Logs ] (149,860 rows)
     │
     ▼
[ Step 1: Ingestion & Sanity Checks ] ──────► Audited initial shape, memory footprint & dtypes
     │
     ▼
[ Step 2: Missingness Diagnostics ] ────────► Handled 143 reason_start & 117 reason_end nulls (<0.2%)
     │                                        Result: 149,648 rows remaining
     ▼
[ Step 3: Exact Deduplication ] ────────────► Removed 1,185 duplicate telemetry captures
     │                                        Result: 148,463 rows remaining
     ▼
[ Step 4: Disambiguation Analysis ] ────────► Investigated 324 identical (URI, Time_Stamp) collisions
     │                                        Preserved valid distinct play sessions
     ▼
[ Step 5: Normalization & Type Cast ] ──────► Cast `ts` to `datetime64[us]`; stripped string whitespace
     │
     ▼
[ Step 6: Outlier Verification & Audit ] ───► Verified 1,561,125 ms against Spotify Catalog
     │                                        Confirmed legitimate track ('Tubular Bells - Pt. I')
     ▼
[ Step 7: Feature Engineering & Export ] ───► Added `was_played`, `minutes_played`; exported CSV
```

### 1. Missing Value Diagnosis & Treatment
- **Diagnosis:** `df.isnull().sum()` revealed missing entries exclusively in two categorical telemetry indicators: `reason_start` (143 nulls) and `reason_end` (117 nulls).
- **Strategy:** Because missingness accounted for less than **0.2%** of all observations (well below standard imputation necessity thresholds), listwise deletion via `df.dropna()` was executed to guarantee strict reporting accuracy, followed by `df.reset_index(drop=True)`.

### 2. Deduplication & Collision Disambiguation
- **Exact Duplicates:** 1,185 identical rows across all 11 columns were identified and purged via `df.drop_duplicates()`.
- **Timestamp Collisions:** Further inspection identified **324 rows** sharing identical track URIs and completion timestamps. A deep-dive confirmed these records exhibited distinct playback durations (`Actually_Played(ms)`) and end reasons (e.g., app load handoffs, rapid re-clicks). Because they represent distinct client events rather than redundant entries, they were intentionally retained.

### 3. Chronological & Data Type Alignment
- The raw timestamp `ts` was formatted as a flat UTC string. It was parsed into a native `datetime64[us]` Series, enabling high-performance time-delta slicing and validation.
- The temporal distribution covers **4,178 calendar days** (from `2013-07-08 02:44:34` to `2024-12-15 23:06:25`).

### 4. Categorical Normalization & Whitespace Stripping
- Unseen leading/trailing whitespaces can cause artificial fragmentation during aggregation (e.g., `'Radiohead '` vs `'Radiohead'`). A vectorized `.str.strip()` was executed across `track_name`, `artist_name`, and `album_name`.

### 5. Domain-Specific Outlier & Catalog Cross-Referencing
- **Maximum Duration Evaluation:** Descriptive statistics indicated a maximum playback of **1,561,125 ms (~26.02 minutes)**. Rather than naively truncating this as a statistical outlier, the record was extracted using `df.loc[df['Actually_Played(ms)'].idxmax()]`:
  - **Track Name:** *Tubular Bells - Pt. I*
  - **Artist:** *Mike Oldfield*
  - **Album:** *Tubular Bells*
  - **Catalog Verification:** Spotify's official catalog confirms the studio recording is **25 minutes and 29 seconds** long. The record was verified as authentic and preserved.

### 6. Telemetry Anomaly Handling (`0 ms` Plays)
- An audit identified **3,733 entries with `Actually_Played(ms) == 0`**. These represent UI click-throughs, rapid skips before buffer initialization, or network drops.
- **Handling Strategy:** Instead of dropping these records (which would destroy behavioral skip/navigation data), an explicit boolean indicator `was_played` was engineered (`df['Actually_Played(ms)'] > 0`), and durations were scaled to `minutes_played` (`ms / 60000`).

---

## 📈 Quality Comparison: Before vs. After

| Metric | Raw Dataset (`spotify_history.csv`) | Cleaned Dataset (`Cleaned_data_spotify_history.csv`) |
| :--- | :---: | :---: |
| **Total Rows** | 149,860 | **148,463** |
| **Total Columns** | 11 | **13** |
| **Null Values** | 260 | **0** |
| **Duplicate Rows** | 1,185 | **0** |
| **Timestamp Format** | String object | **Datetime64[us] (UTC)** |
| **Text Cleanliness** | Unsanitized text strings | **Whitespace-normalized** |
| **Zero-Play Handled** | Ambiguous numeric `0` | **Flagged via `was_played` boolean** |
| **Duration Units** | Milliseconds only | **Milliseconds + Standardized Minutes** |

---

## 🚀 Getting Started & Reproducibility

### Prerequisites
- Python 3.9+
- Jupyter Notebook / JupyterLab

### Installation
1. Clone this repository:
   ```bash
   git clone https://github.com/CodeWithAnurup/Spotify-Streaming-History.git
   cd Spotify-Streaming-History
   ```

2. Set up a virtual environment (optional but recommended):
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn jupyter
   ```

4. Launch the notebook:
   ```bash
   jupyter notebook data_cleaning.ipynb
   ```

---

## 🔮 Roadmap / Next Steps (Task 2 Preview)

With high-integrity data prepared, the project transitions into **Task 2: Exploratory Data Analysis (EDA)**:
- [ ] **Temporal Analysis:** Heatmaps of hourly streaming distribution across weekdays vs. weekends.
- [ ] **Artist & Track Lifetime Value:** Frequency-based vs. duration-based ranking of top musical creators.
- [ ] **Skip Behavioral Mechanics:** Modeling skip probability against start/end telemetry drivers.
- [ ] **Platform Shifts:** Tracing the client device evolution across the 11.5-year span.

---

## 👤 Author & Acknowledgments

- **Intern / Author:** [Anurup Samanta](https://github.com/CodeWithAnurup)
- **Organization:** SoftNexis Internship Program
- **Data Source:** Spotify Listening History (Maven Analytics / Spotify User Privacy Export)

---
*Developed with rigorous attention to data hygiene, reproducible methodologies, and industry standards.*
