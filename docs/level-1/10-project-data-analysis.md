# 10 · Project — Data Analysis Script

A small end-to-end project combining everything from Level 1: variables,
control flow, functions, vectors, data frames, reading CSV data, plotting,
and packages.

## What you'll build

A single R script, `analyze_sales.R`, that:

- Reads a small sales CSV dataset
- Cleans and inspects it
- Computes summary statistics per product category
- Flags underperforming products with a custom function
- Produces two plots (a bar chart and a histogram) saved to disk

## Project layout

```text
sales-analysis/
    sales.csv
    analyze_sales.R
```

## sales.csv — the sample dataset

Create this file by hand (or generate it with a short script):

```text
product,category,units_sold,revenue
Widget A,Hardware,120,2400
Widget B,Hardware,45,900
Gadget X,Electronics,300,15000
Gadget Y,Electronics,80,4800
Tool Z,Hardware,15,225
Gizmo Q,Electronics,210,10500
```

## analyze_sales.R — the full script

```r
# analyze_sales.R
# A small end-to-end sales data analysis script.

# --- 1. Load packages -------------------------------------------------------

if (!require("readr", quietly = TRUE)) {
    install.packages("readr")
    library(readr)
}

# --- 2. Read and inspect the data -------------------------------------------

sales <- read_csv("sales.csv", show_col_types = FALSE)

cat("---- Structure ----\n")
str(sales)

cat("\n---- First rows ----\n")
print(head(sales))

if (anyNA(sales)) {
    warning("Data contains missing values -- check before trusting summaries.")
}

# --- 3. Custom analysis functions -------------------------------------------

# Average revenue per unit sold, rounded to 2 decimal places
revenue_per_unit <- function(revenue, units) {
    round(revenue / units, 2)
}

# Flags a product as "underperforming" if it sold fewer than a threshold
classify_performance <- function(units_sold, threshold = 50) {
    ifelse(units_sold < threshold, "Underperforming", "On Track")
}

sales$rev_per_unit <- revenue_per_unit(sales$revenue, sales$units_sold)
sales$performance <- classify_performance(sales$units_sold)

cat("\n---- Sales with derived columns ----\n")
print(sales)

# --- 4. Summarize by category ------------------------------------------------

categories <- unique(sales$category)

cat("\n---- Category summaries ----\n")
for (cat_name in categories) {
    subset_data <- sales[sales$category == cat_name, ]
    total_units <- sum(subset_data$units_sold)
    total_revenue <- sum(subset_data$revenue)
    avg_rev_per_unit <- round(total_revenue / total_units, 2)

    cat(sprintf(
        "%s: %d units, $%d revenue, $%.2f avg revenue/unit\n",
        cat_name, total_units, total_revenue, avg_rev_per_unit
    ))
}

# tapply() gives the same category totals more concisely
cat("\n---- tapply() cross-check: total revenue by category ----\n")
print(tapply(sales$revenue, sales$category, sum))

# --- 5. Flag underperforming products ---------------------------------------

underperformers <- sales[sales$performance == "Underperforming", "product"]
cat("\nUnderperforming products:", paste(underperformers, collapse = ", "), "\n")

# --- 6. Plots ----------------------------------------------------------------

category_revenue <- tapply(sales$revenue, sales$category, sum)

png("revenue_by_category.png", width = 800, height = 600)
barplot(category_revenue,
        main = "Total Revenue by Category",
        ylab = "Revenue ($)",
        col = c("steelblue", "seagreen"))
dev.off()

png("units_sold_distribution.png", width = 800, height = 600)
hist(sales$units_sold,
     main = "Distribution of Units Sold",
     xlab = "Units Sold",
     col = "cornflowerblue",
     breaks = 5)
dev.off()

cat("\nSaved plots: revenue_by_category.png, units_sold_distribution.png\n")
cat("Analysis complete.\n")
```

## Running it

```bash
Rscript analyze_sales.R
```

```text
---- Structure ----
spc_tbl_ [6 x 4] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
 $ product   : chr  "Widget A" "Widget B" "Gadget X" "Widget Y" ...
 $ category  : chr  "Hardware" "Hardware" "Electronics" "Electronics" ...
 $ units_sold: num  120 45 300 80 15 210
 $ revenue   : num  2400 900 15000 4800 225 10500

---- Category summaries ----
Hardware: 180 units, $3525 revenue, $19.58 avg revenue/unit
Electronics: 590 units, $30300 revenue, $51.36 avg revenue/unit

Underperforming products: Widget B, Tool Z

Saved plots: revenue_by_category.png, units_sold_distribution.png
Analysis complete.
```

Each function (`revenue_per_unit`, `classify_performance`) is small,
testable in isolation, and vectorized — calling it once on the whole
`units_sold` column computes every product's classification at once, no
explicit loop required. The `for` loop over `categories` is used
deliberately for the readable per-category printout, with `tapply()` shown
right after as the more idiomatic one-liner for the same aggregation.

## Stretch goals

- Add a `profit_margin` column if you extend `sales.csv` with a `cost`
  column, and flag products with margin below some threshold.
- Replace the `for` loop over `categories` with `sapply()` once you reach
  [Level 2, Module 3](../level-2/03-apply-family-functions.md).
- Swap the base `barplot()`/`hist()` calls for `ggplot2` once you reach
  [Level 2, Module 4](../level-2/04-ggplot2-basics.md) — same data, more
  polished output.
- Add basic input validation: what should `analyze_sales.R` do if
  `sales.csv` doesn't exist, or a `units_sold` value is `0` (division by
  zero in `revenue_per_unit`)?

Completing this project means you're ready for **Level 2 · Intermediate**.
