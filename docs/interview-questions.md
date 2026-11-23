---
title: "R Interview Questions and Answers (182 Questions)"
description: "182 R interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the R Mastery Path."
---

# R Interview Questions and Answers

**182 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free R Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Entry

### [01 · Setup &amp; First Script](level-1/01-setup.md)

**Q1.** What is R mainly used for?

**Answer:** Statistical computing, data analysis and visualization

Data-focused language.

**Q2.** What does CRAN host?

**Answer:** The Comprehensive R Archive Network of packages and R itself

install.packages().

**Q3.** Which tool is the popular R IDE?

**Answer:** RStudio (Posit)

Also VS Code.

**Q4.** How run a script from the terminal?

**Answer:** Rscript script.R

Non-interactive.

### [02 · Variables &amp; Types](level-1/02-variables-types.md)

**Q5.** What does this print?

```r
x <- c(1L, 2L); y <- 2.5; z <- "7"
cat(class(x), class(y), class(z), class(TRUE), class(NULL), "\n")
cat(as.integer(z) + 1L, is.na(as.numeric("abc")), 5 %/% 2, 5 %% 2, "\n")
```

**Answer:** integer numeric character logical NULL 
8 TRUE 2 1

class() of a double is numeric; as.numeric('abc') gives NA with a warning. Verified by compiling and running it.

**Q6.** What does this print?

```r
x <- 5
f <- function() { x <- 10; x }
cat(f(), x, "\n")
cat(identical(1L, 1), 1L == 1, "\n")
```

**Answer:** 10 5 
FALSE TRUE

Local vs global; identical checks type too. Verified by compiling and running it.

**Q7.** What is the assignment operator preferred in R style?

**Answer:** &lt;-

= also works at top level.

**Q8.** What does NA mean?

**Answer:** A missing value (not available), propagating through most calculations

Use na.rm = TRUE.

### [03 · Control Flow](level-1/03-control-flow.md)

**Q9.** What does this print?

```r
x <- 7
r <- if (x > 5) "big" else "small"
s <- 0
for (i in 1:10) { if (i %% 2 == 0) next; if (i > 7) break; s <- s + i }
i <- 0; while (TRUE) { i <- i + 1; if (i >= 3) break }
cat(r, s, i, "\n")
cat(ifelse(c(1, 5, 10) > 4, "hi", "lo"), "\n")
```

**Answer:** big 16 3 
lo hi hi

next, break and vectorized ifelse. Verified by compiling and running it.

**Q10.** What does this print?

```r
sw <- function(x) switch(x, a = "A", b = , c = "BC", "other")
cat(sw("a"), sw("b"), sw("z"), "\n")
cat(length(seq_len(0)), "\n")
```

**Answer:** A BC other 
0

switch falls through empty arms; seq_len(0) is empty. Verified by compiling and running it.

**Q11.** Why prefer seq_along(x) to 1:length(x)?

**Answer:** 1:length(x) misbehaves on empty vectors (gives 1,0)

Safe loops.

**Q12.** What does the vectorized if-else function do?

**Answer:** ifelse() evaluates a condition elementwise over a vector

Avoid loops.

### [04 · Functions](level-1/04-functions.md)

**Q13.** What does this print?

```r
f <- function(x, y = 2, ...) { extras <- list(...); x * y + length(extras) }
cat(f(3), f(3, 4), f(3, y = 1, 99, 100), "\n")
compose <- function(f, g) function(x) g(f(x))
h <- compose(sqrt, function(v) v + 1)
cat(h(16), "\n")
```

**Answer:** 6 12 5 
5

Defaults, dots and closures. Verified by compiling and running it.

**Q14.** What does this print?

```r
make_counter <- function() { n <- 0; function() { n <<- n + 1; n } }
c1 <- make_counter(); c2 <- make_counter()
invisible(c1()); invisible(c1())
cat(c1(), c2(), "\n")
cat(sapply(1:3, function(i) i^2), "\n")
```

**Answer:** 3 1 
1 4 9

&lt;&lt;- modifies the enclosing environment. Verified by compiling and running it.

**Q15.** What does a function return by default?

**Answer:** The value of its last evaluated expression

return() is optional.

**Q16.** What does `...` pass along?

**Answer:** Extra arguments to other functions

Wrapper functions.

### [05 · Vectors &amp; Basic Data Structures](level-1/05-vectors-data-structures.md)

**Q17.** What does this print?

```r
v <- c(a = 1, b = 2, c = 3)
cat(v["b"], v[-1], v[v > 1], "\n")
print(rev(v))
l <- list(x = 1, y = "a", z = 1:3)
cat(length(l), l$z[2], class(l[["y"]]), class(l["y"]), "\n")
m <- matrix(1:6, nrow = 2)
cat(dim(m), m[2, 3], "\n")
```

**Answer:** 2 2 3 2 3 
c b a 
3 2 1 
3 2 character list 
2 3 6

