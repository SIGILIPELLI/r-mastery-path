# 01 · Data Frames Deep Dive (dplyr)

Base R's `df[rows, columns]` syntax gets tedious fast once you're chaining
several operations together. **dplyr** is the tidyverse package for data
manipulation — it gives you a small set of verbs (`filter`, `select`,
`mutate`, `arrange`, `group_by`, `summarise`) that read almost like English
and compose cleanly with the pipe operator. Almost every real R data-analysis
script leans on dplyr, so fluency here pays off immediately.

```r
library(dplyr)

students <- data.frame(
    name = c("Alice", "Bob", "Carol", "Dave", "Eve"),
    grade = c("A", "B", "A", "C", "B"),
    score = c(92, 78, 88, 65, 81),
    year = c(1, 2, 1, 3, 2)
)
```

## The pipe: `|>`

R has a native pipe operator, `|>` (base R 4.1+; you'll also see the older
`%>%` from the `magrittr` package in tidyverse code — they behave almost
identically). `x |> f()` is exactly `f(x)`, and it chains left to right
instead of nesting parentheses:

```r
# Without a pipe -- reads inside-out
arrange(filter(students, score > 70), desc(score))

# With a pipe -- reads top to bottom, in the order things happen
students |>
    filter(score > 70) |>
    arrange(desc(score))
#    name grade score year
# 1 Alice     A    92    1
# 2 Carol     A    88    1
# 3   Eve     B    81    2
# 4   Bob     B    78    2
```

## `filter()` — keep rows matching a condition

```r
students |> filter(grade == "A")
#    name grade score year
# 1 Alice     A    92    1
# 2 Carol     A    88    1

students |> filter(score >= 80, year <= 2)   # commas = AND
students |> filter(grade == "A" | score > 85) # explicit OR
```

**Trap:** `filter()` silently drops rows where the condition evaluates to
`NA` rather than `TRUE` — it does *not* treat `NA` as a match, and it does
not error. If a column has missing values, `filter(col > 5)` quietly removes
those rows along with the ones that genuinely fail the test. Use
`filter(is.na(col) | col > 5)` if you want to keep the missing values too.

## `select()` — keep, drop, or reorder columns

```r
students |> select(name, score)          # by name
students |> select(-year)                # drop one column
students |> select(starts_with("s"))     # helper: score (and any other s* column)
students |> select(name, everything())   # reorder: name first, rest unchanged
```

## `mutate()` — add or change columns

```r
students <- students |>
    mutate(
        passed = score >= 70,
        score_pct = score / 100,
        grade_year = paste0(grade, "-", year)
    )

students
#    name grade score year passed score_pct grade_year
# 1 Alice     A    92    1   TRUE      0.92        A-1
# 2   Bob     B    78    2   TRUE      0.78        B-2
# 3 Carol     A    88    1   TRUE      0.88        A-1
# 4  Dave     C    65    3  FALSE      0.65        C-3
# 5   Eve     B    81    2   TRUE      0.81        B-2
```

Every new column inside one `mutate()` call can reference columns defined
earlier in the *same* call, evaluated left to right:

```r
students |> mutate(
    curved = score + 5,
    passed_curved = curved >= 70   # uses `curved`, defined one line above
)
```

## `arrange()` — sort rows

```r
students |> arrange(score)             # ascending
students |> arrange(desc(score))       # descending
students |> arrange(year, desc(score)) # multiple keys
```

## `group_by()` + `summarise()` — aggregate by group

This is the pair that replaces `tapply()` for anything beyond a single
group/single statistic:

```r
students |>
    group_by(grade) |>
    summarise(
        n = n(),
        avg_score = mean(score),
        max_score = max(score)
    )
# # A tibble: 3 x 4
#   grade     n avg_score max_score
#   <chr> <int>     <dbl>     <dbl>
# 1 A         2      90          92
# 2 B         2      79.5        81
# 3 C         1      65          65
```

`n()` (no arguments) counts rows in each group — it only works inside
`summarise()`/`mutate()` after a `group_by()`. Forgetting `group_by()`
before `summarise()` doesn't error; it just computes one summary over the
*entire* data frame, which is a common silent bug when refactoring code.

**Ungroup when you're done.** A `group_by()` sticks to the result until you
either pipe into `summarise()` (which drops one level of grouping) or call
`ungroup()` explicitly. Leaving a data frame grouped means later `mutate()`
calls on it run *per group* without you intending that — a frequent source
of "why is this number different from what I expected" bugs.

## Joins — combining two data frames

```r
grades_info <- data.frame(
    grade = c("A", "B", "C"),
    gpa_points = c(4.0, 3.0, 2.0)
)

students |> left_join(grades_info, by = "grade")
#    name grade score year gpa_points
# 1 Alice     A    92    1          4
# 2   Bob     B    78    2          3
# 3 Carol     A    88    1          4
# 4  Dave     C    65    3          2
# 5   Eve     B    81    2          3
```

| Join | Keeps |
|------|-------|
| `inner_join(a, b)` | only rows with a match in both |
| `left_join(a, b)` | all rows of `a`; unmatched `b` columns become `NA` |
| `right_join(a, b)` | all rows of `b`; unmatched `a` columns become `NA` |
| `full_join(a, b)` | all rows from either, `NA` where unmatched |
| `anti_join(a, b)` | rows in `a` with **no** match in `b` (useful for finding orphans) |

**Trap:** a join where the key column has duplicate values on either side
silently fans out into more rows than either input had (every match pairs
up), which can turn a 100-row `left_join` into a 400-row result if you're
not careful. Check `nrow()` before and after a join on unfamiliar data.

## Non-standard evaluation — why you write `filter(score > 70)`, not `filter(students$score > 70)` or `filter("score > 70")`

dplyr verbs use **tidy evaluation**: column names inside `filter()`,
`mutate()`, etc. are evaluated *inside the data frame*, not in your normal R
environment. That's why you write the bare name `score` rather than
`students$score` — dplyr already knows which data frame you're working
with because it's the first pipe argument. This is convenient for readable
code but has one common gotcha: if you want to filter using a column name
stored in a *variable*, `filter(students, col_name > 70)` won't work as
expected (it looks for a literal column called `col_name`). The fix is the
embrace operator from `rlang`/dplyr's tidy-eval machinery:

```r
my_filter <- function(data, column, threshold) {
    data |> filter({{ column }} > threshold)
}

my_filter(students, score, 80)
```

You won't need this often as a beginner, but recognizing the "object
'col_name' not found" error as an NSE issue — rather than a typo — will
save you real debugging time.

## dplyr verb cheat sheet

| Verb | Purpose |
|------|---------|
| `filter(cond)` | keep rows matching `cond` |
| `select(cols)` | keep/drop/reorder columns |
| `mutate(new = expr)` | add or change columns |
| `arrange(col)` | sort rows (`desc(col)` for descending) |
| `group_by(col)` | mark groups for the next summarise/mutate |
| `summarise(stat = expr)` | one row per group with aggregated values |
| `left_join(a, b, by = "key")` | combine two data frames on a key |
| `n()` | row count within a group (inside summarise/mutate) |

## Exercise

Using the `students` data frame above: find each grade's average score,
sorted from highest to lowest average, but only among students in `year`
1 or 2. Do it as a single pipe chain using `filter()`, `group_by()`,
`summarise()`, and `arrange()`. Then use `left_join()` to attach a `gpa_points`
column from `grades_info` to the *original* (non-summarised) `students`
table, and check `nrow()` matches before and after the join.
