# SAANS — Source-Aware Air-quality Nowcast & Scheduling
### Proposed solution for VIZ-A-THON 2026 (North Indian Air Quality, 2025)

## 1. Problem statement (unique)

> **"Most AQI systems tell you how bad the air is. Almost none tell a school, a construction crew or a health clinic *when* the air will be least harmful, *how much* exposure can be avoided by moving a 3-hour outdoor block, and *whether* tomorrow's pollution is coming from the next district or from the street outside. Can we build a 24-hour, wind-aware, station-level 'Clean-Air Window' engine that turns an AQI forecast into a dose-minimising schedule — and attributes each episode to local stagnation vs regional transport?"**

The unit of analysis is not the AQI. It is the **inhaled dose of a person who has to be outside**.

## 2. Why existing solutions fall short

- They predict a **daily number**, but a school needs a **3-hour window**.
- They treat all pollution as local, so enforcement is blanket (GRAP).
- They ignore ventilation rate and activity level, so "AQI 250" does not say how much is inhaled.
- They are app-only, excluding feature-phone users and rural schools.
- They rarely quantify uncertainty or coverage gaps.

## 3. The proposed solution — SAANS

A four-module, station-level decision-support layer on top of CPCB data.

### A. Forecast engine (built in this notebook)
- 24-hour PM2.5 per station: seasonal prior (2017–2023) + delta/ratio LightGBM blend.
- Honest Nov–Dec backtest: R² ≈ 0.75, MAE ≈ 24 µg/m³ (persistence 0.71 / 25).
- 3-tier risk: Low (≤100), Moderate (101–200), High (>200), ~81% accuracy.

### B. Source attribution (the differentiator)
- Wind-aligned upwind index: stations in the ±55° upwind cone, 15–400 km, weighted by distance and wind alignment.
- Regime classification:
  - **Local stagnation** — calm, local PM2.5 > upwind → local traffic/dust controls.
  - **Regional transport** — windy, upwind PM2.5 ≥ local → inter-state coordination.
  - **Mixed** — both.
- On 14 Dec 2025 the map shows the plume bridge; the model's top features include regional PM2.5 at 1–2-day lags.

### C. Clean-Air Window optimiser (the product)
- For each station and the next 24 h, choose a contiguous 2–4 h window minimising inhaled dose subject to AQI < threshold and daylight/schedule constraints.
- Dose = concentration × ventilation rate × duration.
- Quantified outcome: **~33% average dose reduction** from shifting a 3 h daylight block; 82% of days have a safe window; 8% have none (→ stay indoors).
- Best window is usually 08:00 or 15:00, but it moves with the season.

### D. Delivery (the adoption layer)
- Schools: SMS/IVR at 06:00 with the day's window; printable timetable.
- Outdoor workers: shift-level advisory to supervisors.
- Clinics: 24 h alert for COPD/asthma patients; pre-position inhalers.
- Municipalities: source-attribution dashboard for targeted inspection instead of blanket bans.

## 4. Architecture

```
CPCB / OpenAQ ──► ingestion (hourly) ──► cleaning + QC
                                              │
                       ┌──────────────────────┼──────────────────────┐
                       ▼                      ▼                      ▼
              24h forecast (LGBM)     source attribution      clean-window optimiser
                       │                      │                      │
                       └──────────────┬───────┴──────────────────────┘
                                      ▼
                         advisory API (station × hour)
                           │          │           │
                        SMS/IVR   school app   municipal dashboard
```

## 5. Real-world impact

| Problem | Current practice | SAANS response | Measurable outcome |
|---|---|---|---|
| Schools cancel recess only after AQI is severe | Daily AQI number | 3 h clean window + SMS at 06:00 | ~33% lower dose for the same outdoor time |
| Construction workers cannot choose hours | Blanket stop-work orders | Shift-level window + indoor fallback | Fewer lost work-hours, lower dose |
| Clinics overwhelmed during episodes | Reactive admissions | 24 h risk alert + source flag | Earlier inhaler/OPD prep |
| GRAP is blanket and blunt | City-wide bans | Local vs transport attribution | Targeted inspections, less economic damage |
| Rural/feature-phone users excluded | App-only dashboards | SMS/IVR in Hindi/Punjabi | Last-mile reach |
| Data gaps hide vulnerable districts | Averages over incomplete stations | Coverage flags + low-coverage warnings | Honest, equitable reporting |

## 6. Novelty

1. Dose-minimising schedule, not just AQI prediction (objective = µg inhaled).
2. Wind-aware source attribution at station level using only public CPCB data.
3. Season-aware window logic (winter 4 a.m. peak vs pre-monsoon afternoon dust).
4. Feature-phone-first delivery (SMS/IVR, school bell, worksite shift plan).
5. Coverage-aware equity (low-coverage stations flagged, not silently averaged).

## 7. Pilot plan (90 days)

- Pilot cities: Delhi (Wazirpur/Nehru Nagar), Ludhiana, Panchkula.
- Partners: 10 schools, 3 construction sites, 2 clinics.
- Data: CPCB hourly + IMD forecast + school timetables.
- Evaluation: randomised stepped-wedge; primary outcome estimated inhaled dose (µg) per child-week; secondary: symptom days, absenteeism, compliance.
- Success criteria: ≥25% dose reduction, ≥70% schedule compliance, <5% false "safe window" rate.

## 8. Limitations and ethics

- Coverage bias: Haryana's summer data is sparse; state comparisons are labelled.
- No chemical speciation: "transport" is a wind-aligned statistical signature, not source apportionment (no PMF/CMB).
- Coordinates are approximate city/station centroids, suitable for regional geometry only.
- Forecast uncertainty: MAE ~24 µg/m³; use the upper quantile for vulnerable groups.
- Equity: SMS/IVR must be free; do not shift the burden of protection to workers without employer obligations.
- Privacy: no individual tracking; scheduling is station-level and opt-in.
