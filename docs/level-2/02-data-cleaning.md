# 02 · Data Cleaning

Real datasets are messy: missing values, inconsistent categories, numbers
stored as text, impossible outliers. Cleaning isn't a detour before "real"
analysis — it usually *is* most of the work, and a silent cleaning mistake
(like accidentally dropping half your rows) will quietly corrupt every
result downstream. This module covers the most common problems and the
idiomatic dplyr/base-R ways to handle them.

```r
library(dplyr)

messy <- data.frame(
    name = c("Alice", "bob", "CAROL", "Dave", "  Eve  "),
    age = c("23", "25", "unknown", "22", "29"),
    score = c(88, NA, 95, 999, 76),
    signup_date = c("2023-01-15", "2023-01-16", "2023-01-17", "2023-01-18", "2023-01-19"),
    stringsAsFactors = FALSE
)
```

## Finding missing values

`NA` is R's dedicated "missing" marker (distinct from `NULL`, which means
"no value at all", and `NaN`, which means "not a mathematically valid
number", e.g. `0/0`).

```r
is.na(messy$score)
# [1] FALSE  TRUE FALSE FALSE FALSE

sum(is.na(messy$score))     # count of NAs
# [1] 1

anyNA(messy)                 # is there ANY NA anywhere in the data frame?
# [1] TRUE

colSums(is.na(messy))        # NA count per column
#        name         age       score signup_date
#           0           0           1           0
```

Notice `colSums(is.na(messy))` reports 0 for `age`, even though it clearly
has a non-numeric "unknown" value — because that value is the *string*
`"unknown"`, not `NA`. That distinction matters for the next section.

## The NA-propagation trap

`NA` means "unknown," so almost any calculation touching an `NA` produces
`NA` — R refuses to guess:

```r
mean(messy$score)
# [1] NA

sum(c(1, 2, NA))
# [1] NA
```

This is correct behavior (an unknown value really does make the sum
unknown), but it surprises beginners who expect the `NA` to just be
skipped. Every base R aggregate function accepts `na.rm = TRUE` to opt into
skipping:

```r
mean(messy$score, na.rm = TRUE)
# [1] 314.5
```

(That number looks high because `score` includes the `999` outlier this
page investigates below — `na.rm = TRUE` skips the missing value, but it
does nothing about a valid-but-wrong value like `999`.)

**Trap:** `na.rm = TRUE` silently changes *what* you're computing (the mean
of the non-missing values, not the true mean including the missing ones) —
it doesn't recover missing information. Always ask *why* a value is missing
before defaulting to `na.rm = TRUE` everywhere; sometimes the right answer
is to investigate or impute, not average around the gap.

Comparisons with `NA` also propagate rather than evaluating to `FALSE`:

```r
NA == NA
# [1] NA        -- not TRUE! Use is.na() to test for missingness.

NA > 5
# [1] NA

if (NA > 5) "big" else "small"
# Error in if (NA > 5) "big" else "small" :
#   missing value where TRUE/FALSE needed
```

## Handling missing values with dplyr

```r
messy |> filter(!is.na(score))              # drop rows missing score
messy |> mutate(score = coalesce(score, 0)) # replace NA with a default
messy |> mutate(score = ifelse(is.na(score), mean(score, na.rm = TRUE), score)) # impute with the mean
```

`coalesce(x, replacement)` returns `x` unless it's `NA`, in which case it
returns `replacement` — the tidyverse equivalent of SQL's `COALESCE`.

## Type conversions

The `age` column above is stored as **character**, not numeric, because one
value is the text `"unknown"` rather than a number. Converting it exposes
exactly the value that doesn't fit:

```r
as.numeric(messy$age)
# Warning message:
# NAs introduced by coercion
# [1] 23 25 NA 22 29
```

This warning is R telling you something important, not just noise: any
value that can't be parsed as a number silently becomes `NA` rather than
erroring. **Always read that warning** — silently swallowing it (e.g. with
`suppressWarnings()`) hides real data problems.

```r
messy <- messy |>
    mutate(age = as.numeric(age))     # "unknown" -> NA, with a warning above
```

## Factor vs. character — a classic R gotcha

Older R code (and `data.frame()` before R 4.0) converts character columns
to **factors** by default — categorical values backed by integer codes.
Factors are useful for ordered categories (e.g. `"Low" < "Medium" <
"High"`), but they surprise people in two specific ways:

