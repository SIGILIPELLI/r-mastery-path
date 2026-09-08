# 10 · Capstone Project

This project pulls together every Level 4 module into one small but
complete system: a `data.table`-backed pipeline that generates and
aggregates data, a linear model fit on it, a `plumber` API that serves
both the aggregate and a live prediction, and a `testthat` suite that
checks the pipeline logic independently of the API. It's the shape a
real internal analytics service tends to take — generate/ingest, model,
serve, test — just small enough to build and run end to end in one
sitting.

## Project layout

```
capstone/
  R/
    pipeline.R   # data generation, aggregation, model fitting
    api.R        # plumber routes that use pipeline.R
  tests/
    testthat.R
    testthat/
      test-pipeline.R
```

## The pipeline: generate, aggregate, model

`R/pipeline.R` holds three functions with no web or I/O concerns at
all — everything in this file is pure and independently testable:

```r
library(data.table)

generate_sales_data <- function(n = 50000, seed = 42) {
  set.seed(seed)
  dt <- data.table(
    order_id = seq_len(n),
    region   = sample(c("north", "south", "east", "west"), n, replace = TRUE),
    product  = sample(c("widget", "gadget", "gizmo"), n, replace = TRUE),
    units    = sample(1:10, n, replace = TRUE),
    price    = round(runif(n, 5, 100), 2)
  )
  dt[, revenue := units * price]
  dt
}

summarize_sales <- function(dt) {
  dt[, .(total_revenue = sum(revenue), total_units = sum(units), n_orders = .N),
     by = .(region, product)][order(-total_revenue)]
}

fit_revenue_model <- function(dt) {
  lm(revenue ~ region + product + units, data = dt)
}
```

`generate_sales_data()` stands in for an ingestion step (in a real
project this would read a file or query a database instead); everything
downstream — aggregation, modeling, serving — doesn't care where the
`data.table` came from.

## Testing the pipeline in isolation

Because `pipeline.R` has no server or randomness that isn't seeded, it's
fully testable without ever starting the API:

```r
test_that("generate_sales_data produces expected shape", {
  dt <- generate_sales_data(n = 100)
  expect_equal(nrow(dt), 100)
  expect_true(all(c("order_id","region","product","units","price","revenue") %in% names(dt)))
  expect_equal(dt$revenue, dt$units * dt$price)
})

test_that("summarize_sales aggregates correctly", {
  dt <- generate_sales_data(n = 1000)
  summary <- summarize_sales(dt)
  expect_equal(sum(summary$total_revenue), sum(dt$revenue))
  expect_equal(sum(summary$n_orders), nrow(dt))
})
```

```r
test_file("tests/testthat/test-pipeline.R")
```

```
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 5 ]
```

Both tests check an invariant that must hold regardless of the random
seed: total revenue is conserved between the raw rows and the
aggregate, and every row is accounted for in some group's `n_orders`.
That's a stronger, more durable test than hardcoding expected numbers,
since it still catches a broken aggregation even if `generate_sales_data()`
changes later.

## Fitting the model

```r
sales <- generate_sales_data()
fit <- fit_revenue_model(sales)
summary(fit)
```

```
Call:
lm(formula = revenue ~ region + product + units, data = dt)

Coefficients:
              Estimate Std. Error t value Pr(>|t|)
(Intercept)    -0.5651     2.3720  -0.238   0.8117
regionnorth     2.0820     2.1467   0.970   0.3321
regionsouth     3.6908     2.1587   1.710   0.0873 .
regionwest     -1.2586     2.1619  -0.582   0.5605
productgizmo   -0.1495     1.8706  -0.080   0.9363
productwidget   0.3621     1.8688   0.194   0.8464
units          52.2527     0.2644 197.619   <2e-16 ***
---
Residual standard error: 170.6 on 49993 degrees of freedom
Multiple R-squared:  0.4386,	Adjusted R-squared:  0.4385
```

The data was generated with region and product having no real effect on
revenue (only `units × price` does), which the fit confirms honestly:
`units` is wildly significant (`t = 197.6`) while every `region` and
`product` coefficient is statistically indistinguishable from zero
(`p > 0.05` for all of them). A capstone project's model doesn't need to
find a dramatic effect to be a good example — it needs to report
correctly on the data it was actually given, including a null result.

## Serving it with plumber

`R/api.R` wraps the same pipeline functions as HTTP routes:

