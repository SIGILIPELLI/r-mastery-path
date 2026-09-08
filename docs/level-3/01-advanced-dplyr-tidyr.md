# 01 · Advanced dplyr/tidyr

Level 1-2 covered single-table `dplyr` verbs and basic `pivot_longer()` /
`pivot_wider()`. Real analysis work almost always involves more than one
table, and "tidy" data doesn't always arrive tidy — this module covers
joins, multi-column pivots, `rowwise()` and `across()`, and the
non-standard evaluation (NSE) trap that catches nearly everyone the first
time they try to write a reusable `dplyr` function.

```r
library(dplyr)
library(tidyr)
```

## Joins

```r
orders <- tibble(
  order_id = 1:6,
  customer_id = c(1, 2, 1, 3, 2, 1),
  product = c("Widget", "Gadget", "Widget", "Gizmo", "Widget", "Gadget"),
  qty = c(2, 1, 3, 1, 2, 5)
)
customers <- tibble(
  customer_id = c(1, 2, 3),
  name = c("Ana", "Ben", "Cy"),
  region = c("East", "West", "East")
)

inner_join(orders, customers, by = "customer_id")
# order_id customer_id product   qty name  region
#        1           1 Widget      2 Ana   East
#        2           2 Gadget      1 Ben   West
#        3           1 Widget      3 Ana   East
#        4           3 Gizmo       1 Cy    East
#        5           2 Widget      2 Ben   West
#        6           1 Gadget      5 Ana   East
```

`inner_join()` keeps only rows with a match on both sides. `left_join()`
keeps every row from the left table and fills unmatched right-side columns
with `NA`:

```r
customers2 <- tibble(customer_id = c(1, 2), name = c("Ana", "Ben"), region = c("East", "West"))
left_join(orders, customers2, by = "customer_id")
# order 4 (customer_id 3) survives, with name and region as <NA>
```

`anti_join()` is the one people forget exists — it returns rows from the
left table that have **no** match in the right table, which makes it the
natural tool for "which orders reference a customer we don't have a
record for":

```r
anti_join(orders, customers2, by = "customer_id")
# A tibble: 1 x 4
#   order_id customer_id product qty
#          4           3 Gizmo     1
```

**Trap:** a join on a key with duplicate values on *either* side silently
fans out into a cartesian product of the matching rows — no warning is
raised for what looks like a one-to-many join gone wrong. If your row count
after a join is higher than you expected, check `customers %>%
count(customer_id) %>% filter(n > 1)` before blaming the join itself.

## Pivoting wide and long

```r
wide <- tibble(customer_id = c(1, 2, 3), Widget = c(5, 2, 0), Gadget = c(1, 3, 2))
long <- pivot_longer(wide, cols = c(Widget, Gadget), names_to = "product", values_to = "qty")
# customer_id product qty
#           1 Widget    5
#           1 Gadget    1
#           2 Widget    2
#           2 Gadget    3
#           3 Widget    0
#           3 Gadget    2

pivot_wider(long, names_from = product, values_from = qty)
# customer_id Widget Gadget
#           1      5      1
#           2      2      3
#           3      0      2
```

Real column names are rarely as clean as `Widget`/`Gadget`. When a wide
table encodes *two* pieces of information in each column name (a year and
a metric, say), `names_sep` plus the special `.value` sentinel splits them
apart and reshapes to long in one step:

```r
df_multi <- tibble(id = 1:2, `2023_sales` = c(100, 200), `2024_sales` = c(150, 250))
pivot_longer(df_multi, cols = -id, names_to = c("year", ".value"), names_sep = "_")
#    id year  sales
#     1 2023    100
#     1 2024    150
#     2 2023    200
#     2 2024    250
```

`.value` tells `pivot_longer()` "this chunk of the split column name is a
column name in the output, not a value" — everything else in `names_to`
becomes a regular key column.

## NA propagation in summaries

```r
df_na <- tibble(x = c(1, NA, 3), y = c(NA, 2, 3))
df_na %>% summarise(total = sum(x, na.rm = TRUE), total_bad = sum(x))
#   total total_bad
#       4        NA
```

**Trap:** this is base R's `NA` propagation rule, not a `dplyr` quirk, but
it bites hardest inside a long summarise pipeline where it's easy to
forget one `na.rm = TRUE` among many aggregations. A single `NA` in the
input silently turns the *entire* aggregate into `NA` — it does not throw
an error or a warning, so a wrong number can flow undetected into a report.
Audit every `sum()`, `mean()`, `min()`, `max()` in a summarise block for a
missing `na.rm` before trusting the output on real data.

## group_by / summarise and rowwise

