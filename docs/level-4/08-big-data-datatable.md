# 08 · Working with Big Data in R (data.table)

`data.frame` and `dplyr` are fine until a dataset gets into the
millions-of-rows range, where their overhead (copying, indexing by
name, non-vectorized grouping in some paths) starts to matter.
`data.table` keeps the same rectangular-data mental model but rewrites
the internals for speed and adds one very useful trick: modifying data
in place, without copying it.

```r
library(data.table)
```

## Creating one and the `[i, j, by]` syntax

```r
set.seed(1)
n <- 2000000
dt <- data.table(
  id  = sample(1:1000, n, replace = TRUE),
  grp = sample(c("a", "b", "c", "d"), n, replace = TRUE),
  val = rnorm(n)
)
nrow(dt)
```

```
[1] 2000000
```

Every `data.table` operation follows the same three-part shape:
`dt[i, j, by]` — filter rows with `i`, compute columns with `j`, group
with `by`. Aggregating by group:

```r
dt[, .(mean_val = mean(val), n = .N), by = .(grp)]
```

```
      grp      mean_val      n
   <char>         <num>  <int>
1:      b  0.0024201781 500307
2:      c -0.0015784674 499408
3:      d  0.0008492469 500248
4:      a -0.0013531090 500037
```

`.()` is shorthand for `list()` inside `data.table` expressions, and
`.N` is the built-in row count for the current group — no separate
`length()` or `n()` call needed.

## Why it's faster: a direct comparison

The same aggregation with base R's `aggregate()` on an equivalent
`data.frame`:

```r
df <- as.data.frame(dt)

system.time(dt[, .(mean_val = mean(val), n = .N), by = .(grp)])
system.time(aggregate(val ~ grp, data = df, FUN = mean))
```

```
data.table agg:
   user  system elapsed
  0.080   0.011   0.049
base R agg:
   user  system elapsed
  0.368   0.017   0.387
```

Roughly 8x faster on 2 million rows, and the gap widens as the row
count grows — `aggregate()`'s overhead comes from its generic formula
interface, while `data.table`'s `by` compiles down to a specialized
radix grouping pass.

## Filtering and keyed lookups

A row filter uses the `i` slot directly, with no `subset()` or `$`
needed:

```r
system.time(dt[val > 2 & grp == "a"])
```

```
   user  system elapsed
  0.017   0.000   0.017
```

For repeated lookups on the same column, `setkey()` sorts the table
once and builds a binary-search index on that column, turning later
lookups into something close to O(log n) instead of a full scan:

```r
setkey(dt, id)
system.time(dt[.(500)])
```

```
   user  system elapsed
  0.001   0.000   0.001
```

`dt[.(500)]` looks up all rows where `id == 500` using the key — this is
the syntax that benefits from `setkey()`; an unkeyed `dt[id == 500]`
still works but falls back to a full vector scan.

## Updating by reference with `:=`

The single biggest structural difference from `data.frame`/`dplyr`:
`:=` modifies a `data.table` in place, without making a copy of the
whole object:

```r
dt2 <- copy(dt)
system.time(dt2[, val_scaled := val * 2])
```

```
   user  system elapsed
  0.001   0.000   0.002
```

Two milliseconds to add a new derived column to two million rows,
because no copy of the other columns happens — `dt2` is mutated
directly. `copy()` above is deliberate: without it, `dt2 <- dt` would
make `dt2` a reference to the same underlying table, and `:=` on `dt2`
would silently also modify `dt`.

## R-specific traps

**`data.table` masks a few base and `dplyr`/`purrr`-adjacent function
names** — loading it can trigger "masked from 'package:base'" or
similar messages (as with `%notin%` above). These are warnings, not
errors, but check them when combining `data.table` with other packages
that define similarly-named helpers.

**`:=` mutates in place — assignment does not create an independent
copy.** `dt2 <- dt; dt2[, x := 1]` also changes `dt`, because both names
point to the same object. Use `copy(dt)` explicitly whenever you need an
independent table before mutating.

**`setkey()` sorts the table by the key column(s) as a side effect** —
if row order matters elsewhere in your pipeline (e.g. it corresponds to
original file order or a time sequence), sort a copy or capture the
original order in its own column before keying.

## Cheat sheet

| Task | Syntax |
|---|---|
| Create a data.table | `data.table(col = values, ...)` |
| Filter rows | `dt[condition]` |
| Compute/aggregate columns | `dt[, .(new = expr), by = .(group_col)]` |
| Row count in a group | `.N` |
| Add/modify a column in place | `dt[, newcol := expr]` |
| Independent copy before mutating | `copy(dt)` |
| Build a lookup index | `setkey(dt, col)` |
| Keyed lookup | `dt[.(value)]` |
| Convert from data.frame | `as.data.table(df)` |
| Convert to data.frame | `as.data.frame(dt)` |

## Exercise

1. Build a 5-million-row `data.table` with an `id`, a `category`
   (5 levels), and a numeric value, then compute per-category mean,
   median, and count in a single `dt[, .(...), by = category]` call.
2. Compare timing for a filtered lookup (`dt[id == 12345]`) before and
   after `setkey(dt, id)`, and explain the difference in your own words.
3. Add a new column that's a rolling transformation of an existing one
   (e.g. `val - mean(val)` within each group) using `:=` combined with
   `by =`, and confirm it modified the table in place rather than
   returning a new one.
