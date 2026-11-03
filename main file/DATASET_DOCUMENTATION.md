# Dataset Documentation
## VIZ-A-THON 2026 — Air Quality in North India (2025)

**Prepared from:** `data_raw/VIZ-A-THON_2026_Dataset/` (CPCB CAAQM)  
**Companion files:** `Air_Quality_Analysis.ipynb`, `Cleaning_and_Preprocessing.pdf`, `README.md`, `SOLUTION_BRIEF.md`

---

## 1. Overview

This dataset contains real air-quality and weather measurements from **48 government monitoring stations**
in Chandigarh, Punjab, Haryana and Delhi for the whole of 2025, plus each station's hourly AQI history
for 2017–2023. The data is given exactly as published by CPCB (Central Pollution Control Board) and the
state pollution control boards: **not cleaned, not filled, not corrected**.

| File | Rows | Columns | Grain |
|---|---:|---:|---|
| `air_quality_hourly_2025.csv` | 414,255 | 28 | station × hour (UTC) |
| `aqi_history_2017_2023.csv` | 102,744 | 29 | station × day (24 hourly AQI columns) |
| `stations.csv` | 48 | 4 | one monitoring station |

Stations per state: Chandigarh (3), Punjab (8), Haryana (31), Delhi (6).

## 2. Source and licence

- **Data:** CPCB CAAQM data repository and AQI repository (`airquality.cpcb.gov.in`), compiled by the
  open-source project `india-cpcb-aqi` (github.com/Vonter/india-cpcb-aqi). This extract keeps only the
  stations listed in `stations.csv`; values are unchanged.
- **Licence:** Open Database License (ODbL) 1.0. Some contents are copyright CPCB. Credit CPCB and
  india-cpcb-aqi when publishing work built on this data.
- **Guide:** `Dataset_Guide.pdf` (in the raw folder) is the official reference.

## 3. File 1 — `air_quality_hourly_2025.csv`

One row = the 15-minute reading recorded at the **start of the hour**, in **UTC**. IST = UTC + 5 h 30 min.
A blank cell means no reading was published for that hour.

### 3.1 Schema

| # | Column (as in file) | Clean name | Meaning | Unit |
|---|---|---|---|---|
| 1 | Station ID | `station_id` | Unique station code, e.g. site_5127 | — |
| 2 | State | `state` | State / UT | — |
| 3 | City | `city` | City of the station | — |
| 4 | Station Name | `station_name` | Station name + operating agency | — |
| 5 | Timestamp | `timestamp_utc` | Date/time of reading, UTC | YYYY-MM-DD HH:MM:SS |
| 6 | PM2.5 (µg/m³) | `pm25` | Fine particulate matter (≤2.5 µm) | µg/m³ |
| 7 | PM10 (µg/m³) | `pm10` | Particulate matter (≤10 µm) | µg/m³ |
| 8 | NO (µg/m³) | `no` | Nitric oxide | µg/m³ |
| 9 | NO2 (µg/m³) | `no2` | Nitrogen dioxide | µg/m³ |
| 10 | NOx (ppb) | `nox_ppb` | Nitrogen oxides (NO + NO2) | ppb |
| 11 | NH3 (µg/m³) | `nh3` | Ammonia | µg/m³ |
| 12 | SO2 (µg/m³) | `so2` | Sulphur dioxide | µg/m³ |
| 13 | CO (mg/m³) | `co` | Carbon monoxide | mg/m³ |
| 14 | Ozone (µg/m³) | `ozone` | Ground-level ozone | µg/m³ |
| 15 | Benzene (µg/m³) | `benzene` | Benzene (VOC) | µg/m³ |
| 16 | Toluene (µg/m³) | `toluene` | Toluene (VOC) | µg/m³ |
| 17 | Xylene (µg/m³) | `xylene` | Xylene (VOC) | µg/m³ |
| 18 | Eth-Benzene (µg/m³) | `ethylbenzene` | Ethylbenzene (VOC) | µg/m³ |
| 19 | MP-Xylene (µg/m³) | `mp_xylene` | Meta/para-xylene (VOC) | µg/m³ |
| 20 | AT (°C) | `temp_c` | Ambient air temperature | °C |
| 21 | RH (%) | `rh_pct` | Relative humidity | % |
| 22 | WS (m/s) | `ws_ms` | Wind speed | m/s |
| 23 | WD (deg) | `wd_deg` | Wind direction (degrees from north) | deg |
| 24 | RF (mm) | `rain_mm` | Rainfall | mm |
| 25 | TOT-RF (mm) | `total_rain_mm` | Total rainfall | mm |
| 26 | SR (W/mt2) | `solar_wm2` | Solar radiation | W/m² |
| 27 | BP (mmHg) | `pressure_mmhg` | Barometric pressure | mmHg |
| 28 | VWS (m/s) | `vws_ms` | Vertical wind speed | m/s |

