# 04 · R Markdown Reports

Every module so far has run R code and shown you the output separately.
R Markdown (`.Rmd`) files combine narrative text, R code, and rendered
output into a single document — the code chunks actually run when the
document is rendered, and the results (tables, printed values, plots) are
woven directly into the final HTML, PDF, or Word file. This is how you
turn a one-off analysis script into a shareable, reproducible report.

## Anatomy of an .Rmd file

```markdown
---
title: "Sales Summary"
output: html_document
params:
  min_qty: 2
---

## Overview

This report summarizes orders with quantity at least `r params$min_qty`.

​```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
library(dplyr)
​```

​```{r}
orders <- tibble(order_id = 1:5, qty = c(1, 3, 2, 5, 1))
orders %>% filter(qty >= params$min_qty)
​```
```

Three distinct pieces make up the file: the **YAML header** (between the
`---` fences) sets the title, output format, and any `params` the report
accepts; **narrative text** is plain Markdown, with inline R via
`` `r expression` `` for values that should appear in a sentence; **code
chunks** (fenced with ` ```{r} `) are R code that actually executes when
the document renders, with its printed output inserted into the document
right where the chunk sits.

## Rendering

```r
rmarkdown::render("report.Rmd", params = list(min_qty = 2))
```

```text
processing file: report.Rmd
1/6
2/6 [setup]
...
Output created: report.html
```

`render()` runs every code chunk top to bottom in a fresh R session,
knits the results into Markdown, then hands that off to Pandoc to produce
the final HTML (or PDF, or Word doc, depending on the `output:` field).
The `params` list lets one `.Rmd` template drive multiple reports with
different inputs — the report above with `min_qty = 2` filters
differently than the same file rendered with `min_qty = 4`, with no code
changes needed.

## Chunk options that matter

```r
​```{r, echo=FALSE}
cat("Total qualifying orders:", sum(orders$qty >= params$min_qty), "\n")
​```
```

| Option | Effect |
|---|---|
| `echo = FALSE` | Run the code, but don't show the code itself — only its output |
| `echo = TRUE` (default) | Show both the code and its output |
| `include = FALSE` | Run the code, show nothing (used for setup/library-loading chunks) |
| `eval = FALSE` | Show the code, don't run it (for illustrating syntax without executing) |
| `message = FALSE` / `warning = FALSE` | Suppress package-load messages or warnings from cluttering the report |
| `fig.width`, `fig.height` | Control the size of any plot the chunk produces |

A report meant for a non-technical audience typically sets `echo = FALSE`
globally in the `setup` chunk (`knitr::opts_chunk$set(echo = FALSE)`) so
the reader sees findings, not source code; a report meant to teach or
document the method itself does the opposite.

## Parameterized reports for repeat use

The real payoff of `params` shows up when the same report needs to run
for several inputs — one region, one month, one customer segment — without
copy-pasting the `.Rmd` file:

```r
regions <- c("East", "West", "South")
for (r in regions) {
  rmarkdown::render(
    "report.Rmd",
    params = list(region = r),
    output_file = paste0("report_", r, ".html")
  )
}
```

**Trap:** `render()` runs in a fresh environment by default, which means
any object created *outside* the `.Rmd` (in your interactive session) is
invisible inside it — a report that works when you manually step through
its chunks in the console can fail with "object not found" the moment you
call `render()`, because the console had leftover variables the fresh
render session does not. Every object the report needs must be created
*inside* the `.Rmd` itself, typically in the `setup` chunk.

## Tables that render properly

A bare `print()` of a data frame inside a chunk produces plain
fixed-width text, which looks fine in the R console but often wraps badly
in HTML output. `knitr::kable()` produces a properly formatted table in
whatever output format you're rendering to:

```r
​```{r}
knitr::kable(orders, caption = "Orders")
​```
```

## Cheat sheet

| Task | How |
|---|---|
| Render a report | `rmarkdown::render("file.Rmd")` |
| Pass different inputs to the same template | `params:` in YAML + `params = list(...)` at render time |
| Show output but hide the code | chunk option `echo = FALSE` |
| Run code with no output at all shown | chunk option `include = FALSE` |
| Show code without running it | chunk option `eval = FALSE` |
| Render a well-formatted table | `knitr::kable(df)` |
| Inline a computed value in a sentence | `` `r expression` `` |

## Exercise

1. Write an `.Rmd` file with a `params` block for a `min_qty` threshold,
   render it twice with two different values via `rmarkdown::render()`,
   and confirm the two output files show different filtered tables.
2. Add a chunk that produces a `ggplot2` plot, and use `fig.width`/
   `fig.height` chunk options to control its rendered size.
3. Convert a table in one of your reports from a bare `print()` to
   `knitr::kable()` and compare the two renders — note when `kable()`'s
   cleaner formatting actually matters (long reports, non-technical
   audiences) versus when it's overkill (a quick internal script).
