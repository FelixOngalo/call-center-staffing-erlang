# Call-Centre Staffing in R: Arrival Forecasts, Erlang C and Service-Level Trade-offs

How many agents does a call centre need in each half hour to answer 80% of calls within 20 seconds? This project forecasts call arrivals, applies the Erlang C queueing model, and evaluates staffing policies on a held-out month using the *actual* arrivals. It shows why staffing to the forecast mean misses the target in a large share of half hours, even with a perfect forecast, and how quantile staffing trades extra agent-hours for reliability.

## Question

Sixteen weeks of call records (12 for training, 4 for testing). Which arrival forecast works best, how often does textbook staffing meet the 80/20 target, what does an oracle that knows the true arrival rate achieve, how much does quantile staffing cost and buy, and which level minimises total cost?

## Key findings

*(Fill in after running the scripts. Suggested: forecast accuracy against the Poisson floor; the share of half hours meeting 80/20 under mean staffing and under the oracle; agent-hours, call-weighted service level and missed calls for each policy; the cost-minimising quantile; the bootstrap interval for the improvement; the index of dispersion showing extra-Poisson variance.)*

## What is in it

- **Call simulator** with weekday and intraday patterns, a trend, a day-level random factor, demand spikes and lognormal handling times. Messy raw data: duplicate call IDs, test calls, impossible handle times and inconsistent timestamp formats.
- **Cleaning** with a log of every step, and construction of the day by half-hour arrivals matrix.
- **Exploration**: volume by weekday, intraday profile, handling time, abandonment and an **index of dispersion** test for extra-Poisson variance.
- **Three arrival forecasts**: naive (same slot last week), 4-week average and a trend-adjusted 8-week weekday profile, scored against actual arrivals and against the true rate.
- **Erlang C**, from scratch: Erlang B recursion, waiting probability, service level, and minimum agents for a target.
- **Staffing policies**: mean staffing, an oracle, and Poisson or negative-binomial quantile staffing.
- **Evaluation on the actual test arrivals**: call-weighted service level, share of half hours meeting target, agent-hours, missed calls and total cost.
- **Cost and service frontier** over demand quantiles, with the cost-minimising choice.
- **Bootstrap over test days** for the gain in reliability and cost.
- **A Quarto report** with the method, charts and limitations.

## Tools

R, dplyr, tidyr, tibble, ggplot2, knitr, Quarto. The queueing model is written in base R; no queueing or forecasting packages are needed.

## Reproduce

```r
install.packages(c("dplyr", "tidyr", "tibble", "ggplot2", "knitr"))
source("R/00_simulate_data.R")
source("R/01_clean_explore.R")
source("R/02_forecast_arrivals.R")
source("R/03_staffing_policies.R")
quarto::quarto_render("reports/report.qmd")
```

Run from the project root (open `call-center-staffing-erlang.Rproj`). The seeds are fixed, so results are reproducible.

## Project structure

```
call-center-staffing-erlang/
├── R/
│   ├── staffing_functions.R        # simulator, cleaning, Erlang C, forecasts
│   ├── 00_simulate_data.R
│   ├── 01_clean_explore.R
│   ├── 02_forecast_arrivals.R
│   └── 03_staffing_policies.R
├── data/
│   ├── raw/                        # call_log.csv (generated)
│   └── clean/                      # .rds results (generated)
├── reports/report.qmd
├── call-center-staffing-erlang.Rproj
└── README.md
```

## Limitations

- Each half hour is treated independently; no carry-over of queues, shifts or breaks.
- Identical, fully available agents; no shrinkage.
- Fixed average handling time; no variation by call type.
- No abandonment feedback, callbacks or retries.
- Cost parameters are assumptions; the best quantile moves with them.
- A 28-day test period with one demand spike gives wide intervals.