### 3.2 Data-quality notes

- **Missingness is structural.** 43.6% of station-hours have all nine pollutant readings missing.
  Haryana's PM2.5 coverage is ~5% in Jul–Sep and ~87% in Nov–Dec.
- **No hourly duplicates.** 0 duplicate `(station_id, timestamp_utc)` pairs.
- **Mixed units.** `NOx` is in ppb; convert to µg/m³ as NO₂: `nox_ugm3 = nox_ppb × 1.88`.
- **Duplicate rain column.** `TOT-RF (mm)` is identical to `RF (mm)` (max diff 0.0) — drop one.
- **Near-empty column.** `VWS` is 98.3% missing — drop.
- **Instrument ceilings.** PM2.5 = 999.99 (11 rows), 500 (7); PM10 = 1000 (1); CO = 10 (10). Kept as the tail signal; capped only inside AQI mapping.
- **Below-detection zeros.** CO 1.46%, Benzene 1.42% are 0 — kept (CPCB publishes 0 for below detection).
- **Station coverage.** 48 stations, each nominally 8,760 hours; one station (site_161, Panchkula) has only 2,957 raw rows.

## 4. File 2 — `aqi_history_2017_2023.csv`

One row = one station × one day, with 24 columns `00:00:00` … `23:00:00` holding the AQI published by CPCB
for each hour. Missingness 5.7–9.1% by hour.

### 4.1 Schema

| Column | Meaning |
|---|---|
| Station ID | Station code (but see §4.2 — reused across stations) |
| State, City | Administrative labels (unreliable for collided IDs) |
| Station Name | The only trustworthy identity field |
| Date | Date of the record |
| 00:00:00 … 23:00:00 | Hourly AQI (integer, 0–500) |

### 4.2 Critical quality issue — Station ID collision

Four IDs are reused across multiple physical stations. `site_5046` carries 40 different station names
(Kanpur, Jaipur, Bhopal, Ghaziabad, Ratlam, …). Joining on Station ID alone attaches the wrong city's AQI.
**Use a composite key (normalised station name → station_id, exact match only).** After repair: 71,294 rows /
40 physical stations recovered; 31,450 rows dropped as non-target or ambiguous.

### 4.3 History coverage

- 40 of 48 hourly stations have history (8 do not).
- Years: 2017 (sparse) → 2023 (full). 2020 shows a COVID lockdown dip (~132 vs ~139 in 2019).
- Annual mean AQI on a balanced 7-station panel: 2017 204 → 2020 132 → 2022 162 → 2023 140.
- Monthly climatology: November peak (≈248), December (≈216), January (≈184), monsoon minimum (≈82–89).

## 5. File 3 — `stations.csv`

| Column | Meaning |
|---|---|
| State | State or union territory |
| Station ID | Unique station code |
| City | City of the station |
| Station Name | Station name and operating agency (CPCC / DPCC / HSPCB / PPCB / IMD / IITM) |

48 stations: Chandigarh 3, Punjab 8, Haryana 31, Delhi 6.

## 6. CPCB Air Quality Index reference

India's National AQI (CPCB 2014) groups air quality into six categories. Each pollutant gets a sub-index
from its concentration; the AQI is the **highest** sub-index, using at least three pollutants, one of which
must be PM2.5 or PM10.

| Category | AQI | PM2.5 (24h) | PM10 (24h) | NO₂ (24h) | O₃ (8h) | CO (8h) | SO₂ (24h) | NH₃ (24h) |
|---|---|---|---|---|---|---|---|---|
| Good | 0–50 | 0–30 | 0–50 | 0–40 | 0–50 | 0–1.0 | 0–40 | 0–200 |
| Satisfactory | 51–100 | 31–60 | 51–100 | 41–80 | 51–100 | 1.1–2.0 | 41–80 | 201–400 |
| Moderate | 101–200 | 61–90 | 101–250 | 81–180 | 101–168 | 2.1–10 | 81–380 | 401–800 |
| Poor | 201–300 | 91–120 | 251–350 | 181–280 | 169–208 | 10.1–17 | 381–800 | 801–1200 |
| Very Poor | 301–400 | 121–250 | 351–430 | 281–400 | 209–748 | 17.1–34 | 801–1600 | 1201–1800 |
| Severe | 401–500 | 250+ | 430+ | 400+ | 748+ | 34+ | 1600+ | 1800+ |

