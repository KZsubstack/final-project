---
output: github_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = FALSE, warning = FALSE, message = FALSE)
library(tidyverse)
library(broom)

airport <- bind_rows(
  read_csv("data/airport_traffic_2025.csv"),
  read_csv("data/airport_traffic_2026.csv")
) |>
  mutate(
    MONTH_MON = fct_reorder(MONTH_MON, MONTH_NUM),
    YEAR = factor(YEAR)
  )

n_rec <- nrow(airport)
n_apts <- n_distinct(airport$APT_ICAO)
n_states <- n_distinct(airport$STATE_NAME)
mean_daily <- round(mean(airport$FLT_TOT_1, na.rm = TRUE), 0)
median_daily <- round(median(airport$FLT_TOT_1, na.rm = TRUE), 0)

model2_log <- lm(log(FLT_TOT_1 + 1) ~ MONTH_MON + APT_ICAO + YEAR, data = airport)
m2_glance <- glance(model2_log)
jul_coef <- tidy(model2_log) |> filter(term == "MONTH_MONJUL") |> pull(estimate)
jul_pct <- round((exp(jul_coef) - 1) * 100, 1)
```

# Seasonality and Geography of Airport Flight Traffic

**Math 225 Final Project Kaden Zhu**

## Research question

Are daily flight totals at international airports systematically
related to month of year and country, after accounting for which
specific airports are in the sample? In other words, is there a
seasonal pattern in air traffic that holds within airports, and how
much of the apparent country-level variation is really a story about
where the major hubs happen to sit?

## Dataset

The dataset is a Kaggle release of EUROCONTROL-style airport traffic
statistics covering daily IFR flight movements at international
airports across 2025 and 2026. Each row is one airport on one day.
After combining the two yearly files I had `r format(n_rec, big.mark = ",")`
records spanning `r n_apts` airports across `r n_states` countries, with
heavy European coverage and a handful of neighboring states. The
outcome variable is `FLT_TOT_1`, total daily flight movements. The
key predictors are month abbreviation and country, with airport ICAO
code and calendar year as controls.

## Methodology

Exploratory analysis covered the distribution of the outcome,
month-by-month and country-by-country summary tables, and three
visualizations: a seasonal line plot, a country-level boxplot, and a
bar chart of the fifteen busiest airports. The histogram revealed a
strong right skew driven by a small number of mega-hubs, which
motivated a log transformation in the modeling stage.

The main analysis fits two linear regressions, deliberately
side-by-side. Model 1 regresses daily flights on month, country, and
year without controlling for airport identity. Model 2 swaps country
for airport-level fixed effects because every airport sits in
exactly one country, including both would be perfectly collinear, so
airport replaces country. The contrast between the two models is the
analytical point: if the seasonal effect is real and within-airport,
its coefficients should stay stable across both. If the country
effect is mostly a hub-composition artifact, its apparent magnitude
in model 1 should be absorbed by the airport dummies in model 2.
Finally, the log-transformed version of model 2 serves as a
robustness check against the right tail in the outcome.

## Key findings

The dataset shows a typical airport-day at around `r median_daily` IFR
movements, with a mean of `r mean_daily` pulled upward by a few
mega-hubs running over a thousand movements per day. The seasonal
pattern is unmistakable in both years: average daily traffic climbs
from a winter low through spring, crests in mid-summer, and drops
back through autumn, with both calendar years tracing very similar
shapes.

The two-model comparison is the substantive payoff. Month
coefficients are remarkably stable across the model with country
control and the model with airport fixed effects, which means the
seasonal signal is genuinely within-airport variation, not a story
about which countries happen to have summer records. The country
contrast is the opposite case: countries that looked like
"high-traffic countries" in the boxplots are revealed to be
single-hub stories. Denmark looks busy because of Copenhagen.
Switzerland looks busy because of Zurich. In the log-scale model,
July traffic at the typical airport runs roughly `r jul_pct`% above
January, holding airport and year constant. The model 2 fit has an
R² of `r round(m2_glance$r.squared, 3)`, which is dominated by the
airport fixed effects i.e. most of the variance in daily flights
is between airports, with seasonality a smaller but very consistent
within-airport effect.

## Limitations

Four constraints bound the conclusions. First, geographic coverage
is essentially European, so findings generalize to EUROCONTROL
airspace rather than to global aviation. Second, the outcome lumps
passenger, cargo, business, and general IFR aviation into a single
daily count, which means the regression cannot speak to mode mix or
to the tourism-specific story the seasonal pattern suggests. Third,
two calendar years confirm a stable seasonal pattern but cannot
separate "seasonality" from "year-specific events" such as strikes
or weather disruptions; with only two years, robustness against
one-off shocks is limited. Fourth, the analysis is observational,
so even with airport fixed effects the regression describes
associations rather than causes.

## Repository contents

```
.
├── README.md                rendered executive summary
├── README.Rmd               source for this file
├── analysis.Rmd             full EDA, regression, diagnostics
├── presentation.Rmd         xaringan slides
├── presentation.html        rendered slides
├── SPEAKER_NOTES.md         five-minute speaker script
├── data/
│   ├── airport_traffic_2025.csv
│   ├── airport_traffic_2026.csv
│   └── README.md            data dictionary
└── proposal/
    └── proposal.Rmd         original proposal, EDA expanded per feedback
```
