# 07 · Data Visualization at Scale

`ggplot2` scales visually to any dataset — the code to plot 200,000
points is identical to the code for 200. What doesn't scale is the
plot's *usefulness*: a scatterplot of 200,000 points is mostly
overplotting, and rendering every one of them is slower than it needs
to be for a chart nobody can actually read.

```r
library(ggplot2)
```

## The problem: plotting 200,000 points directly

```r
set.seed(1)
n <- 200000
df <- data.frame(x = rnorm(n), y = rnorm(n) + 0.5 * rnorm(n))

system.time({
  p1 <- ggplot(df, aes(x, y)) + geom_point(alpha = 0.02)
  ggsave("scatter_full.png", p1, width = 5, height = 4, dpi = 100)
})
```

```
   user  system elapsed
  1.295   0.013   1.317
```

Over a second just to render and save one PNG, and the result is a
dense grey blob — `alpha = 0.02` helps show density, but at this scale
you're fighting the renderer, not communicating anything a coarser
summary wouldn't show faster.

## Fix 1: bin the data instead of plotting every point

`geom_hex()` (from the `hexbin` package) aggregates points into hexagonal
bins and colors by count — turning an overplotted scatter into an actual
density map:

```r
system.time({
  p2 <- ggplot(df, aes(x, y)) + geom_hex(bins = 60)
  ggsave("hexbin.png", p2, width = 5, height = 4, dpi = 100)
})
```

```
   user  system elapsed
  0.184   0.006   0.193
```

About 7x faster than the raw scatter, and the resulting plot is more
informative, not less — density is now an explicit color scale instead
of something you had to squint at through overlapping points. `geom_bin2d()`
does the same thing with square bins if hexagons aren't wanted.

## Fix 2: sample before plotting

When the exact shape of individual points still matters (rather than
aggregate density), a random sample is usually visually indistinguishable
from the full dataset and is dramatically cheaper:

```r
samp <- df[sample(nrow(df), 5000), ]

system.time({
  p3 <- ggplot(samp, aes(x, y)) + geom_point(alpha = 0.3)
  ggsave("scatter_sample.png", p3, width = 5, height = 4, dpi = 100)
})
```

```
   user  system elapsed
  0.074   0.002   0.075
```

Roughly 18x faster than plotting all 200,000 points, from a dataset
that's 2.5% the size — for exploratory work this is usually the right
first move, since it lets you iterate on the plot's aesthetics quickly
before committing to a full render for a final figure.

## Building a small system: a reusable theme

At scale you're rarely making one plot — you're making many that need
to look consistent. Define the shared styling once as a `theme()` object
and reuse it, rather than repeating `theme()` calls across every plot:

```r
theme_report <- function(base_size = 12) {
  theme_minimal(base_size = base_size) +
    theme(
      panel.grid.minor = element_blank(),
      plot.title = element_text(face = "bold"),
      legend.position = "bottom"
    )
}

ggplot(df, aes(x, y)) +
  geom_hex(bins = 60) +
  labs(title = "Joint distribution of x and y") +
  theme_report()
```

Every plot built with `theme_report()` shares margins, grid style, and
title weight, so a dashboard of a dozen figures reads as one system
instead of a dozen different defaults.

## Faceting instead of a loop of separate plots

Producing one plot per group with a manual loop means manually managing
file names and axis consistency across each; `facet_wrap()` produces
all the panels in one call with shared, comparable scales:

```r
df$group <- sample(c("A", "B", "C", "D"), nrow(df), replace = TRUE)

ggplot(df, aes(x, y)) +
  geom_hex(bins = 40) +
  facet_wrap(~group) +
  theme_report()
```

Faceting is almost always preferable to a hand-rolled loop over
`unique(df$group)` — it guarantees consistent axis ranges across panels,
which a loop of independently-scaled plots does not.

## R-specific traps

**`ggsave()`'s default size comes from the *current graphics device*, not
a fixed default** — omitting `width`/`height` can silently produce a
different image size depending on what plotting device happened to be
open, which is why the examples above pass them explicitly.

**`geom_hex()` requires the separate `hexbin` package** even though it
ships as part of `ggplot2`'s API — without it, `geom_hex()` fails at
render time with `"The package \"hexbin\" is required"`, not at
`library(ggplot2)` time, so the failure shows up later than you'd
expect.

**Sampling for display can distort a skewed distribution's tails** — a
5,000-point random sample from 200,000 will faithfully represent the
bulk of the distribution but may drop or under-represent rare extreme
values that matter for a report. Bin the full dataset (Fix 1) rather
than sample it (Fix 2) whenever tail behavior is the point of the plot.

## Cheat sheet

| Task | Call |
|---|---|
| Bin points into hexagons by density | `geom_hex(bins = n)` |
| Bin points into squares by density | `geom_bin2d()` |
| Random sample of rows for quick iteration | `df[sample(nrow(df), k), ]` |
| Save a plot with explicit dimensions | `ggsave("file.png", p, width = w, height = h, dpi = d)` |
| Reusable custom theme | `theme_x <- function() theme_minimal() + theme(...)` |
| One plot per group, shared scales | `facet_wrap(~group)` |
| Time how long a plot takes to render/save | `system.time({ ... })` |

## How It Actually Works

Rendering thousands to millions of points with `ggplot2` slows down
because every geometric primitive still goes through the same
grid-graphics grob pipeline from Module 4 — each point becomes an
individual drawable object the graphics device has to process, and vector
output formats (PDF/SVG) end up storing one drawing instruction per point,
producing enormous files. Techniques like `geom_hex()`/`geom_bin2d()`
sidestep this by first **aggregating** raw points into a coarser grid of
bins (a single pass computing counts per bin, same mechanism as
`dplyr::group_by()`'s hashed grouping) and drawing only that much smaller
set of summary shapes — the visual cost becomes proportional to the number
of bins, not the number of raw rows.

Interactive/scalable alternatives (`plotly`, rendering to a raster PNG
device instead of vector PDF for dense scatter plots) work by changing
*where* the pixel-density cost is paid: a raster device rasterizes each
point to actual pixels once at a fixed resolution (further points just
overdraw the same pixel, so file size stays bounded regardless of point
count), while `plotly` converts the ggplot object into a JSON
specification that a JavaScript/WebGL renderer draws in the browser using
GPU-accelerated primitives rather than R's CPU-bound graphics device.

## Exercise

1. Generate a data frame of 500,000 points from two overlapping normal
   distributions (a mixture), then produce both a raw `geom_point()`
   scatter and a `geom_hex()` version, timing each with `system.time()`.
2. Write a `theme_report()` function like the one above and apply it to
   three different plot types (scatter/hex, bar, boxplot) to confirm the
   styling is visually consistent across all three.
3. Add a categorical grouping column to your mixture dataset and use
   `facet_wrap()` to produce one density panel per group in a single
   `ggplot()` call.
