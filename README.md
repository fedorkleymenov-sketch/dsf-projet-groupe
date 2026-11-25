# Predicting monthly asylum applications in Switzerland

Projet de groupe - Data Science Fundamentals (HSG)

**Collaborators:** Louis Bitar, Thibaud Marti, Axel Walther, Fedor Kleymenov (Group 11)

**Goal:** Forecast the number of asylum applications in Switzerland one to three months ahead, to support the capacity planning of federal reception places.

## Run

Goal: reproduce all outputs (figures and tables) from the raw data.

- Use Python 3.13 and install the packages in `requirements.txt` (`python -m pip install -r requirements.txt` from the project root). Install Python separately: pip installs only the packages.
- Open `code/main.ipynb`, restart the kernel and run all cells.
- Use `code/` as the working directory. If you start at the project root, run `%cd code` in a notebook cell first.

## Workflow

1. `01_preprocess.ipynb`: load and check the monthly series, split the months by time (first training window, backtest, test), explore the months before the test period, save `data/processed/asylum_clean.csv`.
1b. `01b_preprocess_indicators.ipynb`: load and check the Frontex detections by route, build the indicator (sum of the Central Mediterranean, Eastern Mediterranean and Western Balkan routes), state what is known at the cutoff, explore it before the test period, save `data/processed/routes_clean.csv`.
2. `02_estimate.ipynb`: seasonal naive, 12-month average, random walk (added on 23 November, after the test) and three (S)ARIMA models in an expanding-window backtest (141 origins, h = 1, 2, 3); saves `data/processed/backtest_forecasts.csv`. Runs in about 30 seconds.
2b. `02b_estimate_regression.ipynb`: linear regression, Ridge and Lasso on the series' own past (one model per horizon), same backtest; the penalty is chosen inside each training window. Saves `data/processed/backtest_forecasts_regression.csv`. Runs in about 75 seconds.
2c. `02c_estimate_indicator.ipynb`: the same three regressions plus two route features (changes of the indicator, using only months known at the cutoff), same backtest and same rules. Saves `data/processed/backtest_forecasts_indicator.csv`. Runs in about 60 seconds.
3. `03_evaluate.ipynb`: backtest errors of all twelve models (MAE in persons, RMSE in logs) per model and horizon; main comparison at h = 2; results without 2015-16 and 2020 and from 2022; year-by-year comparison; does the route indicator help (each regression against its own version with the routes)? Saves tables in `output/tables/` and figures in `output/figures/`. The test period is not used. Section 8 writes down the final model before the test: the linear regression. Section 9 (added on 23 November) compares it with the random walk.
4. `04_test.ipynb`: the chosen linear regression and the 12-month average on the test period (2024-10 to 2026-09), used once; forecasts made as in the backtest (refit at each origin on the months known then). The random walk is reported for information (added on 23 November; the forecasts of the chosen model are unchanged). Saves `data/processed/test_forecasts.csv`, `output/tables/04_test_errors.csv` and `output/figures/04_test_forecasts_h2.png`.
5. `05_ranges_and_forecast.ipynb` (added on 23 November): 80% and 90% ranges around the forecasts of the chosen model, from the quantiles of its past errors known at each origin; how often the actual number falls inside (backtest and test); the forecast from the newest data (origin 2026-09). Saves `output/tables/05_range_coverage.csv`, `output/tables/05_forecast_from_2026_09.csv` and `output/figures/05_ranges_h2.png`.

`main.ipynb` runs the eight notebooks in order (about 2.5 minutes). Rerunning it from the raw data reproduces the same outputs (no random steps; seed fixed at 42).

## Main result

Forecasting two months ahead, the linear regression on the series' own past misses by 188 applications per month on average in the test period (2024-10 to 2026-09), against 331 for the 12-month average (43% lower MAE). In the backtest (2013 to 2024-09) the gain was 32% (358 against 528). Against the best SARIMA model (369), a baseline in our plan, the gain is only 3%, and its RMSE of the log is slightly lower than ours; SARIMA was not part of the test. We changed our main comparison from the best SARIMA model to the best simple rule when we added the regressions, so we report all comparisons (`03_evaluate.ipynb`, section 3).

Against the random walk (repeat the last known month), added after the test, the gain is small in the backtest (379 against 358, 5% lower MAE, below the 10% of our success rule) and 30% in the test (269 against 188). Most of what can be predicted two months ahead is already in the last month.

The 80% range from past errors contains 80% of the actual numbers in the backtest (about ±30% of the forecast at h = 2), but it is too wide in the calm test period and too narrow at the start of the 2015 crisis.

The route indicator (Frontex) did not improve the forecasts. Its links with Swiss applications are strongest in the same month and one month later, and at the cutoff those months are already known (our timing rule assumes Frontex is one month behind SEM, based on the date of one file).

## Data

- `data/raw/frontex_detections_by_route_monthly.csv`: Frontex detections of illegal border-crossings by route, 2009-01 to 2026-09 (file dated 4 November 2026).
- `data/raw/eurostat_asylum_ch_monthly.csv`: Eurostat `migr_asyappctzm`, Switzerland, all citizenships, 2008-01 to 2026-09. Source, download date and checks against SEM's figures in `data/raw/SOURCES.md`.