```r
library(plumber)
source("pipeline.R")

sales <- generate_sales_data()
model <- fit_revenue_model(sales)

#* Summary of revenue by region and product
#* @get /summary
function() {
  summarize_sales(sales)
}

#* Predict revenue for a given region, product, and unit count
#* @param region:character
#* @param product:character
#* @param units:numeric
#* @get /predict
function(region, product, units) {
  units <- as.numeric(units)
  newdata <- data.frame(region = region, product = product, units = units)
  pred <- predict(model, newdata = newdata, interval = "confidence")
  list(predicted_revenue = round(pred[1, "fit"], 2))
}

#* Health check
#* @get /health
function() {
  list(status = "ok", n_orders = nrow(sales))
}
```

`sales` and `model` are computed once when the file is sourced (Module
2's lesson on plumber startup behavior) — every request reuses the same
fitted model instead of refitting it per call.

## Running it

```r
pr <- plumb("api.R")
pr$run(port = 8321, host = "127.0.0.1")
```

```bash
curl "http://127.0.0.1:8321/health"
# {"status":["ok"],"n_orders":[50000]}

curl "http://127.0.0.1:8321/summary"
# [{"region":"north","product":"gizmo","total_revenue":1227833.78,"total_units":23401,"n_orders":4222},
#  {"region":"north","product":"gadget","total_revenue":1225349.61,"total_units":23352,"n_orders":4228},
#  {"region":"south","product":"widget","total_revenue":1223180.8,"total_units":22965,"n_orders":4197}, ...]

curl "http://127.0.0.1:8321/predict?region=north&product=widget&units=5"
# {"predicted_revenue":[263.14]}
```

All three routes ran end to end against the real fitted model on this
machine — `/health` reports the row count from `generate_sales_data()`,
`/summary` is the same aggregation validated by the test suite, and
`/predict` calls `predict()` on the in-memory `lm` object with a fresh
`region`/`product`/`units` combination.

## How the pieces connect

| Layer | Module | File |
|---|---|---|
| Fast aggregation on the full dataset | 08 · data.table | `pipeline.R` — `summarize_sales()` |
| Statistical model | 01 · Advanced Statistical Methods | `pipeline.R` — `fit_revenue_model()` |
| HTTP serving | 02 · plumber | `api.R` |
| Automated correctness checks | 04 · Testing at Scale & CI | `tests/testthat/test-pipeline.R` |
| Pinned dependencies for reproducibility | 06 · renv | `renv.lock` (add via `renv::snapshot()`) |
| Packaging for reuse elsewhere | 09 · CRAN Basics | move `pipeline.R`'s functions into an installable package |

Wiring the test file into a CI workflow (Module 4's GitHub Actions
example) and running `renv::snapshot()` on the project (Module 6) are
the two steps that would take this from "runs on my machine" to "runs
the same way on any machine" — both are direct extensions of what's
already here.

## How It Actually Works

Wiring together simulation, modeling, and a plumber API in one pipeline
means each stage's output has to survive as a **serialized R object**
between process boundaries: the fitted model from the modeling stage is
written to disk with `saveRDS()` (which serializes the object's full
internal representation — including, for an `lm`/`glm` object, its stored
QR decomposition from Module 3 — into R's own binary serialization
format), and the plumber API process, running independently, calls
`readRDS()` to reconstruct that exact object in its own memory rather than
refitting the model on every request.

This separation matters mechanically: the plumber process never needs the
original training data in memory at request time, only the already-fitted
model object and a `predict()` call, which is why `predict.lm()` can score
new data using just the fitted coefficients and QR factor stored inside
the serialized object — no re-solving of the original least-squares system
is needed per request. The "how the pieces connect" boundary in this
project is exactly the boundary between R-object serialization
(`saveRDS`/`readRDS`) on one side and HTTP/JSON serialization (via
`jsonlite`, as in Module 8) on the other — two entirely different encoding
mechanisms bridging the pipeline's stages.

## Stretch goals

1. Replace `generate_sales_data()` with a real CSV or database read, and
   add a test that the pipeline still produces the same aggregate
   invariants (`sum(revenue)` conserved, `sum(n_orders) == nrow(dt)`) on
   real data.
2. Add a `/retrain` `@post` route that accepts a new dataset in the
   request body, refits the model, and replaces the in-memory `model`
   object — then write a test (using `plumber`'s `pr_mock()` request
   simulation, or a live `curl`) confirming a prediction changes after
   retraining.
3. Turn `pipeline.R` into an installable package (Module 9): move its
   three functions into a package's `R/` directory, add roxygen docs and
   a `DESCRIPTION`, run `R CMD check --as-cran`, and have `api.R` call
   `library(yourpkg)` instead of `source("pipeline.R")`.
4. Add a `renv.lock` for the whole project via `renv::snapshot()`
   (Module 6), and a GitHub Actions workflow (Module 4) that installs
   from the lockfile and runs `testthat::test_dir()` with
   `stop_on_failure = TRUE` on every push.
