# 04 · ggplot2 Basics

[Level 1's base plotting](../level-1/08-basic-plotting.md) (`plot()`,
`barplot()`, `hist()`) is quick and dependency-free, but each function has
its own quirky set of arguments, and combining multiple layers (points plus
a trend line plus grouping by color) gets awkward fast. **ggplot2**
implements a different idea — the "grammar of graphics" — where every plot
is built by *adding* layers to a base declaration of data and mappings.
Once the grammar clicks, it scales to far more complex plots with barely
more code than a simple one.

```r
library(ggplot2)

sales <- data.frame(
    quarter = rep(c("Q1", "Q2", "Q3", "Q4"), each = 2),
    region = rep(c("East", "West"), times = 4),
    revenue = c(120, 90, 150, 130, 170, 140, 200, 160)
)
```

## The grammar: `ggplot() + aes() + geom_*()`

Every ggplot2 plot follows the same skeleton:

```r
ggplot(data = sales, mapping = aes(x = quarter, y = revenue)) +
    geom_col()
```

- `ggplot(data, aes(...))` declares *which data* and *which columns map to
  which visual properties* (x position, y position, color, size, ...) —
  this alone draws nothing.
- `geom_col()` (or `geom_point()`, `geom_line()`, ...) is a **layer** that
  says *how* to draw it — bars, points, lines. `+` adds layers.

This separation is the entire idea: swap `geom_col()` for `geom_line()` and
you get a completely different chart from the exact same data declaration.

## Scatter plots — `geom_point()`

```r
students <- data.frame(
    hours_studied = c(1, 2, 3, 4, 5, 6, 7, 8),
    score = c(52, 58, 63, 70, 75, 82, 88, 91)
)

ggplot(students, aes(x = hours_studied, y = score)) +
    geom_point(color = "steelblue", size = 3) +
    labs(title = "Study Hours vs. Score", x = "Hours Studied", y = "Exam Score")
```

`labs()` sets the title and axis labels — the ggplot2 equivalent of base
R's `main`/`xlab`/`ylab` arguments, but added as its own layer rather than
passed to the plotting call itself.

## Bar charts — `geom_col()` vs. `geom_bar()`

This is a very common point of confusion:

```r
# geom_col(): the data ALREADY has one row per bar with the height you want
ggplot(sales, aes(x = quarter, y = revenue)) +
    geom_col(fill = "seagreen")

# geom_bar(): counts rows per category automatically -- no y needed
ggplot(sales, aes(x = quarter)) +
    geom_bar()   # height = number of rows per quarter, not revenue
```

`geom_col()` plots a value you already computed; `geom_bar()` computes a
*count* for you. Using `geom_bar()` when you meant `geom_col()` produces a
chart of row counts instead of your actual values — a genuinely common
mistake, since both draw bars and the error is silent (no error message,
just a wrong-looking chart).

## Mapping a column to color — grouping for free

```r
ggplot(sales, aes(x = quarter, y = revenue, fill = region)) +
    geom_col(position = "dodge") +
    labs(title = "Revenue by Quarter and Region", x = "Quarter", y = "Revenue")
```

Mapping `region` to `fill` inside `aes()` automatically colors and legends
each region — no manual loop over groups, no manual legend construction.
`position = "dodge"` places same-quarter bars side by side instead of
stacked (the default for `geom_col()` with a `fill` grouping).

## Line plots over time

```r
temps <- data.frame(
    day = 1:7,
    temp = c(15, 17, 16, 19, 22, 21, 18)
)

ggplot(temps, aes(x = day, y = temp)) +
    geom_line(color = "darkred", linewidth = 1) +
    geom_point(color = "darkred") +
    labs(title = "Temperature Over a Week", x = "Day", y = "°C")
```

Layers stack: this plot has both a line *and* points on top of it, from two
separate `geom_*()` calls sharing the same `aes()` mapping declared in
`ggplot()`.

## Histograms and boxplots

```r
set.seed(42)
exam_scores <- data.frame(score = rnorm(200, mean = 75, sd = 10))

ggplot(exam_scores, aes(x = score)) +
    geom_histogram(bins = 15, fill = "cornflowerblue", color = "white")

groups <- data.frame(
    group = rep(c("A", "B"), each = 5),
    score = c(88, 92, 79, 65, 85, 70, 95, 60, 55, 80)
)

ggplot(groups, aes(x = group, y = score, fill = group)) +
    geom_boxplot()
```

## Facets — one small plot per category, automatically

```r
ggplot(sales, aes(x = quarter, y = revenue)) +
    geom_col(fill = "steelblue") +
    facet_wrap(~ region)
```

`facet_wrap(~ column)` splits one plot into a grid of small multiples, one
panel per unique value of `column`, all with matching axes by default —
the ggplot2 way to compare groups side by side without a manual loop or a
cluttered single plot.

## Themes — controlling overall appearance

```r
ggplot(sales, aes(x = quarter, y = revenue, fill = region)) +
    geom_col(position = "dodge") +
    theme_minimal() +     # a clean built-in theme -- try theme_bw(), theme_classic() too
    labs(title = "Revenue by Quarter and Region")
```

## Saving a plot — `ggsave()`

```r
p <- ggplot(sales, aes(x = quarter, y = revenue)) +
    geom_col(fill = "seagreen") +
    labs(title = "Quarterly Revenue")

ggsave("revenue.png", plot = p, width = 8, height = 6, dpi = 150)
```

Unlike base R's `png()`/`dev.off()` pair, `ggsave()` takes a plot object
directly and infers the file format from the extension — no explicit device
open/close needed. Saving `p` as a variable first (instead of printing the
plot immediately) is also useful for reusing or modifying a plot before
deciding to save it.

## ggplot2 vs. base R cheat sheet

| Task | Base R | ggplot2 |
|------|--------|---------|
| Declare data + mapping | (implicit in each call) | `ggplot(data, aes(x=, y=))` |
| Scatter | `plot(x, y)` | `+ geom_point()` |
| Line | `plot(x, y, type="l")` | `+ geom_line()` |
| Bar (pre-computed values) | `barplot(x)` | `+ geom_col()` |
| Bar (row counts) | `table(x)` then `barplot()` | `+ geom_bar()` |
| Histogram | `hist(x)` | `+ geom_histogram()` |
| Boxplot | `boxplot(x, y)` | `+ geom_boxplot()` |
| Group by color | manual loop/legend | `aes(fill = group)` |
| Small multiples | manual `par(mfrow=)` + loop | `+ facet_wrap(~ group)` |
| Save to file | `png()` / `dev.off()` | `ggsave("file.png", plot)` |

## Exercise

Using the `sales` data frame from the top of this page: build a single
ggplot2 chart that shows revenue per quarter, grouped/colored by `region`,
faceted is *not* required here — instead use `position = "dodge"` so both
regions' bars sit side by side within each quarter. Add a title and axis
labels with `labs()`, apply `theme_minimal()`, and save the result to
`quarterly_revenue.png` with `ggsave()`. Then make a second version that
uses `facet_wrap(~ region)` instead of dodged bars, and write one sentence
comparing which version communicates the East/West comparison better.
