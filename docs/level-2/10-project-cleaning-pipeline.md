# 10 · Project — Data Cleaning & Reporting Pipeline

A small end-to-end project combining everything from Level 2: cleaning
messy real-world data with dplyr and stringr, testing the cleaning logic
with testthat, and visualizing the result with ggplot2.

## What you'll build

A two-file project:

```text
orders-pipeline/
    R/
        pipeline.R          # clean_orders() and summarize_by_category()
    tests/
        testthat/
            test-pipeline.R  # unit tests for both functions
    run_pipeline.R           # the main script that ties it all together
```

This mirrors [Module 7](07-functions-package-structure.md)'s recommended
`R/` layout and [Module 8](08-testing-testthat.md)'s `tests/testthat/`
convention — the same structure a real installable package would use.

## The messy raw dataset

A small orders table with every problem [Module 2](02-data-cleaning.md)
covered: inconsistent name/category casing and whitespace, prices stored
as text (some with a `$` sign), a missing quantity, an invalid negative
quantity, and blank/missing region values.

```r
orders_raw <- data.frame(
    order_id = 1:12,
    customer_name = c(
        "alice johnson", "BOB SMITH", "  Carol Lee", "dave Kim", "Eve Chen",
        "frank o'brien", "GRACE HALL", "Heidi Young", "  ivan Petrov",
        "Judy Nguyen", "Karl Weber", "liam Foster"
    ),
    category = c(
        "electronics", "Electronics", "GROCERY", "grocery", "Clothing",
        "clothing ", "Electronics", "Grocery", "Clothing", "electronics",
        "GROCERY", "Clothing"
    ),
    quantity = c(2, 1, 5, NA, 3, -1, 2, 4, 1, 6, 3, 2),
    unit_price = c(
        "$249.99", "899.00", "$4.50", "12.00", "$59.99", "$18.00",
        "1999.99", "$3.75", "45.00", "$199.99", "$5.25", "$29.99"
    ),
    order_date = c(
        "2024-01-15", "2024-01-22", "2024-02-03", "2024-02-10", "2024-02-14",
        "2024-02-20", "2024-03-01", "2024-03-05", "2024-03-11", "2024-03-15",
        "2024-03-19", "2024-03-25"
    ),
    region = c(
        "West", "East", "West", "", "East", "South", "West", "East", NA,
        "South", "West", "East"
    ),
    stringsAsFactors = FALSE
)
```

## R/pipeline.R — the cleaning and summarizing functions

```r
# R/pipeline.R
library(dplyr)
library(stringr)
library(lubridate)

#' Clean a raw orders data frame
#'
#' @param df Raw orders data frame with messy text, prices, and dates.
#' @return A cleaned data frame: standardized text columns, numeric price,
#'   parsed dates, invalid quantities flagged (not silently dropped), and
#'   outlier prices flagged via the IQR rule.
clean_orders <- function(df) {
    df |>
        mutate(
            customer_name = str_trim(customer_name),
            customer_name = str_to_title(customer_name),
            category = str_trim(category),
            category = str_to_title(category),
            region = ifelse(region == "" | is.na(region), "Unknown", region),
            unit_price = as.numeric(str_replace(unit_price, "\\$", "")),
            order_date = ymd(order_date),
            quantity_is_invalid = is.na(quantity) | quantity < 0,
            quantity = ifelse(quantity_is_invalid, NA, quantity)
        ) |>
        mutate(
            revenue = quantity * unit_price,
            price_iqr_low = quantile(unit_price, 0.25, na.rm = TRUE) -
                1.5 * IQR(unit_price, na.rm = TRUE),
            price_iqr_high = quantile(unit_price, 0.75, na.rm = TRUE) +
                1.5 * IQR(unit_price, na.rm = TRUE),
            price_is_outlier = !is.na(unit_price) &
                (unit_price < price_iqr_low | unit_price > price_iqr_high)
        ) |>
        select(-price_iqr_low, -price_iqr_high)
}

#' Summarize cleaned orders by category
#'
#' @param cleaned_df A data frame already processed by clean_orders().
#' @return One row per category: order count, total revenue, average
#'   revenue per order. Rows with invalid quantity or missing revenue are
#'   excluded from the totals so they don't silently distort them.
summarize_by_category <- function(cleaned_df) {
    cleaned_df |>
        filter(!quantity_is_invalid, !is.na(revenue)) |>
        group_by(category) |>
        summarise(
            n_orders = n(),
            total_revenue = sum(revenue),
            avg_revenue = round(mean(revenue), 2),
            .groups = "drop"
        ) |>
        arrange(desc(total_revenue))
}
```