Concentrations in µg/m³, except CO in mg/m³.

### National Ambient Air Quality Standards (CPCB 2009, residential)

| Pollutant | Annual avg | 24-h avg | Shorter period |
|---|---|---|---|
| PM2.5 | 40 µg/m³ | 60 µg/m³ | — |
| PM10 | 60 | 100 | — |
| NO₂ | 40 | 80 | — |
| SO₂ | 50 | 80 | — |
| Ozone | — | — | 100 (8 h), 180 (1 h) |
| CO | — | — | 2 mg/m³ (8 h), 4 mg/m³ (1 h) |
| NH₃ | 100 | 400 | — |
| Benzene | 5 | — | — |

## 7. Preprocessing transformations applied

| Step | Transformation | Reason |
|---|---|---|
| Timestamps | UTC → IST (+5:30); keep both | Activity/traffic/peaks are local |
| Columns | Rename by position; snake_case; units in name | Mojibake in headers; mixed units |
| NOx | ppb → µg/m³ as NO₂ (×1.88) | Unit consistency |
| Station identity (history) | Composite key (normalised name → ID), exact match only | ID reuse across 40 stations |
| Duplicates | Drop exact; collapse repaired history to station-day mean | Identity artefact, not repeated measurements |
| Long gaps | Reindex to full 8,760-h grid; never fill | Lags need regular axis; explicit NaN |
| Short gaps | No global imputation; LGBM handles NaN; daily means need ≥16 valid hours | Missingness is block-structured (50–95%) |
| Daily aggregation | 24 h mean (PM/NO₂/SO₂/NH₃); 8 h max (O₃/CO); diurnal blocks | CPCB AQI rule; exposure-relevant blocks |
| AQI | CPCB sub-indices, piecewise linear; max sub-index; ≥3 pollutants incl. PM | Legal definition; auditable |
| Outliers/ceilings | Keep; flag; cap only inside AQI mapping | Extremes are the signal |
| Below-detection zeros | Keep, flag | CPCB publishes 0 for below detection |
| Wind direction | sin/cos + u/v components | Circular discontinuity |
| Scaling | None (tree models) | Scale-invariant |
| Train/test split | Time-based: Jan–Sep train, Oct val, Nov–Dec test | Avoids leakage from autocorrelation |
| Seasonal extrapolation | Seasonal prior (2017–2023) + residual/ratio model | Direct model cannot extrapolate (R² = −0.12) |
| Class imbalance | 3-tier labels + balanced weights; no SMOTE | Operational decision; preserves real prior |

## 8. Derived columns in the clean datasets

### 8.1 `hourly_2025_clean` (420,480 rows × 36 cols)

Original 28 columns (renamed) + `timestamp_ist`, `date_ist`, `hour_ist`, `nox_ugm3`,
`wind_u`, `wind_v`, `pm25_flag`, `pm10_flag`. Dropped `total_rain_mm` (duplicate) and `vws_ms` (98.3% missing).

### 8.2 `daily_model_ready` (17,520 rows × ~169 cols)

- **Identity:** `station_id`, `state`, `city`, `station_name`, `lat`, `lon`, `date_ist`, `month`, `season`, `split`
- **Daily pollutants:** `pm25_mean`, `pm10_mean`, `no2_mean`, `so2_mean`, `nh3_mean`, `co_mean`, `o3_mean`,
  `o3_8h_max`, `co_8h_max`, `benzene_mean`, `pm25_max`, `pm25_p90`, `n_pm25`, `coverage`
- **Meteorology:** `temp_mean`, `temp_min`, `temp_max`, `rh_mean`, `ws_mean`, `ws_max`, `rain_sum`,
  `solar_mean`, `pressure_mean`, `calm_frac`, `wind_u_mean`, `wind_v_mean`, `wind_dir_deg`, `wind_vec_mag`
- **Diurnal:** `pm25_morning`, `pm25_afternoon`, `pm25_evening`, `pm25_night`
- **AQI:** `aqi`, `aqi_official`, `aqi_category`, `aqi_n`, `si_*` (7 sub-indices), `tier`
- **Transport:** `upwind_pm25_d0/d1/d2`, `regional_pm25_d0/d1/d2`, `upwind_minus_local`,
  `stagnation`, `transport_index`, `regime`
- **History priors:** `prior_aqi_final`, `prior_pm25`, `prior_pm25_next`, `prior_aqi_next`,
  `prior_ratio_today`, `prior_aqi_ratio_today`