```r
f <- factor(c("10", "2", "33"))
as.numeric(f)
# [1] 1 2 3      -- the *factor codes* (levels sorted as TEXT: "10"<"2"<"33"), NOT the numbers you'd expect!

as.numeric(as.character(f))
# [1] 10  2 33   -- correct: go through character first
```

`as.numeric()` on a factor gives you the underlying integer *level code*,
not the number the label represents — a bug that silently produces
plausible-looking but wrong numbers. The fix is always to go through
`as.character()` first. Since R 4.0, `data.frame()` and `read.csv()` default
to `stringsAsFactors = FALSE`, so this trap is rarer than it used to be, but
you'll still hit it with any code (or dataset) written for older R, or when
a column is explicitly created with `factor()`.

## Standardizing text

```r
messy <- messy |>
    mutate(
        name = trimws(name),          # strip leading/trailing whitespace
        name = tolower(name),         # normalize case ...
        name = tools::toTitleCase(name)  # ... then Title Case it consistently
    )

messy$name
# [1] "Alice" "Bob"   "Carol" "Dave"  "Eve"
```

Without this step, `"bob"`, `"Bob"`, and `" Bob "` would be treated as three
different categories by `group_by()`, `table()`, or a join — a very common
source of "why does this have more groups than I expect" bugs.

## Detecting outliers

An impossible value like `score = 999` (on what's presumably a 0–100 scale)
won't be caught by `is.na()` — it's a valid number, just a wrong one. A
simple, robust check uses the interquartile range (IQR):

```r
q <- quantile(messy$score, probs = c(0.25, 0.75), na.rm = TRUE)
iqr <- q[2] - q[1]
lower <- q[1] - 1.5 * iqr
upper <- q[2] + 1.5 * iqr

messy |> filter(score < lower | score > upper)   # flag, don't just delete
#   name age score signup_date
# 1 Dave  22   999  2023-01-18
```

**Trap:** notice the row label printed is `1`, not `4` — unlike base R's
`messy[messy$score > upper, ]` (which keeps each row's *original* row name),
`dplyr::filter()` always renumbers rows sequentially in its result. This
rarely matters if you're working entirely in dplyr, but it means you can't
rely on a filtered data frame's row names to tell you a row's original
position — keep an explicit ID column (like this data's row order, or a
real primary key) if you need to trace a filtered row back to its source.

Flag outliers and look at them before deciding whether to cap, remove, or
keep them — deleting automatically can throw away a legitimately extreme
but real observation.

## Vectorization vs. loops for cleaning

Every operation above works on a whole column at once (`is.na(messy$score)`
tests all five values in one call). It's tempting, especially coming from
another language, to write:

```r
# Works, but un-idiomatic and much slower on real-sized data
for (i in seq_len(nrow(messy))) {
    if (is.na(messy$score[i])) {
        messy$score[i] <- 0
    }
}
```

R's vectorized functions (`ifelse()`, `coalesce()`, `is.na()`) apply to the
whole column in one call, run faster (they're implemented in C under the
hood), and read more clearly once you're used to the style. Reach for an
explicit loop only when a step genuinely depends on a *previous* row's
result (e.g. a running total that resets on a condition).

## Cleaning cheat sheet

| Problem | Function |
|---------|----------|
| Detect missing | `is.na(x)`, `anyNA(x)`, `colSums(is.na(df))` |
| Aggregate ignoring NA | `mean(x, na.rm = TRUE)` (also `sum`, `sd`, `median`, ...) |
| Fill missing with a default | `coalesce(x, default)` |
| String -> number | `as.numeric(x)` — watch for the coercion warning |
| Factor -> number (correctly) | `as.numeric(as.character(f))` |
| Trim whitespace | `trimws(x)` |
| Normalize case | `tolower(x)`, `toupper(x)`, `tools::toTitleCase(x)` |
| Flag outliers (IQR method) | `quantile(x, c(0.25, 0.75))` +/- `1.5 * IQR` |

## Exercise

Starting from `messy` as defined at the top: write a single dplyr pipe that
(1) trims and title-cases `name`, (2) converts `age` to numeric, capturing
which rows became `NA` in a new logical column `age_was_invalid`, (3) flags
`score` values outside the IQR bounds in a new column `score_is_outlier`
instead of deleting them, and (4) reports, with `sum()`, how many rows have
at least one problem across the three checks.