```r
orders %>% group_by(customer_id) %>% summarise(n = n(), total_qty = sum(qty)) %>% ungroup()
#   customer_id n total_qty
#             1 3        10
#             2 2         3
#             3 1         1
```

Always `ungroup()` after a grouped summarise you're about to pipe further —
a lingering grouping structure silently changes the behavior of later
verbs (a `mutate()` after a forgotten `group_by()` computes per-group
instead of over the whole table).

`mutate()` operates column-wise by default, computing across all rows at
once. When you genuinely need one row at a time — combining several
columns' values per row rather than a whole column's — `rowwise()` switches
that behavior:

```r
df_rw <- tibble(a = 1:3, b = 4:6)
df_rw %>% rowwise() %>% mutate(total = sum(a, b)) %>% ungroup()
#   a b total
#   1 4     5
#   2 5     7
#   3 6     9
```

Without `rowwise()`, `sum(a, b)` would sum *both entire columns* into a
single recycled scalar — a common source of silently wrong per-row totals.

## across() for multi-column operations

```r
df_across <- tibble(a = c(1, 2, 3), b = c(4, 5, 6), grp = c("x", "x", "y"))
df_across %>% group_by(grp) %>% summarise(across(c(a, b), mean))
#   grp   a   b
#   x   1.5 4.5
#   y   3   6
```

`across()` replaces the old `_at`/`_if`/`_all` suffixed verb families
(`summarise_at`, `mutate_if`, etc.) from earlier `dplyr` versions — it's
the single, composable way to apply a function to multiple columns inside
any verb.

## The NSE trap: passing column names as strings

```r
col <- "qty"
orders %>% summarise(total = sum(col))
# Error in `summarise()`:
# Caused by error in `sum()`:
# ! invalid 'type' (character) of argument
```

`dplyr` verbs use non-standard evaluation — `qty` inside `sum(qty)` is
looked up as a column in `orders`, not as an R variable. When `col` holds
the *string* `"qty"`, `sum(col)` tries to sum the literal text `"qty"`, not
the column it names. The fix is `.data[[...]]`, which tells `dplyr` to
look up the column by the string value:

```r
orders %>% summarise(total = sum(.data[[col]]))
#   total
#      14
```

This matters the moment you write a function that takes a column name as
an argument — `my_summary <- function(df, col) df %>% summarise(total =
sum(.data[[col]]))` is the correct pattern; `sum(col)` inside that function
body fails the same way.

## Cheat sheet

| Task | Function |
|---|---|
| Keep only matching rows from both tables | `inner_join()` |
| Keep all left rows, fill unmatched with `NA` | `left_join()` |
| Rows in left with no match in right | `anti_join()` |
| Rows in left that *do* match right, left columns only | `semi_join()` |
| Wide → long | `pivot_longer()` |
| Long → wide | `pivot_wider()` |
| Split a compound column name while pivoting | `names_sep` + `.value` |
| Apply a function to many columns at once | `across()` |
| Operate one row at a time | `rowwise()` |
| Use a string as a column name inside a verb | `.data[[string_var]]` |

## How It Actually Works

`tidyr::pivot_longer()`/`pivot_wider()` reshape data by rebuilding the
underlying list-of-columns structure rather than moving values in place:
`pivot_longer()` computes, for every output row, which combination of
(original row, selected column) it corresponds to, then constructs new
vectors by indexing the original columns in that computed order — it's
conceptually a big `merge`/`join`-like index computation, not a literal
in-place transpose. This is also why pivoting on data with mismatched
types across the pivoted columns forces coercion into one common type in
the output's single "value" column, using the same coercion hierarchy
covered in Module 2.

`dplyr::group_by()` followed by a summary doesn't loop over groups one at a
time in R code — it computes a single grouping index (which rows belong to
which group, via hashing the grouping columns) once, then dispatches the
summary computation across all groups using vectorized or C-level grouped
aggregation where possible. `across()` works by taking your column
selection, resolving it to actual column names once via tidyselect's
non-standard evaluation, then applying your function(s) to each resolved
column and assembling the results back into a single data frame — it's
sugar over what would otherwise be a manual loop plus `bind_cols()`.

## Exercise

1. Build a small `orders` / `products` pair of tibbles where `products`
   has a duplicate `product_id`, join them, and confirm the row count
   fans out unexpectedly — then fix it with `distinct()` before joining.
2. Take a wide table with columns `q1_revenue`, `q1_units`, `q2_revenue`,
   `q2_units` and pivot it to long form with one row per quarter, using
   `names_sep` and `.value` so `revenue` and `units` remain separate
   columns.
3. Write a function `total_by_group(df, group_col, value_col)` that takes
   both column names as strings and returns a grouped sum using
   `.data[[...]]` — call it on `orders` grouping by `"customer_id"` and
   summing `"qty"`.