- **Station history:** `hist_mean_aqi`, `hist_p90_aqi`, `hist_max_aqi`, `hist_days`,
  `hist_winter_aqi`, `hist_postmon_aqi`, `hist_monsoon_aqi`, `hist_summer_aqi`, `hist_trend_per_year`
- **Date features:** `doy`, `dow`, `is_weekend`, `doy_sin`, `doy_cos`, `winter_index`, `wind_sin`, `wind_cos`
- **Lags/rolls:** `{pm25,aqi,pm10,no2,so2,co,o3}_mean_lag{1,2,3,5,7,14,21,28}`,
  `{...}_roll{3,7,14,28}`; met lags
- **Targets:** `pm25_next`, `aqi_next`, `cat_next`, `tier_next`, `target_ratio`
- **Split:** `train` / `validation` / `test`

### 8.3 `clean_air_windows_2025` (9,367 rows)

`station_id`, `date_ist`, `daily_mean`, `best_start_hour`, `best_3h_mean`, `worst_start_hour`,
`worst_3h_mean`, `dose_reduction_pct`, `dose_avoided_ug`, `state`, `city`, `month`, `season`.

### 8.4 `stations_resolved` (48 rows)

`state`, `station_id`, `city`, `station_name`, `raw_rows`, `pm25_valid`, `pm25_cov_pct`,
`lat`, `lon`, `hist_mean_aqi`, `hist_winter_aqi`, `hist_trend_per_year`, `has_history`.

### 8.5 `aqi_history_daily_clean` (71,294 rows)

`station_id`, `date`, `hist_aqi_mean`, `hist_aqi_max`, `hist_hours`, `year`, `month`, `doy`.

### 8.6 `model_predictions_test` (2,501 rows)

`date_ist`, `station_id`, `state`, `city`, `pm25_mean`, `pm25_next`, `prior_pm25_next`, `pred`,
`persistence`, `delta_model`, `ratio_model`, `level_model`, `residual`, `month`, `abs_error`,
`tier_next`, `cat_next`, `tier_pred`, `cat_pred`.

## 9. Key results (for reference)

- **24 h PM2.5 forecast (Nov–Dec hold-out):** R² ≈ 0.760, MAE ≈ 23.4 µg/m³ (persistence 0.712 / 24.8).
- **3-tier AQI risk classification:** accuracy ≈ 0.808, macro-F1 ≈ 0.739.
- **6-class CPCB:** accuracy ≈ 0.609, macro-F1 ≈ 0.449, Severe recall 0.055.
- **Clean-Air Window:** ~33% average dose reduction; 82% of days have a safe 3 h daylight window; 18% have none.
- **Blend weights:** persistence 0.481, ratio 0.519 (delta and level ≈ 0).

## 10. Caveats for users

1. **Coverage bias.** Haryana's summer data is sparse; state comparisons must be coverage-aware.
   `pm25_cov_pct` is provided in `stations_resolved.csv`.
2. **"Transport" is a statistical index**, not chemical source apportionment (no PMF/CMB). It is a
   wind-aligned upwind signal, labelled as an index, not a source share.
3. **Coordinates are approximate** city/station centroids — suitable for regional geometry, not
   hyperlocal mapping.
4. **Forecast uncertainty.** MAE ≈ 23.4 µg/m³; use the upper quantile for vulnerable groups.
5. **History identity.** Always join history on the repaired `station_id` (composite key), never on the
   raw `Station ID`.
6. **Timezone.** Hourly timestamps are UTC; use `timestamp_ist` / `date_ist` for any local analysis.
7. **Units.** `NOx` is converted to µg/m³ as NO₂ (`nox_ugm3`); the original `nox_ppb` is retained.
   `CO` is in mg/m³, all other pollutants in µg/m³.

## 11. Reproduce

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn lightgbm nbformat nbconvert pyarrow fpdf2
python build_notebook.py                       # rebuild notebook from nb_cells.py
jupyter-nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=1800 Air_Quality_Analysis.ipynb
python build_viz.py                            # visualization notebook
```

## 12. Credits and licence

- Data: CPCB CAAQM and AQI repositories; compiled by `india-cpcb-aqi` (ODbL 1.0).
- Guide: `Dataset_Guide.pdf` (raw folder).
- Methods: CPCB AQI breakpoints; LightGBM; seasonal-prior/residual decomposition; wind-aligned upwind
  index; dose = concentration × ventilation × duration.
- Credit CPCB and india-cpcb-aqi when publishing work built on this data.