[[ extracts the element; [ keeps a list. Verified by compiling and running it.

**Q18.** What does this print?

```r
x <- 1:10
cat(sum(x[x %% 2 == 0]), mean(x), median(c(5, 1, 9, 3)), "\n")
cat(1:3 + c(10, 20, 30), 1:6 + 1:2, "\n")
cat(which(c(F, T, T)), any(x > 9), all(x > 0), "\n")
```

**Answer:** 30 5.5 4 
11 22 33 2 4 4 6 6 8 
2 3 TRUE TRUE

Vectorized arithmetic with recycling. Verified by compiling and running it.

**Q19.** What is recycling?

**Answer:** Shorter vectors are repeated to match the longer one's length

Warns if not a multiple.

**Q20.** Difference between list and vector?

**Answer:** A vector holds one type; a list can hold mixed types and structures

Atomic vs generic.

### [06 · Data Frames Basics](level-1/06-data-frames-basics.md)

**Q21.** What does this print?

```r
df <- data.frame(g = c("a","a","b","b","c"), v = c(1, 2, 3, NA, 5), stringsAsFactors = FALSE)
cat(nrow(df), ncol(df), names(df), "\n")
cat(mean(df$v), mean(df$v, na.rm = TRUE), sum(is.na(df$v)), "\n")
print(df[df$g == "a", ])
print(subset(df, v > 2, select = v))
```

**Answer:** 5 2 g v 
NA 2.75 1 
  g v
1 a 1
2 a 2
  v
3 3
5 5

Subsetting drops NA rows in subset() but df[ , ] would keep NA rows. Verified by compiling and running it.

**Q22.** What does this print?

```r
df <- data.frame(x = 1:3, y = c("a", "b", "c"))
df$z <- df$x * 2
df <- rbind(df, data.frame(x = 4L, y = "d", z = 8))
print(df[order(-df$x), ][1:2, ])
str(df)
```

**Answer:** x y z
4 4 d 8
3 3 c 6
'data.frame':	4 obs. of  3 variables:
 $ x: int  1 2 3 4
 $ y: chr  "a" "b" "c" "d"
 $ z: num  2 4 6 8

Adding columns, binding rows and sorting. Verified by compiling and running it.

**Q23.** What does str(df) show?

**Answer:** Structure: dimensions, column types and sample values

Quick inspection.

**Q24.** Difference between head(df) and summary(df)?

**Answer:** head shows the first rows; summary gives per-column statistics

Explore data.

### [07 · Reading Data](level-1/07-reading-data.md)

**Q25.** What does this print?

```r
tmp <- tempfile(fileext = ".csv")
writeLines(c("id,name,score", "1,Ann,3.5", "2,Bob,NA", "3,Cy,4"), tmp)
d <- read.csv(tmp)
str(d)
cat(sum(is.na(d$score)), mean(d$score, na.rm = TRUE), "\n")
d2 <- read.csv(tmp, colClasses = c("character", "character", "numeric"))
cat(class(d2$id), "\n")
```

**Answer:** 'data.frame':	3 obs. of  3 variables:
 $ id   : int  1 2 3
 $ name : chr  "Ann" "Bob" "Cy"
 $ score: num  3.5 NA 4
1 3.75 
character

read.csv infers types; colClasses overrides. Verified by compiling and running it.

**Q26.** What does readr::read_csv do differently from read.csv?

**Answer:** It is faster, never converts strings to factors and prints a column spec

tidyverse.

**Q27.** How read an Excel file?

**Answer:** readxl::read_excel()

Package needed.

**Q28.** Why set stringsAsFactors explicitly in older R?

**Answer:** Before R 4.0 strings were converted to factors by default

Changed in R 4.0.

### [08 · Basic Plotting](level-1/08-basic-plotting.md)

**Q29.** What does this print?

```r
x <- c(2, 4, 4, 4, 5, 5, 7, 9)
h <- hist(x, breaks = c(0, 3, 6, 9), plot = FALSE)
cat(h$counts, "\n")
b <- boxplot.stats(c(1, 2, 3, 4, 100))
cat(b$stats, b$out, "\n")
cat(round(quantile(x, c(.25, .75)), 2), "\n")
```

**Answer:** 1 5 2 
1 2 3 4 4 100 
4 5.5

Histogram bin counts, outliers and quantiles. Verified by compiling and running it.

**Q30.** Which base function draws a scatter plot?

**Answer:** plot(x, y)

Base graphics.

**Q31.** When is a histogram appropriate?

**Answer:** To show the distribution of one numeric variable

Binning.

**Q32.** Why label axes and titles?

**Answer:** So a plot is understandable without extra explanation

Communication.

### [09 · Packages](level-1/09-packages.md)

**Q33.** How install and load a package?

**Answer:** install.packages('x') once, then library(x) in each session

Two steps.

**Q34.** What does renv do?

**Answer:** Records and restores project-specific package versions

Reproducibility.

**Q35.** What does dplyr::filter mask?

**Answer:** stats::filter — use dplyr::filter() if conflicts matter

Namespace.

**Q36.** What does `pkg::fun()` do?

**Answer:** Calls a function without attaching the package

Explicit namespaces.

### [10 · Project — Data Analysis Script](level-1/10-project-data-analysis.md)

**Q37.** What does the data analysis project combine?

**Answer:** Reading data, cleaning, summarizing and plotting with a short report

Level 1 project.

**Q38.** How make the analysis reproducible?

**Answer:** A script (or R Markdown) that runs top to bottom from raw data

No manual steps.

**Q39.** How validate cleaned data?

**Answer:** Check counts, ranges and missingness before and after

Sanity checks.

**Q40.** What belongs in the conclusion?

**Answer:** Findings with caveats and the data limitations

Honesty.

## Level 2 · Intermediate

### [01 · Data Frames Deep Dive (dplyr)](level-2/01-dplyr-deep-dive.md)

**Q41.** What does this print?

```r
suppressPackageStartupMessages(library(dplyr))
df <- data.frame(g = c("a","a","b","b","c"), v = c(1, 2, 3, 4, 5))
r <- df %>% filter(v > 1) %>% group_by(g) %>% summarise(n = n(), total = sum(v), .groups = "drop") %>% arrange(desc(total))
print(as.data.frame(r))
print(df %>% mutate(share = round(v / sum(v), 2)) %>% slice_max(v, n = 2) %>% as.data.frame())
```

**Answer:** g n total
1 b 2     7
2 c 1     5
3 a 1     2
  g v share
1 c 5  0.33
2 b 4  0.27

filter, group_by, summarise, arrange, mutate, slice_max. Verified by compiling and running it.

**Q42.** What does this print?

```r
suppressPackageStartupMessages(library(dplyr))
a <- data.frame(id = 1:3, x = c("p", "q", "r"))
b <- data.frame(id = c(2, 3, 4), y = c(20, 30, 40))
print(left_join(a, b, by = "id"))
cat(nrow(inner_join(a, b, by = "id")), nrow(full_join(a, b, by = "id")), nrow(anti_join(a, b, by = "id")), "\n")
```

**Answer:** id x  y
1  1 p NA
2  2 q 20
3  3 r 30
2 4 1

Join types. Verified by compiling and running it.

**Q43.** What does the pipe %&gt;% (or |&gt;) do?

**Answer:** Passes the left result as the first argument of the next function

Readable pipelines.

**Q44.** What does group_by() change?

**Answer:** Subsequent summaries run per group

Grouped data.

### [02 · Data Cleaning](level-2/02-data-cleaning.md)

**Q45.** What does this print?

```r
x <- c(" Ann ", "bob", NA, "Cy", "")
clean <- trimws(x)
clean[clean == ""] <- NA
cat(sum(is.na(clean)), toupper(substr(clean, 1, 1))[1:2], "\n")
d <- c(10, 12, 11, 400, 9)
q1 <- quantile(d, .25); q3 <- quantile(d, .75); iqr <- q3 - q1
cat(d[d < q1 - 1.5 * iqr | d > q3 + 1.5 * iqr], "\n")
cat(complete.cases(data.frame(a = c(1, NA), b = c(2, 3))), "\n")
```

**Answer:** 2 A B 
400 
TRUE FALSE

Trim, recode blanks, IQR outliers. Verified by compiling and running it.

**Q46.** What does this print?

```r
f <- factor(c("lo", "hi", "mid", "hi"), levels = c("lo", "mid", "hi"))
cat(as.integer(f), levels(f), "\n")
print(table(f))
cat(duplicated(c(1, 2, 1, 3, 2)), "\n")
```

**Answer:** 1 3 2 3 lo mid hi 
f
 lo mid  hi 
  1   1   2 
FALSE FALSE TRUE FALSE TRUE

Factor levels define order. Verified by compiling and running it.

**Q47.** What is tidy data?

**Answer:** Each variable is a column, each observation a row, each value a cell

Wickham.

**Q48.** Why not silently drop NAs?

**Answer:** It can bias results; understand why values are missing first

Missingness.

### [03 · Apply Family Functions](level-2/03-apply-family-functions.md)

**Q49.** What does this print?

```r
x <- list(a = 1:3, b = 4:6)
print(sapply(x, sum))
print(lapply(1:2, function(i) i * 10))
print(vapply(c("a", "bb", "ccc"), nchar, integer(1)))
print(mapply(function(a, b) a + b, 1:3, 4:6))
print(Map(`+`, 1:2, 3:4))
```

**Answer:** a  b 
 6 15 
[[1]]
[1] 10

[[2]]
[1] 20

  a  bb ccc 
  1   2   3 
[1] 5 7 9
[[1]]
[1] 4

[[2]]
[1] 6

sapply simplifies; vapply enforces the result type. Verified by compiling and running it.

**Q50.** Why prefer vapply over sapply in packages?

**Answer:** It guarantees the return type and fails loudly otherwise

Type safety.

**Q51.** What does apply(m, 1, f) do?

**Answer:** Applies f to each row of a matrix

MARGIN 1 = rows.

**Q52.** What does Reduce do?

**Answer:** Combines elements of a vector or list pairwise into one value

Fold.

### [04 · ggplot2 Basics](level-2/04-ggplot2-basics.md)

**Q53.** What does this print?

```r
suppressPackageStartupMessages(library(ggplot2))
p <- ggplot(mtcars, aes(wt, mpg, colour = factor(cyl))) + geom_point() + geom_smooth(method = "lm", se = FALSE) + labs(title = "MPG vs weight", x = "Weight", y = "MPG")
cat(length(p$layers), sapply(p$layers, function(l) class(l$geom)[1]), "\n")
```

**Answer:** 2 GeomPoint GeomSmooth

A ggplot is built from layers. Verified by compiling and running it.

**Q54.** What are the core parts of a ggplot?

**Answer:** Data, aesthetic mappings and geoms (layers)

Grammar of graphics.

**Q55.** What does aes() do?

**Answer:** Maps data columns to visual properties

Mapping.

**Q56.** What does facet_wrap do?

**Answer:** Splits a plot into small multiples by a variable

Comparisons.

### [05 · Working with Strings (stringr)](level-2/05-strings-stringr.md)

**Q57.** What does this print?

```r
suppressPackageStartupMessages(library(stringr))
s <- c("order-123 shipped", "order-45 pending", "no id")
cat(str_extract(s, "\\d+"), "\n")
cat(str_detect(s, "ship"), str_length(s), "\n")
cat(str_replace_all("a-b-c", "-", "+"), str_to_title("hello world"), str_pad("7", 3, pad = "0"), "\n")
cat(unlist(str_split("a,b,c", ",")), str_sub("abcdef", 2, 4), "\n")
```

**Answer:** 123 45 NA 
TRUE FALSE FALSE 17 16 5 
a+b+c Hello World 007 
a b c bcd

stringr verbs. Verified by compiling and running it.

**Q58.** What do regex special characters need in R strings?

**Answer:** Double backslashes, e.g. "\\d+"

Escape twice.

**Q59.** What is the advantage of stringr over base?

**Answer:** Consistent function names and argument order

Readable.

**Q60.** What does str_detect return?

**Answer:** A logical vector (one per string)

Filtering.

### [06 · Statistical Basics](level-2/06-statistical-basics.md)

**Q61.** What does this print?

```r
set.seed(1)
x <- c(2.1, 2.5, 3.0, 3.2, 3.8); y <- c(4.2, 5.1, 6.2, 6.4, 7.7)
m <- lm(y ~ x)
cat(round(coef(m), 3), round(summary(m)$r.squared, 3), "\n")
cat(round(cor(x, y), 3), round(sd(x), 3), round(var(x), 3), "\n")
t <- t.test(c(5.1, 4.9, 5.6, 5.8, 6.0), mu = 5)
cat(round(t$statistic, 3), round(t$p.value, 3), "\n")
```

**Answer:** -0.026 2.036 0.996 
0.998 0.653 0.427 
2.304 0.083

lm coefficients are intercept then slope. Verified by compiling and running it.

**Q62.** What does a p-value of 0.03 mean?

**Answer:** If the null were true, results this extreme would occur about 3% of the time

Not the probability the null is true.

**Q63.** What does R-squared measure?

**Answer:** The share of variance in y explained by the model

Fit, not causality.

**Q64.** Correlation vs causation?

**Answer:** Correlation doesn't imply that one variable causes the other

Confounders.

### [07 · Writing Functions &amp; Package Structure](level-2/07-functions-package-structure.md)

**Q65.** What does this print?

```r
clamp <- function(x, lo = 0, hi = 1) {
  stopifnot(is.numeric(x), lo <= hi)
  pmin(pmax(x, lo), hi)
}
cat(clamp(c(-1, 0.5, 2)), "\n")
r <- tryCatch(clamp("a"), error = function(e) conditionMessage(e))
cat(r, "\n")
#' Add two numbers
#' @param a first
#' @export
add <- function(a, b) a + b
cat(add(1, 2), "\n")
```

**Answer:** 0 0.5 1 
is.numeric(x) is not TRUE 
3

stopifnot gives a default message. Verified by compiling and running it.

**Q66.** What is the standard layout of an R package?

**Answer:** DESCRIPTION, NAMESPACE, R/ for code, man/ for docs, tests/

Conventions.

**Q67.** What generates docs from comments?

**Answer:** roxygen2

#' comments.

**Q68.** What does devtools::load_all() do?

**Answer:** Loads the package code for interactive development without installing

Fast iteration.

### [08 · Testing with testthat](level-2/08-testing-testthat.md)

**Q69.** What does this print?

```r
suppressPackageStartupMessages(library(testthat))
sq <- function(x) x^2
expect_equal(sq(3), 9)
expect_equal(sq(-2), 4)
cat(TRUE, "\n")
r <- tryCatch(expect_equal(sq(3), 10), error = function(e) "failed")
cat(r, "\n")
expect_error(stop("boom"), "boom")
expect_true(all(sq(1:3) == c(1, 4, 9)))
cat("done\n")
```

**Answer:** TRUE 
failed 
done

expect_* throw an error on failure (test_that groups them and reports instead). Verified by compiling and running it.

**Q70.** Where do testthat tests live?

**Answer:** tests/testthat/test-*.R

Conventions.

**Q71.** Which expectation compares numbers with tolerance?

**Answer:** expect_equal

Floating point.

**Q72.** What does expect_error check?

**Answer:** That code throws an error (optionally matching a message)

Negative tests.

### [09 · Working with Dates (lubridate)](level-2/09-dates-lubridate.md)

**Q73.** What does this print?

```r
suppressPackageStartupMessages(library(lubridate))
d <- ymd("2026-01-31")
cat(format(d + months(1), "%Y-%m-%d"), format(d %m+% months(1), "%Y-%m-%d"), "\n")
cat(as.character(wday(d, label = TRUE)), month(d), as.integer(ymd("2026-03-01") - d), "\n")
cat(format(floor_date(ymd_hms("2026-05-17 13:45:10"), "hour"), "%H:%M"), "\n")
```

**Answer:** NA 2026-02-28 
Sat 1 29 
13:00

Adding months to Jan 31 gives NA; %m+% rolls back to month end. Verified by compiling and running it.

**Q74.** What does ymd() do?

**Answer:** Parses year-month-day strings into Date objects

Parsers by order.

**Q75.** Why store timestamps in UTC?

**Answer:** Avoids time zone and DST confusion

Convert for display.

**Q76.** What does difftime compute?

**Answer:** The difference between two dates/times with units

units argument.

### [10 · Project — Data Cleaning &amp; Reporting Pipeline](level-2/10-project-cleaning-pipeline.md)

**Q77.** What does the cleaning pipeline project combine?

**Answer:** Reading messy data, standardizing, validating and documenting a reproducible pipeline

Level 2 project.

**Q78.** How document cleaning decisions?

**Answer:** In code comments and a log of rows changed/dropped

Auditability.

**Q79.** What do you check after each step?

**Answer:** Row counts, missingness and value ranges

Validation.

**Q80.** Why keep raw data untouched?

**Answer:** So the pipeline can be re-run and audited from source

Immutability.

## Level 3 · Advanced

### [01 · Advanced dplyr/tidyr](level-3/01-advanced-dplyr-tidyr.md)

**Q81.** What does this print?

```r
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(tidyr))
w <- data.frame(id = 1:2, q1 = c(10, 20), q2 = c(15, 25))
l <- pivot_longer(w, c(q1, q2), names_to = "quarter", values_to = "sales")
print(l)
print(pivot_wider(l, names_from = quarter, values_from = sales))
d <- data.frame(g = c("a","a","b"), v = c(1, 5, 3))
print(d %>% group_by(g) %>% mutate(rank = row_number(desc(v)), prev = lag(v)) %>% as.data.frame())
```

**Answer:** # A tibble: 4 × 3
     id quarter sales
  &lt;int&gt; &lt;chr&gt;   &lt;dbl&gt;
1     1 q1         10
2     1 q2         15
3     2 q1         20
4     2 q2         25
# A tibble: 2 × 3
     id    q1    q2
  &lt;int&gt; &lt;dbl&gt; &lt;dbl&gt;
1     1    10    15
2     2    20    25
  g v rank prev
1 a 1    2   NA
2 a 5    1    1
3 b 3    1   NA

pivot_longer/pivot_wider and grouped window functions. Verified by compiling and running it.

**Q82.** Why use pivot_longer?

**Answer:** To turn wide columns into tidy key/value rows for plotting and grouping

Tidy data.

**Q83.** What does across() do?

**Answer:** Applies functions to multiple columns inside mutate/summarise

Less repetition.

**Q84.** What is a window function?

**Answer:** One that computes per row using related rows (lag, rank, cumsum)

Within groups.

### [02 · Advanced ggplot2](level-3/02-advanced-ggplot2.md)

**Q85.** What does scale_colour_viridis_d give?

**Answer:** A colour-blind-friendly discrete palette

Accessibility.

**Q86.** How highlight a subset in ggplot2?

**Answer:** Layer a second geom with a filtered data argument

Layered grammar.

**Q87.** What does theme() control?

**Answer:** Non-data styling: fonts, grid lines, legend position

Appearance.

**Q88.** What does ggsave do?

**Answer:** Saves a plot to a file at a chosen size and resolution

Reproducible output.

### [03 · Statistical Modeling](level-3/03-statistical-modeling.md)

**Q89.** What does this print?

```r
fit <- lm(mpg ~ wt + hp, data = mtcars)
cat(round(coef(fit), 3), "\n")
cat(round(summary(fit)$r.squared, 3), round(summary(fit)$adj.r.squared, 3), "\n")
new <- data.frame(wt = 3, hp = 120)
cat(round(predict(fit, new), 2), "\n")
g <- glm(am ~ wt, data = mtcars, family = binomial)
cat(round(coef(g), 3), round(predict(g, data.frame(wt = 3), type = "response"), 3), "\n")
```

**Answer:** 37.227 -3.878 -0.032 
0.827 0.815 
21.78 
12.04 -4.024 0.492

Linear and logistic models. Verified by compiling and running it.

**Q90.** What does a residual plot check?

**Answer:** Whether model assumptions (linearity, constant variance) hold

Diagnostics.

**Q91.** When use logistic regression?

**Answer:** For a binary outcome, modelling log-odds

Classification.

**Q92.** What is overfitting?

**Answer:** A model that fits noise and generalizes poorly; use validation

Train/test.

### [04 · R Markdown Reports](level-3/04-r-markdown-reports.md)

**Q93.** What does R Markdown produce?

**Answer:** Documents that combine text and executed R code (HTML, PDF, Word)

Reproducible reports.

**Q94.** What does knitr do?

**Answer:** Executes code chunks and weaves output into the document

Engine.

**Q95.** How hide code but show output?

**Answer:** Chunk option echo = FALSE

Chunk options.

**Q96.** Why parameterized reports?

**Answer:** Generate many versions from one template (e.g. per region)

params in YAML.

### [05 · Shiny Basics](level-3/05-shiny-basics.md)

**Q97.** What are the two parts of a Shiny app?

**Answer:** UI (layout) and server (logic)

Reactive.

**Q98.** What is reactivity?

**Answer:** Outputs automatically update when the inputs they depend on change

Dependency graph.

**Q99.** What does reactive() do?

**Answer:** Creates a cached expression that recomputes only when dependencies change

Efficiency.

**Q100.** Why avoid heavy work inside render functions?

**Answer:** It reruns on every dependency change; cache or precompute

Performance.

### [06 · Performance &amp; Vectorization](level-3/06-performance-vectorization.md)

**Q101.** What does this print?

```r
x <- 1:100000
f_loop <- function(x) { out <- numeric(length(x)); for (i in seq_along(x)) out[i] <- x[i]^2; out }
f_vec <- function(x) x^2
cat(identical(f_loop(x), f_vec(x)), "\n")
g <- function(n) { v <- c(); for (i in 1:n) v <- c(v, i); length(v) }
cat(g(1000), "\n")
cat(is.na(sum(c(1, NA))), sum(c(1, NA), na.rm = TRUE), "\n")
cat(system.time(for(i in 1:10) NULL)[["elapsed"]] >= 0, "\n")
```

**Answer:** TRUE 
1000 
TRUE 1 
TRUE

Vectorized and preallocated code give the same result faster. Verified by compiling and running it.

**Q102.** Why preallocate vectors?

**Answer:** Growing a vector in a loop copies it each time

O(n²).

**Q103.** Which tool profiles R code?

**Answer:** profvis or Rprof

Measure first.

**Q104.** What is vectorization?

**Answer:** Operating on whole vectors with optimized compiled code instead of loops

Idiomatic R.

### [07 · Package Development Basics](level-3/07-package-development-basics.md)

**Q105.** What does DESCRIPTION list?

**Answer:** Package name, version, authors, license and dependencies

Metadata.

**Q106.** What does R CMD check do?

**Answer:** Runs a comprehensive set of package checks (docs, tests, examples)

CRAN readiness.

**Q107.** What does NAMESPACE control?

**Answer:** Which functions are exported and which are imported

Generated by roxygen2.

**Q108.** Where do user-facing docs live?

**Answer:** man/*.Rd generated from roxygen comments

Maintainability.

### [08 · Calling APIs with httr](level-3/08-apis-httr.md)

**Q109.** Which package makes HTTP requests?

**Answer:** httr (or httr2)

APIs.

**Q110.** What does this print?

```r
suppressPackageStartupMessages(library(jsonlite))
j <- '{"name":"Ann","tags":["a","b"],"n":{"k":1.5}}'
x <- fromJSON(j)
cat(x$name, x$tags, x$n$k, "\n")
cat(toJSON(list(a = 1, b = "x"), auto_unbox = TRUE), "\n")
cat(toJSON(data.frame(i = 1:2, s = c("p", "q"))), "\n")
```

**Answer:** Ann a b 1.5 
{"a":1,"b":"x"} 
[{"i":1,"s":"p"},{"i":2,"s":"q"}]

JSON conversion helpers. Verified by compiling and running it.

**Q111.** How handle API errors?

**Answer:** Check status codes and use retry/backoff

Robustness.

**Q112.** Where store API keys?

**Answer:** Environment variables (e.g. in .Renviron), never in code

Security.

### [09 · Advanced Testing with testthat](level-3/09-testing-advanced.md)

**Q113.** What does this print?

```r
suppressPackageStartupMessages(library(testthat))
library(testthat)
fake_get <- function(url) list(status = 200, body = "{\"n\": 3}")
fetch_n <- function(url, get = fake_get) {
  r <- get(url)
  if (r$status != 200) stop("HTTP ", r$status)
  jsonlite::fromJSON(r$body)$n
}
expect_equal(fetch_n("x"), 3)
bad <- function(url) list(status = 500, body = "")
expect_error(fetch_n("x", bad), "HTTP 500")
res <- testthat::with_reporter("silent", { test_that("ok", { expect_true(TRUE) }) })
cat("passed\n")
```

**Answer:** passed

Dependency injection makes HTTP code testable. Verified by compiling and running it.

**Q114.** What does a test fixture/helper provide?

**Answer:** Reusable setup shared across tests

helper-*.R.

**Q115.** What is snapshot testing?

**Answer:** Comparing output to a stored reference (expect_snapshot)

Detect changes.

**Q116.** What are mocks for?

**Answer:** Replacing external dependencies (network) with controlled stand-ins

testthat::local_mocked_bindings.

### [10 · Project: Shiny Sales Dashboard](level-3/10-project-shiny-dashboard.md)

**Q117.** What does the Shiny dashboard project combine?

**Answer:** Data loading, reactive filters, plots/tables and deployment

Level 3 project.

**Q118.** How deploy a Shiny app?

**Answer:** shinyapps.io, Posit Connect or a server with Shiny Server/Docker

Hosting.

**Q119.** How keep it fast?

**Answer:** Precompute data, cache reactives and avoid recomputation

Performance.

**Q120.** How test it?

**Answer:** shinytest2 and unit tests for helper functions

Reliability.

## Level 4 · Master

### [01 · Advanced Statistical Methods](level-4/01-advanced-statistical-methods.md)

**Q121.** What does this print?

```r
set.seed(42)
x <- rnorm(100, mean = 5, sd = 2)
b <- replicate(500, mean(sample(x, replace = TRUE)))
ci <- quantile(b, c(.025, .975))
cat(length(b), round(mean(x), 2), ci[1] < mean(x) && mean(x) < ci[2], "\n")
w <- wilcox.test(c(1.1, 2.3, 3.2, 4.8), c(2.0, 3.9, 5.1, 6.4))
cat(round(w$p.value, 3), "\n")
```

**Answer:** 500 5.07 TRUE 
0.343

Bootstrapping and non-parametric tests. Verified by compiling and running it.

**Q122.** When prefer a bootstrap?

**Answer:** When the sampling distribution is unknown or complex

Resampling.

**Q123.** What is multiple-comparison correction for?

**Answer:** Controlling false positives when running many tests

p.adjust.

**Q124.** What is a confidence interval?

**Answer:** A range of plausible values for a parameter given the data and method

Interpretation.

### [02 · Production R with plumber](level-4/02-production-plumber.md)

**Q125.** What does plumber do?

**Answer:** Turns R functions into HTTP API endpoints via annotations

#* @get /path.

**Q126.** What should a production API return on bad input?

**Answer:** A 4xx status with a clear error message

Validation.

**Q127.** How run plumber in production?

**Answer:** In a container behind a reverse proxy with logging and health checks

Deployment.

**Q128.** How version an API?

**Answer:** Path prefixes or headers and a deprecation policy

Compatibility.

### [03 · Shiny at Scale (Modules)](level-4/03-shiny-at-scale.md)

**Q129.** How scale a Shiny app?

**Answer:** Cache, use async/future, split modules and run multiple processes

Architecture.

**Q130.** What are Shiny modules?

**Answer:** Reusable UI+server components with namespaced IDs

Organize big apps.

**Q131.** What does promises/future allow?

**Answer:** Running slow work asynchronously so the app stays responsive

Concurrency.

**Q132.** What should be monitored?

**Answer:** Load time, errors and active sessions

Operations.

### [04 · Testing at Scale &amp; CI](level-4/04-testing-at-scale-ci.md)

**Q133.** What should CI run for an R package?

**Answer:** R CMD check, tests and lintr/styler on multiple R versions

usethis::use_github_action.

**Q134.** What does covr measure?

**Answer:** Test coverage of package code

Quality signal.

**Q135.** Why test on multiple OSes?

**Answer:** Behavior and dependencies can differ across platforms

Matrix builds.

**Q136.** What gates a release?

**Answer:** Passing checks and reviewed NEWS/documentation

Process.

### [05 · Performance Optimization (Rcpp Intro)](level-4/05-performance-rcpp.md)

**Q137.** What does Rcpp let you do?

**Answer:** Call fast C++ code from R

Performance.

**Q138.** When is Rcpp worth it?

**Answer:** For tight loops that can't be vectorized, after profiling

Measure first.

**Q139.** What does sourceCpp do?

**Answer:** Compiles and loads a C++ file for use in R

Quick iteration.

**Q140.** What is a cost of Rcpp?

**Answer:** Compilation toolchain and C++ maintenance burden

Trade-offs.

### [06 · Reproducible Research Practices](level-4/06-reproducible-research.md)

**Q141.** What makes research reproducible?

**Answer:** Version-controlled code, pinned dependencies and a documented run order

renv + git.

**Q142.** What does set.seed() do?

**Answer:** Makes random number generation repeatable

Simulations.

**Q143.** What does a targets pipeline give you?

**Answer:** Dependency-aware rebuilds so only changed steps rerun

Workflow.

**Q144.** What belongs in a research compendium?

**Answer:** Data (or access instructions), code, environment and docs

Share.

### [07 · Data Visualization at Scale](level-4/07-data-viz-at-scale.md)

**Q145.** How plot millions of points?

**Answer:** Aggregate or bin (hexbin, ggplot2 stat_bin2d) or use rasterization

Overplotting.

**Q146.** What does ggrepel solve?

**Answer:** Overlapping text labels

Readability.

**Q147.** Why use consistent themes?

**Answer:** Visual coherence across reports and dashboards

Branding.

**Q148.** How make plots accessible?

**Answer:** Colour-blind-safe palettes, contrast and alt text

Inclusion.

### [08 · Working with Big Data in R (data.table)](level-4/08-big-data-datatable.md)

**Q149.** What does this print?

```r
suppressPackageStartupMessages(library(data.table))
dt <- data.table(g = c("a","a","b","b","c"), v = c(1, 2, 3, 4, 5))
print(dt[v > 1, .(n = .N, total = sum(v)), by = g][order(-total)])
dt[, w := v * 2]
cat(names(dt), "\n")
setkey(dt, g)
print(dt["b"])
```

**Answer:** g     n total
   &lt;char&gt; &lt;int&gt; &lt;num&gt;
1:      b     2     7
2:      c     1     5
3:      a     1     2
g v w 
Key: &lt;g&gt;
        g     v     w
   &lt;char&gt; &lt;num&gt; &lt;num&gt;
1:      b     3     6
2:      b     4     8

data.table: concise grouping and by-reference updates. Verified by compiling and running it.

**Q150.** Why is data.table fast on big data?

**Answer:** Optimized C code, keys/indexes and modification by reference

Memory efficiency.

**Q151.** What does := do?

**Answer:** Adds or updates columns by reference without copying

Careful with side effects.

**Q152.** What is arrow useful for?

**Answer:** Reading Parquet/large files lazily and efficiently

Larger-than-memory.

### [09 · Package Publishing (CRAN Basics)](level-4/09-package-publishing-cran.md)

**Q153.** What does CRAN require of packages?

**Answer:** Passing R CMD check with no errors or warnings, plus docs and a license

Policies.

**Q154.** What does usethis help with?

**Answer:** Automating package setup tasks like tests, docs and CI

Workflow.

**Q155.** What should a NEWS file record?

**Answer:** User-visible changes per version

Communication.

**Q156.** What does semantic versioning communicate?

**Answer:** Compatibility: major breaking, minor features, patch fixes

Expectations.

### [10 · Capstone Project](level-4/10-capstone-project.md)

**Q157.** What does the capstone combine?

**Answer:** Data pipeline, analysis, tests, package/app and reproducible reporting

Level 4 project.

**Q158.** How prove it is reproducible?

**Answer:** Run it from scratch in a clean environment (CI or container)

Evidence.

**Q159.** What documentation helps?

**Answer:** README, usage examples and a data dictionary

Onboarding.

**Q160.** What sustains a project?

**Answer:** Tests, dependency updates and clear ownership

Maintenance.

## Mixed questions across all levels

**Q161.** What does this print?

```r
x <- c(5, NA, 3)
cat(sum(x), sum(x, na.rm = TRUE), mean(x, na.rm = TRUE), is.na(x[2]), "\n")
```

**Answer:** NA 8 4 TRUE

NA propagates unless na.rm = TRUE. Verified by compiling and running it.

**Q162.** What does this print?

```r
x <- c(a = 1, b = 2, c = 3)
cat(x[-1], x[c("a", "c")], x[x > 1], length(x[0]), "\n")
```

**Answer:** 2 3 1 3 2 3 0

Negative, named and logical indexing. Verified by compiling and running it.

**Q163.** What does this print?

```r
f <- function(n) { total <- 0; for (i in seq_len(n)) total <- total + i; total }
cat(f(0), f(4), "\n")
```

**Answer:** 0 10

seq_len(0) loops zero times. Verified by compiling and running it.

**Q164.** What does this print?

```r
suppressPackageStartupMessages(library(dplyr))
d <- data.frame(g = c("a", "a", "b"), v = c(1, 2, 10))
r <- d %>% group_by(g) %>% summarise(m = mean(v), n = n(), .groups = "drop")
print(as.data.frame(r))
```

**Answer:** g    m n
1 a  1.5 2
2 b 10.0 1

Group summaries. Verified by compiling and running it.

**Q165.** What does this print?

```r
x <- c(10, 20, 30)
names(x) <- c("a", "b", "c")
lst <- list(x = x, n = 3)
cat(lst$x[["b"]], lst[["n"]], class(lst["n"]), "\n")
```

**Answer:** 20 3 list

[ keeps a list; [[ extracts. Verified by compiling and running it.

**Q166.** What does this print?

```r
s <- c("apple pie", "banana", "cherry tart")
cat(sapply(strsplit(s, " "), length), toupper(substr(s, 1, 1)), grepl("an", s), "\n")
```

**Answer:** 2 1 2 A B C FALSE TRUE FALSE

strsplit, substr and grepl. Verified by compiling and running it.

**Q167.** What does this print?

```r
set.seed(7)
x <- sample(1:10, 5)
set.seed(7)
y <- sample(1:10, 5)
cat(identical(x, y), length(unique(x)), "\n")
```

**Answer:** TRUE 5

set.seed makes sampling reproducible. Verified by compiling and running it.

**Q168.** What does this print?

```r
f <- function(x, ...) UseMethod("f")
f.default <- function(x, ...) "default"
f.numeric <- function(x, ...) "number"
f.character <- function(x, ...) "text"
cat(f(1), f("a"), f(TRUE), "\n")
```

**Answer:** number text default

S3 dispatch on class. Verified by compiling and running it.

**Q169.** A script works on your laptop but not on a colleague's. What helps most?

**Answer:** Pin package versions with renv and document the R version

Reproducibility.

**Q170.** A for loop over 1:length(x) fails when x is empty. What's the fix?

**Answer:** Use seq_along(x) or seq_len(length(x))

1:0 is c(1, 0).

**Q171.** A join produces more rows than expected. What do you check?

**Answer:** Duplicate keys on either side

Many-to-many joins multiply rows.

**Q172.** What is the safest way to read a CSV where IDs have leading zeros?

**Answer:** Read the column as character (colClasses or col_types)

Avoid losing zeros.

**Q173.** When is a loop fine in R?

**Answer:** When logic depends on previous iterations or clarity matters more than speed

Preallocate.

**Q174.** What does expect_equal use for numbers?

**Answer:** A tolerance, so tiny floating point differences pass

Use expect_identical for exact.

**Q175.** Why write R functions with stopifnot or clear errors?

**Answer:** Fail early with understandable messages when inputs are invalid

Defensive code.

**Q176.** What belongs in tests/testthat?

**Answer:** Automated tests named test-*.R using expect_* functions

Package testing.

**Q177.** What makes a Shiny app slow?

**Answer:** Recomputing heavy work on every reactive update

Cache and precompute.

**Q178.** What does an R Markdown report give you?

**Answer:** Narrative plus code and results in one reproducible document

knitr.

**Q179.** What do you do before trusting a regression?

**Answer:** Check residuals, outliers and assumptions, not just p-values

Diagnostics.

**Q180.** A p-value is 0.04 with n = 20000. What matters too?

**Answer:** Effect size and practical significance

Big samples find tiny effects.

**Q181.** How do you speed up grouped aggregation on 50M rows?

**Answer:** Use data.table (or arrow/duckdb) instead of copying data repeatedly

Efficient engines.

**Q182.** What does plumber add to R code?

**Answer:** An HTTP API around your functions

Deployment.

## Practical coding questions

### Clean, validate and summarise a messy customer file

A raw customer table has duplicates, messy names and emails, mixed date formats and bad numbers. Standardise it with dplyr/stringr/lubridate, flag each bad row with a reason (case_when), split good from rejected, and report counts, reject reasons, the good rows, monthly revenue, the total and the days since the first signup.

Starter code:

```
suppressPackageStartupMessages({ library(dplyr); library(stringr); library(lubridate); library(tidyr) })

raw <- data.frame(
  id    = c(1, 2, 2, 3, 4, 5, 6),
  name  = c(" Ann Lee ", "bob  stone", "bob  stone", "CY ray", NA, "Dee Fox", "eli  park"),
  email = c("ann@x.com", "BOB@X.COM", "BOB@X.COM", "cy-at-x.com", "dee@x.com", "dee2@x.com", "eli@x.com"),
  signup = c("2026-01-05", "2026/02/10", "2026/02/10", "2026-03-01", "not a date", "2026-04-15", "2026-04-20"),
  spend = c("120.50", "75", "75", "abc", "40", "-5", "300"),
  stringsAsFactors = FALSE
)

# TODO 1: remove duplicate rows (distinct)
# TODO 2: standardise: name -> squish whitespace + title case; email -> lower case; signup -> Date (formats like 2026-01-05 or 2026/02/10);
#         spend -> numeric (non-numeric becomes NA without printing a warning)
# TODO 3: add a `problem` column using case_when in this order:
#         "missing name", "bad email" (must look like x@y.z), "bad date", "bad spend", "negative spend", otherwise NA
# TODO 4: good = rows without a problem (signup as a Date); bad = rows with one
# TODO 5: print:  raw rows / after distinct / good / rejected, the reject reasons as id=reason,
#         the good rows (id, name, email, signup, spend), monthly customers and revenue, total revenue,
#         and days from the first signup to 2026-05-01
```

Expected output:

```text
raw rows: 7 | after distinct: 6 | good: 3 | rejected: 3 
reject reasons: 3=bad email, 4=missing name, 5=negative spend 
  id      name     email     signup spend
1  1   Ann Lee ann@x.com 2026-01-05 120.5
2  2 Bob Stone bob@x.com 2026-02-10  75.0
3  6  Eli Park eli@x.com 2026-04-20 300.0
    month customers revenue
1 2026-01         1   120.5
2 2026-02         1    75.0
3 2026-04         1   300.0
total revenue: 495.5 
days since first signup (to 2026-05-01): 116
```

??? success "Show a solution"

    ```
    suppressPackageStartupMessages({ library(dplyr); library(stringr); library(lubridate); library(tidyr) })

    raw <- data.frame(
      id    = c(1, 2, 2, 3, 4, 5, 6),
      name  = c(" Ann Lee ", "bob  stone", "bob  stone", "CY ray", NA, "Dee Fox", "eli  park"),
      email = c("ann@x.com", "BOB@X.COM", "BOB@X.COM", "cy-at-x.com", "dee@x.com", "dee2@x.com", "eli@x.com"),
      signup = c("2026-01-05", "2026/02/10", "2026/02/10", "2026-03-01", "not a date", "2026-04-15", "2026-04-20"),
      spend = c("120.50", "75", "75", "abc", "40", "-5", "300"),
      stringsAsFactors = FALSE
    )

    clean <- raw %>%
      distinct() %>%
      mutate(
        name   = str_to_title(str_squish(name)),
        email  = str_to_lower(email),
        signup = parse_date_time(signup, orders = c("Ymd")),
        spend  = suppressWarnings(as.numeric(spend)),
        problem = case_when(
          is.na(name) ~ "missing name",
          !str_detect(email, "^[^@\\s]+@[^@\\s]+\\.[a-z]+$") ~ "bad email",
          is.na(signup) ~ "bad date",
          is.na(spend) ~ "bad spend",
          spend < 0 ~ "negative spend",
          TRUE ~ NA_character_
        )
      )

    good <- clean %>% filter(is.na(problem)) %>% mutate(signup = as_date(signup))
    bad  <- clean %>% filter(!is.na(problem))

    cat("raw rows:", nrow(raw), "| after distinct:", nrow(distinct(raw)), "| good:", nrow(good), "| rejected:", nrow(bad), "\n")
    cat("reject reasons:", paste(bad$id, bad$problem, sep = "=", collapse = ", "), "\n")
    print(as.data.frame(good %>% select(id, name, email, signup, spend)))

    monthly <- good %>% mutate(month = format(signup, "%Y-%m")) %>%
      group_by(month) %>% summarise(customers = n(), revenue = sum(spend), .groups = "drop")
    print(as.data.frame(monthly))
    cat("total revenue:", sum(good$spend), "\n")
    cat("days since first signup (to 2026-05-01):", as.integer(as_date("2026-05-01") - min(good$signup)), "\n")
    ```

    A tidy pipeline standardises first, then validates with ordered rules so each bad row gets exactly one reason, and nothing is silently dropped. Verified by running the script with R 4.6 and the tidyverse packages.

### Write and test functions: moving average, vectorised running max and a bootstrap interval

Implement moving_avg (validated, vapply), a loop and a vectorised running maximum, and a reproducible bootstrap confidence interval. The script's check() helper runs testthat expectations and prints PASS/FAIL; one expectation is deliberately wrong to show a failure.

Starter code:

```
suppressPackageStartupMessages(library(testthat))

# ---- code under test ----
# TODO: implement
#  moving_avg(x, k): validate (numeric x, whole number k >= 1 -> stopifnot), return numeric(0) when k > length(x),
#                    otherwise the mean of every window of k consecutive values (use vapply)
#  cum_max_loop(x): running maximum using a loop; cum_max_vec(x): the vectorised version
#  bootstrap_ci(x, stat = mean, reps = 400, level = 0.95, seed = 1): set.seed(seed), resample with replacement `reps` times,
#                    return the unnamed lower and upper quantiles
moving_avg <- function(x, k) NULL
cum_max_loop <- function(x) NULL
cum_max_vec <- function(x) NULL
bootstrap_ci <- function(x, stat = mean, reps = 400, level = 0.95, seed = 1) NULL

# ---- checks (each prints PASS/FAIL instead of stopping) ----
check <- function(name, expr) {
  ok <- tryCatch({ force(expr); TRUE }, error = function(e) FALSE)
  cat(sprintf("%-34s %s\n", name, if (ok) "PASS" else "FAIL"))
  invisible(ok)
}
x <- c(3, 1, 4, 1, 5, 9, 2, 6)
check("moving_avg basic", expect_equal(moving_avg(x, 3), c(8/3, 2, 10/3, 5, 16/3, 17/3)))
check("moving_avg window = length", expect_equal(moving_avg(x, 8), mean(x)))
check("moving_avg window too large", expect_equal(moving_avg(x, 9), numeric(0)))
check("moving_avg rejects bad k", expect_error(moving_avg(x, 0)))
check("moving_avg rejects text", expect_error(moving_avg("a", 2)))
check("loop and vectorised agree", expect_identical(cum_max_loop(x), cum_max_vec(x)))
ci <- bootstrap_ci(x)
check("bootstrap interval ordered", expect_lt(ci[1], ci[2]))
check("bootstrap reproducible", expect_equal(bootstrap_ci(x), bootstrap_ci(x)))
check("bootstrap contains the mean", expect_true(ci[1] <= mean(x) && mean(x) <= ci[2]))
check("deliberately wrong expectation", expect_equal(moving_avg(x, 2)[1], 99))

cat("moving_avg(x, 3):", round(moving_avg(x, 3), 2), "\n")
cat("cummax:", cum_max_vec(x), "\n")
cat("95% bootstrap CI for the mean:", round(ci, 2), "\n")
big <- as.numeric(1:200000)
cat("same answer on 200k values:", identical(cum_max_loop(big), cum_max_vec(big)), "\n")
```

Expected output:

```text
moving_avg basic                   PASS
moving_avg window = length         PASS
moving_avg window too large        PASS
moving_avg rejects bad k           PASS
moving_avg rejects text            PASS
loop and vectorised agree          PASS
bootstrap interval ordered         PASS
bootstrap reproducible             PASS
bootstrap contains the mean        PASS
deliberately wrong expectation     FAIL
moving_avg(x, 3): 2.67 2 3.33 5 5.33 5.67 
cummax: 3 3 4 4 5 9 9 9 
95% bootstrap CI for the mean: 2.12 6 
same answer on 200k values: TRUE
```

??? success "Show a solution"

    ```
    suppressPackageStartupMessages(library(testthat))

    # ---- code under test ----
    moving_avg <- function(x, k) {
      stopifnot(is.numeric(x), k >= 1, k == round(k))
      if (k > length(x)) return(numeric(0))
      vapply(seq_len(length(x) - k + 1), function(i) mean(x[i:(i + k - 1)]), numeric(1))
    }
    cum_max_loop <- function(x) { out <- numeric(length(x)); m <- -Inf; for (i in seq_along(x)) { m <- max(m, x[i]); out[i] <- m }; out }
    cum_max_vec <- function(x) cummax(x)

    bootstrap_ci <- function(x, stat = mean, reps = 400, level = 0.95, seed = 1) {
      set.seed(seed)
      b <- replicate(reps, stat(sample(x, replace = TRUE)))
      unname(quantile(b, c((1 - level) / 2, 1 - (1 - level) / 2)))
    }

    # ---- checks (each prints PASS/FAIL instead of stopping) ----
    check <- function(name, expr) {
      ok <- tryCatch({ force(expr); TRUE }, error = function(e) FALSE)
      cat(sprintf("%-34s %s\n", name, if (ok) "PASS" else "FAIL"))
      invisible(ok)
    }
    x <- c(3, 1, 4, 1, 5, 9, 2, 6)
    check("moving_avg basic", expect_equal(moving_avg(x, 3), c(8/3, 2, 10/3, 5, 16/3, 17/3)))
    check("moving_avg window = length", expect_equal(moving_avg(x, 8), mean(x)))
    check("moving_avg window too large", expect_equal(moving_avg(x, 9), numeric(0)))
    check("moving_avg rejects bad k", expect_error(moving_avg(x, 0)))
    check("moving_avg rejects text", expect_error(moving_avg("a", 2)))
    check("loop and vectorised agree", expect_identical(cum_max_loop(x), cum_max_vec(x)))
    ci <- bootstrap_ci(x)
    check("bootstrap interval ordered", expect_lt(ci[1], ci[2]))
    check("bootstrap reproducible", expect_equal(bootstrap_ci(x), bootstrap_ci(x)))
    check("bootstrap contains the mean", expect_true(ci[1] <= mean(x) && mean(x) <= ci[2]))
    check("deliberately wrong expectation", expect_equal(moving_avg(x, 2)[1], 99))

    cat("moving_avg(x, 3):", round(moving_avg(x, 3), 2), "\n")
    cat("cummax:", cum_max_vec(x), "\n")
    cat("95% bootstrap CI for the mean:", round(ci, 2), "\n")
    big <- as.numeric(1:200000)
    cat("same answer on 200k values:", identical(cum_max_loop(big), cum_max_vec(big)), "\n")
    ```

    Validation with stopifnot fails early, vapply guarantees the result type, vectorised code gives the same answer as a loop but faster, and a fixed seed makes bootstrapping reproducible. Verified by running the script with R 4.6 and testthat.

---

More free practice: the [R Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