Notice `quantity_is_invalid` is a **flag column**, not a silent deletion —
[Module 2](02-data-cleaning.md#detecting-outliers) covered this same
principle for outliers: mark the problem so it's visible and auditable,
rather than quietly dropping rows, which would make `nrow()` before and
after cleaning misleadingly different with no explanation in the data
itself.

## tests/testthat/test-pipeline.R — verifying the cleaning logic

```r
library(testthat)
library(dplyr)
source("../../R/pipeline.R")

fixture <- data.frame(
    order_id = 1:4,
    customer_name = c("alice johnson", "  BOB SMITH", "Carol Lee", "dave kim"),
    category = c("electronics", "Electronics", "grocery ", "Grocery"),
    quantity = c(2, -1, 5, NA),
    unit_price = c("$10.00", "20.00", "$5.00", "8.00"),
    order_date = c("2024-01-01", "2024-01-02", "2024-01-03", "2024-01-04"),
    region = c("West", "", NA, "East"),
    stringsAsFactors = FALSE
)

test_that("clean_orders standardizes text case and whitespace", {
    cleaned <- clean_orders(fixture)
    expect_equal(cleaned$customer_name, c("Alice Johnson", "Bob Smith", "Carol Lee", "Dave Kim"))
    expect_equal(cleaned$category, c("Electronics", "Electronics", "Grocery", "Grocery"))
})

test_that("clean_orders converts price strings to numeric", {
    cleaned <- clean_orders(fixture)
    expect_equal(cleaned$unit_price, c(10, 20, 5, 8))
    expect_type(cleaned$unit_price, "double")
})

test_that("clean_orders flags negative and missing quantity without dropping rows", {
    cleaned <- clean_orders(fixture)
    expect_equal(nrow(cleaned), 4)   # no rows silently dropped
    expect_equal(cleaned$quantity_is_invalid, c(FALSE, TRUE, FALSE, TRUE))
    expect_true(is.na(cleaned$quantity[2]))   # invalid quantity nulled out, not left negative
})

test_that("clean_orders fills missing/blank region with 'Unknown'", {
    cleaned <- clean_orders(fixture)
    expect_equal(cleaned$region, c("West", "Unknown", "Unknown", "East"))
})

test_that("summarize_by_category excludes invalid-quantity rows from totals", {
    cleaned <- clean_orders(fixture)
    summary_df <- summarize_by_category(cleaned)

    expect_equal(nrow(summary_df), 2)   # only Electronics and Grocery have valid rows
    electronics_row <- summary_df |> filter(category == "Electronics")
    expect_equal(electronics_row$n_orders, 1)
    expect_equal(electronics_row$total_revenue, 20)   # order 1: 2 * $10.00

    grocery_row <- summary_df |> filter(category == "Grocery")
    expect_equal(grocery_row$n_orders, 1)
    expect_equal(grocery_row$total_revenue, 25)   # order 3: 5 * $5.00
})

test_that("summarize_by_category sorts by total_revenue descending", {
    cleaned <- clean_orders(fixture)
    summary_df <- summarize_by_category(cleaned)
    expect_true(all(diff(summary_df$total_revenue) <= 0))
})
```

## run_pipeline.R — the main script

```r
# run_pipeline.R
library(dplyr)
library(ggplot2)
source("R/pipeline.R")

orders_clean <- clean_orders(orders_raw)   # orders_raw as defined above

cat("---- Rows flagged with invalid quantity ----\n")
print(orders_clean |> filter(quantity_is_invalid) |>
          select(order_id, customer_name, quantity))

cat("\n---- Rows flagged as price outliers ----\n")
print(orders_clean |> filter(price_is_outlier) |>
          select(order_id, customer_name, unit_price))

category_summary <- summarize_by_category(orders_clean)
cat("\n---- Revenue summary by category ----\n")
print(category_summary)

p <- ggplot(category_summary, aes(x = reorder(category, -total_revenue), y = total_revenue)) +
    geom_col(fill = "steelblue") +
    labs(title = "Total Revenue by Category", x = "Category", y = "Revenue ($)") +
    theme_minimal()

ggsave("revenue_by_category.png", plot = p, width = 7, height = 5, dpi = 150)
cat("\nSaved plot: revenue_by_category.png\n")
```

## Running it

```bash
Rscript run_pipeline.R
```

```text
---- Rows flagged with invalid quantity ----
  order_id customer_name quantity
1        4      Dave Kim       NA
2        6 Frank O'brien       NA

---- Rows flagged as price outliers ----
  order_id customer_name unit_price
1        2     Bob Smith     899.00
2        7    Grace Hall    1999.99

---- Revenue summary by category ----
# A tibble: 3 x 4
  category    n_orders total_revenue avg_revenue
  <chr>          <int>         <dbl>       <dbl>
1 Electronics        4        6599.       1650.
2 Clothing           3         285.         95.0
3 Grocery            3          53.2        17.8

Saved plot: revenue_by_category.png
```

Notice `"Frank O'brien"` — `str_to_title()` capitalizes the first letter of
each *whitespace-separated* word, and an apostrophe doesn't count as a word
break, so `"o'brien"` becomes `"O'brien"`, not the more natural `"O'Brien"`.
This is a genuine, common stringr quirk with names containing apostrophes
or hyphens — worth a manual override (`str_replace()` on known patterns) if
your real data has enough such names to matter.

```r
testthat::test_dir("tests/testthat")
```

```text
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 14 ]
```

## Stretch goals

- Add a `discount_pct` column to `orders_raw` and thread it through
  `clean_orders()` and `revenue` calculation — write a new test confirming
  the discounted revenue math before trusting it in the summary.
- Replace the single IQR-based `price_is_outlier` flag with a per-category
  IQR check (an electronics price of $1999.99 might be entirely normal for
  that category but an outlier overall) — group by `category` before
  computing the IQR bounds.
- Extend `summarize_by_category()` to also break down by `region` using a
  second `group_by()` key, and add a `facet_wrap(~ region)` to the ggplot2
  chart per [Module 4](04-ggplot2-basics.md).
- Turn `quantity_is_invalid` and `price_is_outlier` into a single combined
  `data_quality_issues` count per row, and add a test asserting the total
  count of flagged rows matches your expectation for the fixture data.

Completing this project means you're ready for **Level 3 · Advanced**.
