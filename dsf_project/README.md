# Predicting monthly asylum applications in Switzerland

**Collaborators:** Louis Bitar, Thibaud Marti, Axel Walther, Fedor Kleymenov (Group 11)

**Goal:** Forecast the number of asylum applications in Switzerland one to three months ahead, to support the capacity planning of federal reception places.

## Run

Goal: reproduce all outputs (figures and tables) from the raw data.

- Install the packages in `requirements.txt` (`python -m pip install -r requirements.txt` from the project root).
- Open `code/main.ipynb`, restart the kernel and run all cells.
- Use `code/` as the working directory. If you start at the project root, run `%cd code` in a notebook cell first.

## Workflow

1. `01_preprocess.ipynb`: load and check the monthly series, split the months by time (first training window, backtest, test), explore the months before the test period, save `data/processed/asylum_clean.csv`.
2. `02_estimate.ipynb`: (to come) baselines and models in an expanding-window backtest.
3. `03_evaluate.ipynb`: (to come) errors in persons per month, model comparison, final test.

## Data

- `data/raw/eurostat_asylum_ch_monthly.csv`: Eurostat `migr_asyappctzm`, Switzerland, all citizenships, 2008-01 to 2026-09. Source, download date and checks against SEM's figures in `data/raw/SOURCES.md`.
