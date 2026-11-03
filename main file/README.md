# Breathing Bharat — VIZ-A-THON 2026

End-to-end analysis of CPCB air-quality data for North India (Chandigarh, Punjab, Haryana, Delhi), 2025, plus AQI history 2017–2023.

## Files

- `Air_Quality_Analysis.ipynb` — main notebook (executed, with figures).
- `SOLUTION_BRIEF.md` — problem statement + proposed solution (SAANS).
- `figures/` — 15 exported figures (PNG).
- `data_processed/` — cleaned & model-ready datasets.
- `data_raw/VIZ-A-THON_2026_Dataset/` — original files (untouched).

## Data processed (fresh datasets)

- `hourly_2025_clean.parquet` / `.csv.gz` — 420,480 station-hours, IST + UTC, converted units, QC flags.
- `daily_model_ready.csv` — 17,520 station-days with AQI, lags, priors, transport features, targets, split labels.
- `aqi_history_daily_clean.csv` — 71,294 repaired station-days (2017–2023).
- `clean_air_windows_2025.csv` — 9,373 station-days with best/worst daylight 3 h window + dose avoided.
- `model_predictions_test.csv` — 2,501 Nov–Dec station-days, actual vs predicted.
- `stations_resolved.csv` — 48 stations with coordinates, coverage, history availability.
- `feature_importance.csv` — model drivers.
- `model_metrics.json` — headline metrics.

## Reproduce

1. Install: `pip install pandas numpy matplotlib seaborn scipy scikit-learn lightgbm nbformat nbconvert pyarrow`
2. Run `python build_notebook.py` to rebuild the notebook from `nb_cells.py`.
3. Run `jupyter-nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=1800 Air_Quality_Analysis.ipynb`

## Headline results

- 24-hour PM2.5 forecast (Nov–Dec hold-out): R² ≈ 0.75, MAE ≈ 24 µg/m³ (persistence 0.71 / 25).
- 3-tier AQI risk classification: ≈ 81% accuracy.
- Clean-Air Window: ~33% average dose reduction by shifting a 3 h daylight block.
- 82% of station-days have a safe 3 h daylight window; 18% have none.
