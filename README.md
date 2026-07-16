# Forecasting German Electricity Demand (2015–2020)

Forecasting the national electricity load of Germany over a two-year (104-week)
horizon using the [Open Power System Data (OPSD)](https://data.open-power-system-data.org/time_series/)
time series. The project benchmarks classical, machine-learning, and deep-learning
forecasters against a seasonal-naive baseline and analyses *why* certain model
families succeed or fail at long horizons, including the effect of the March 2020
COVID-19 structural break.

**Author:** Thammishetti Venkat Sai Prathap · **Student ID:** 24089794

---

## Overview

The weekly-aggregated series (299 observations, 11 Jan 2015 – 27 Sep 2020) is split
into a 195-week training set and a 104-week test window. The following models are
fitted and compared:

| Model | Type | Notes |
|---|---|---|
| Mean / Naive / Drift / **Seasonal naive** | Benchmarks | Seasonal naive is the reference baseline |
| SARIMA (1,1,6)(1,1,1,52) | Statistical | Order selected by AIC grid search |
| SARIMAX (+ temperature, holidays) | Statistical | Conditional forecast with exogenous regressors |
| Random Forest | Tree ensemble | Best overall; +4.9% skill vs seasonal naive |
| Gradient Boosting | Tree ensemble | +3.7% skill vs seasonal naive |
| LSTM | Neural (hourly) | One-step-ahead vs recursive multi-step comparison |

Headline result: only the two tree ensembles beat the seasonal-naive benchmark
over the full window, and their advantage roughly triples (≈13–17%) in the
pre-pandemic regime.

---

## Repository contents

```
.
├── Forecasting_German_Electricity_Demand.ipynb   # full analysis pipeline
├── Forecasting_German_Electricity_Demand.pdf     # written report (6–8 pages)
├── requirements.txt
└── README.md
```

> **Note on the data file:** `time_series_60min_singleindex.csv` (~130 MB) is the
> raw OPSD 60-minute package and is **not** committed to this repository. Download
> it separately (see below) and place it alongside the notebook.

---

## Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/Saiprathap22/Forecasting-German-Electricity-Demand-.git
   cd Forecasting-German-Electricity-Demand-
   ```

2. **Create a virtual environment** (recommended)
   ```bash
   python -m venv .venv
   # Windows
   .venv\Scripts\activate
   # macOS / Linux
   source .venv/bin/activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Download the load data**

   Get the OPSD *Time Series* package (release **2020-10-06**) from
   <https://data.open-power-system-data.org/time_series/> and save the file
   `time_series_60min_singleindex.csv` in the project root. Only two columns are
   used: `utc_timestamp` and `DE_load_actual_entsoe_transparency`.

---

## Running the analysis

Launch the notebook and run all cells top to bottom:

```bash
jupyter notebook Forecasting_German_Electricity_Demand.ipynb
```

The notebook runs end to end and reproduces every figure and metric table in the
report. It is organised in the same order as the report:

1. Setup and data preparation (MW → GW, missing-value handling, weekly aggregation)
2. Exploratory analysis, STL decomposition, and stationarity tests (ADF / KPSS)
3. Benchmark models
4. SARIMA (AIC grid search)
5. SARIMAX with temperature and holiday regressors
6. Random Forest and Gradient Boosting (TimeSeriesSplit CV)
7. LSTM on hourly data (one-step-ahead and recursive multi-step)
8. Results, COVID-19 structural-break analysis, and a 52-week future forecast

### Requirements to run

- **Internet connection.** Weekly temperature for Berlin (52.52°N, 13.41°E) is
  fetched at runtime from the [Open-Meteo ERA5 archive API](https://open-meteo.com/).
  No API key is needed.
- **~130 MB free disk** for the OPSD CSV.
- A machine capable of training a small LSTM (CPU is sufficient; a GPU speeds up
  the neural section but is not required).

---

## Key details

- **Weekly aggregation** keeps only weeks with ≥167 hours to preserve an equidistant
  index for SARIMA (a subtle but important choice discussed in the report).
- **No load lags** are used in the tree/neural conditional forecasts, so results are
  genuine 104-week-ahead multi-step forecasts rather than one-step-ahead.
- **Leakage control:** temperature lags are shifted before the train/test split;
  the LSTM scaler is fit on training rows only; tree models use `TimeSeriesSplit`.
- Temperature-based forecasts are **conditional (explanatory)** forecasts, since
  the test-period weather is observed rather than predicted — they represent an
  optimistic upper bound on operational accuracy.

---

## Data sources

- **Load:** Open Power System Data, *Data Package Time Series*, version 2020-10-06
  (`DE_load_actual_entsoe_transparency`, ENTSO-E Transparency Platform).
- **Temperature:** Open-Meteo ERA5 archive, daily mean 2 m temperature, Berlin.

## License

This project was produced as MSc coursework. Please check with the author before reuse.
