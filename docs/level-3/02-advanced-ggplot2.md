# 02 · Advanced ggplot2

Level 2 covered the `ggplot2` grammar — `aes()`, geoms, and basic themes.
This module covers the tools you reach for once a single scatter plot
isn't enough: faceting to split one chart into a grid of small multiples,
custom themes you can reuse across a project, and the factor-ordering
gotcha that quietly reorders bars and legends on almost every real
dataset at some point.

```r
library(ggplot2)
library(dplyr)
```

## Faceting

```r
sales <- tibble(
  month = rep(1:6, times = 2),
  region = rep(c("East", "West"), each = 6),
  revenue = c(100, 120, 115, 140, 160, 155, 90, 95, 105, 110, 130, 128)
)

ggplot(sales, aes(x = month, y = revenue, color = region)) +
  geom_line() + geom_point() +
  facet_wrap(~ region) +
  labs(title = "Revenue by region", x = "Month", y = "Revenue ($k)") +
  theme_minimal()
```

`facet_wrap()` splits one plot into a panel per level of a categorical
variable, each sharing the same x/y scales by default — useful for
comparing shape and trend across groups without cluttering one panel with
overlapping lines. `facet_grid(rows ~ cols)` does the same thing along two
variables at once, arranging panels in an actual grid rather than a wrapped
sequence. Pass `scales = "free_y"` to `facet_wrap()` when the groups have
very different magnitudes and forcing a shared y-axis would flatten the
smaller ones into near-invisibility.

## Reusable custom themes

Repeating the same `theme()` tweaks on every plot in a project is a sign
you want a named theme object instead:

```r
my_theme <- theme_minimal(base_size = 12) +
  theme(
    plot.title = element_text(face = "bold"),
    legend.position = "bottom"
  )

ggplot(sales, aes(month, revenue, color = region)) +
  geom_line() +
  my_theme
```

Building on `theme_minimal()` (or any built-in theme) rather than
`theme()` alone means you inherit sensible defaults for everything you
didn't explicitly override — start from a base theme and only override
what your project's style guide actually specifies.

## Custom scales

```r
ggplot(sales, aes(month, revenue, color = region)) +
  geom_line() +
  scale_y_continuous(labels = scales::dollar_format(scale = 1, suffix = "k"))
```

`scale_*` layers control how a variable maps to a visual property — axis
tick labels and breaks (`scale_y_continuous`), color palettes
(`scale_color_manual()`, `scale_fill_brewer()`), or which values get
plotted at all. The `scales` package's `dollar_format()` and friends turn
raw axis numbers into formatted labels without touching the underlying
data — the values plotted are still plain numbers, only the *display* of
the axis text changes.

## The factor ordering trap

`ggplot2` draws categorical axes and legends in **factor level order**,
not the order values happen to appear in your data or any "natural"
reading order. If you never set factor levels explicitly, R falls back to
alphabetical order — which is rarely the order you want for anything with
an inherent ranking:

```r
grades <- c("Low", "High", "Medium")
levels(factor(grades))
# [1] "High"   "Low"    "Medium"     -- alphabetical, not a ranking

f <- factor(grades, levels = c("Low", "Medium", "High"))
levels(f)
# [1] "Low"    "Medium" "High"       -- the order you actually meant
as.integer(f)
# [1] 1 3 2
```

**Trap:** a bar chart or legend built from an un-leveled character column
of `"Low"`/`"Medium"`/`"High"` will silently render `High, Low, Medium` in
that alphabetical order — no warning, no error, just a chart that reads
oddly to anyone who expects a ranking. Always set `levels = c(...)`
explicitly on any categorical column with a meaningful order before
plotting it, whether via `factor()` beforehand or `scale_x_discrete(limits
= c(...))` at plot time.

## Bar charts with position control

```r
ggplot(sales, aes(month, revenue)) +
  geom_col(aes(fill = region), position = "dodge")
```

`position = "dodge"` places same-x bars for different groups side by side;
the default `"stack"` piles them on top of each other instead — pick
based on whether you want the reader comparing group totals or group
magnitudes at each x value.

## Cheat sheet

| Task | Function |
|---|---|
| Split into a panel per category | `facet_wrap(~ var)` |
| Split into a 2D grid of panels | `facet_grid(rows ~ cols)` |
| Let each facet panel scale independently | `facet_wrap(..., scales = "free_y")` |
| Build a reusable theme | `my_theme <- theme_minimal() + theme(...)` |
| Format axis labels without changing data | `scale_y_continuous(labels = scales::dollar_format())` |
| Fix categorical plotting order | `factor(x, levels = c(...))` |
| Side-by-side bars per group | `geom_col(position = "dodge")` |
| Stacked bars per group | `geom_col(position = "stack")` (default) |

## Exercise

1. Take the `sales` tibble above and facet it by `region` with
   `scales = "free_y"` — compare the result to the shared-scale default
   and note when each is the better choice.
2. Build a `my_theme` object with a title, subtitle, and caption styled
   consistently, and reuse it across two different plots of `sales`.
3. Create a factor column with levels `"Small"`, `"Medium"`, `"Large"`
   from data where those strings appear in a different order, plot it
   with `geom_bar()` unordered vs. explicitly ordered, and compare the
   two charts.
