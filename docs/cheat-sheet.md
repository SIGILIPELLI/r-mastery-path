---
title: "R Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free R cheat sheet: 34 quick-reference tables from every level of the R Mastery Path, each linked to its full lesson."
---

# R Cheat Sheet

Quick-reference tables from **34 lessons** of the free R Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Entry

### [02 · Variables &amp; Types](level-1/02-variables-types.md)

| Type | Example | `class()` |
|------|---------|-----------|
| Double | `3.14`, `42` | `"numeric"` |
| Integer | `42L` | `"integer"` |
| Character | `"hello"` | `"character"` |
| Logical | `TRUE`, `FALSE` | `"logical"` |
| Missing | `NA` | type of the containing vector |
| Nothing | `NULL` | `"NULL"` |

### [03 · Control Flow](level-1/03-control-flow.md)

| Task | Loop style | Vectorized style |
|------|------------|-------------------|
| Double every element | `for (i in seq_along(x)) x[i] <- x[i] * 2` | `x <- x * 2` |
| Classify by threshold | `for` loop with `if`/`else` per element | `ifelse(x >= t, "high", "low")` |
| Sum all elements | `total <- 0; for (v in x) total <- total + v` | `sum(x)` |

### [04 · Functions](level-1/04-functions.md)

| Concept | Syntax |
|---------|--------|
| Define | `f <- function(x, y = 1) { ... }` |
| Call positionally | `f(10, 20)` |
| Call by name | `f(y = 20, x = 10)` |
| Variadic args | `f <- function(...) { c(...) }` |
| Anonymous function | `function(x) x + 1` or `\(x) x + 1` |
| Early exit | `return(value)` |
| Modify outer scope | `x <<- new_value` (avoid unless needed) |

### [05 · Vectors &amp; Basic Data Structures](level-1/05-vectors-data-structures.md)

| Task | Syntax |
|------|--------|
| Create a vector | `c(1, 2, 3)` |
| Sequence | `1:10` or `seq(0, 100, by = 25)` |
| First element | `x[1]` (1-based!) |
| Last element | `x[length(x)]` |
| Filter by condition | `x[x > 10]` |
| Named access | `x["name"]` |
| Create a list | `list(a = 1, b = "two")` |
| Get list element (unwrapped) | `lst[["a"]]` or `lst$a` |
| Get list element (still a list) | `lst["a"]` |

### [06 · Data Frames Basics](level-1/06-data-frames-basics.md)

| Task | Syntax |
|------|--------|
| Create | `data.frame(col1 = ..., col2 = ...)` |
| Inspect structure | `str(df)` |
| Dimensions | `nrow(df)`, `ncol(df)`, `dim(df)` |
| Column as vector | `df$col` or `df[["col"]]` or `df[, "col"]` |
| Column as data frame | `df["col"]` |
| Filter rows | `df[df$col > 10, ]` |
| Add/modify column | `df$new_col <- ...` |
| Add row | `rbind(df, new_row_df)` |
| Sort | `df[order(df$col), ]` |
| Group + summarize | `tapply(df$value, df$group, mean)` |

### [08 · Basic Plotting](level-1/08-basic-plotting.md)

| Function | Use for |
|----------|---------|
| `plot(x, y)` | Scatter plot of two numeric vectors |
| `plot(x, y, type = "l")` | Line plot |
| `barplot(x)` | Bar chart of a (named) vector |
| `hist(x)` | Distribution of one numeric vector |
| `boxplot(x, y, ...)` | Compare distributions across groups |
| `png()` / `dev.off()` | Save a plot to a file |
| `par(mfrow = c(r, c))` | Arrange multiple plots in a grid |

Base R plotting is quick and dependency-free, which is why it's introduced
first. [Level 2](level-2/04-ggplot2-basics.md) introduces **ggplot2**, the
more expressive and widely used plotting package for anything beyond a quick
look at your data.

### [09 · Packages](level-1/09-packages.md)

| Task | Command |
|------|---------|
| Install | `install.packages("pkgname")` |
| Load | `library(pkgname)` |
| Load or error clearly if missing | `require(pkgname)` |
| Call without loading | `pkgname::function_name()` |
| List installed | `installed.packages()[, "Package"]` |
| Update all | `update.packages()` |
| Remove | `remove.packages("pkgname")` |

## Level 2 · Intermediate

### [01 · Data Frames Deep Dive (dplyr)](level-2/01-dplyr-deep-dive.md)

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

### [02 · Data Cleaning](level-2/02-data-cleaning.md)

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

### [03 · Apply Family Functions](level-2/03-apply-family-functions.md)

| Function | Input | Output |
|----------|-------|--------|
| `sapply(x, f)` | vector/list | simplified vector/matrix (or list if it can't simplify) |
| `lapply(x, f)` | vector/list | **always** a list |
| `vapply(x, f, FUN.VALUE)` | vector/list | vector, with a guaranteed, declared shape |
| `mapply(f, x, y)` | multiple vectors, walked together | simplified vector/matrix |
| `apply(m, MARGIN, f)` | matrix/data frame | vector/matrix, one result per row (`MARGIN=1`) or column (`MARGIN=2`) |

### [04 · ggplot2 Basics](level-2/04-ggplot2-basics.md)

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

### [05 · Working with Strings (stringr)](level-2/05-strings-stringr.md)

| Task | Function |
|------|----------|
| Length | `str_length(x)` |
| Trim whitespace | `str_trim(x)` |
| Change case | `str_to_lower(x)`, `str_to_upper(x)`, `str_to_title(x)` |
| Test for a pattern | `str_detect(x, pattern)` |
| Count occurrences | `str_count(x, pattern)` |
| Extract first match | `str_extract(x, pattern)` |
| Extract capture groups | `str_match(x, pattern)` |
| Replace first match | `str_replace(x, pattern, replacement)` |
| Replace all matches | `str_replace_all(x, pattern, replacement)` |
| Split | `str_split(x, pattern)` (list) or `str_split_fixed(x, pattern, n)` (matrix) |
| Concatenate | `str_c(..., sep = "")` |
| Join a vector into one string | `str_c(x, collapse = ", ")` |
| Pad | `str_pad(x, width, pad = "0")` |
| Match literally, not as regex | `str_detect(x, fixed("..."))` |

### [06 · Statistical Basics](level-2/06-statistical-basics.md)

| Task | Function |
|------|----------|
| Mean / median / sd / variance | `mean()`, `median()`, `sd()`, `var()` |
| Five-number summary | `summary(x)` |
| Percentiles | `quantile(x, probs = c(...))` |
| Linear correlation | `cor(x, y)` |
| Rank correlation (outlier-robust) | `cor(x, y, method = "spearman")` |
| Compare two group means | `t.test(a, b)` |
| Compare paired before/after | `t.test(before, after, paired = TRUE)` |
| Fit a linear model | `lm(y ~ x, data = df)` |
| Inspect a model's fit | `summary(model)` |
| Predict new values | `predict(model, newdata = ...)` |

### [07 · Writing Functions &amp; Package Structure](level-2/07-functions-package-structure.md)

| Goal | How |
|------|-----|
| Reject bad input immediately | `stop("clear message")` |
| Warn but continue | `warning("message")` |
| Document a function | `#' Title`, `@param`, `@return`, `@examples` |
| Split functions across files | one file per theme in `R/`, `source()`'d |
| Scaffold a real package | `usethis::create_package("path")` |
| Auto-generate docs/NAMESPACE | `devtools::document()` |
| Load a package for testing | `devtools::load_all()` |
| Validate a package | `devtools::check()` (wraps `R CMD check`) |

### [08 · Testing with testthat](level-2/08-testing-testthat.md)

| Task | Code |
|------|------|
| Load the package | `library(testthat)` |
| Group related checks | `test_that("description", { ... })` |
| Equal (numeric-tolerant) | `expect_equal(a, b)` |
| Exactly identical | `expect_identical(a, b)` |
| Logical checks | `expect_true(x)`, `expect_false(x)` |
| Confirm an error/warning happens | `expect_error(expr)`, `expect_warning(expr)` |
| Run all tests in a package | `devtools::test()` |
| Run tests from the CLI | `testthat::test_dir("tests/testthat")` |

### [09 · Working with Dates (lubridate)](level-2/09-dates-lubridate.md)

| Task | Function |
|------|----------|
| Parse `YYYY-MM-DD` | `ymd(x)` |
| Parse `MM/DD/YYYY` | `mdy(x)` |
| Parse `DD/MM/YYYY` | `dmy(x)` |
| Parse with time | `ymd_hms(x)` |
| Extract component | `year(x)`, `month(x)`, `day(x)`, `wday(x, label=TRUE)` |
| Add calendar time | `x + days(n)`, `x + months(n)`, `x + years(n)` |
| Add exact elapsed time | `x + ddays(n)`, `x + dhours(n)` |
| Round to a unit | `floor_date(x, "month")`, `ceiling_date(x, "week")` |
| Format for display | `format(x, "%B %d, %Y")` (reliable; `stamp("example")(x)` can guess wrong) |
| Convert display time zone | `with_tz(x, "Zone/Name")` |

## Level 3 · Advanced

### [01 · Advanced dplyr/tidyr](level-3/01-advanced-dplyr-tidyr.md)

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

### [02 · Advanced ggplot2](level-3/02-advanced-ggplot2.md)

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

### [03 · Statistical Modeling](level-3/03-statistical-modeling.md)

| Task | Function |
|---|---|
| Linear regression | `lm(y ~ x, data = df)` |
| Logistic regression | `glm(y ~ x, data = df, family = binomial)` |
| Tidy per-coefficient output | `broom::tidy(fit)` |
| Tidy whole-model summary | `broom::glance(fit)` |
| Predict on new data | `predict(fit, newdata, interval = "confidence")` |
| Compare two group means | `t.test(a, b)` |
| Change a factor's reference level | `relevel(f, ref = "level")` |
| Log-odds → odds ratio | `exp(coef(fit))` |

### [04 · R Markdown Reports](level-3/04-r-markdown-reports.md)

| Task | How |
|---|---|
| Render a report | `rmarkdown::render("file.Rmd")` |
| Pass different inputs to the same template | `params:` in YAML + `params = list(...)` at render time |
| Show output but hide the code | chunk option `echo = FALSE` |
| Run code with no output at all shown | chunk option `include = FALSE` |
| Show code without running it | chunk option `eval = FALSE` |
| Render a well-formatted table | `knitr::kable(df)` |
| Inline a computed value in a sentence | `` `r expression` `` |

### [05 · Shiny Basics](level-3/05-shiny-basics.md)

| Task | Function |
|---|---|
| Define the app's layout | `fluidPage(...)` |
| Read a widget's current value | `input$<id>` |
| Shared, cached, auto-updating value | `reactive({...})` |
| Render text output | `renderText({...})` / `textOutput("id")` |
| Render a plot | `renderPlot({...})` / `plotOutput("id")` |
| Render a table | `renderTable({...})` / `tableOutput("id")` |
| Start the app | `shinyApp(ui, server)` |
| Test server logic headlessly | `testServer(server, {...})` |
| Simulate a widget change in a test | `session$setInputs(id = value)` |

### [06 · Performance &amp; Vectorization](level-3/06-performance-vectorization.md)

| Situation | Do this |
|---|---|
| Applying arithmetic/comparison across a vector | Use the operator directly (`x^2`, `x > 0`) — never loop |
| Loop's final output size is known ahead of time | Pre-allocate (`numeric(n)`, `vector("list", n)`) before the loop |
| Growing a result inside a loop with `c()` | Stop — this is quadratic; pre-allocate instead |
| Applying a function element-wise, type matters | `vapply(x, f, template)` over `sapply()` |
| Element-wise if/else across a vector | `ifelse(cond, yes, no)` |
| Logic depends on the previous iteration's result | A pre-allocated loop is fine — don't force a vectorized rewrite |

### [07 · Package Development Basics](level-3/07-package-development-basics.md)

| Task | Command |
|---|---|
| Generate NAMESPACE + docs from roxygen comments | `roxygen2::roxygenise(".")` |
| Bundle the package into a distributable file | `R CMD build <pkg-dir>` |
| Install the built package locally | `R CMD INSTALL <pkg>_<version>.tar.gz` |
| Run the full CRAN-style validation suite | `R CMD check <pkg>_<version>.tar.gz` |
| Make a function visible outside the package | `@export` roxygen tag |
| Run the package's test suite | `testthat::test_dir("tests/testthat")` or via `R CMD check` |
| View a function's generated help | `?function_name` (after install) |

### [08 · Calling APIs with httr](level-3/08-apis-httr.md)

| Task | Function |
|---|---|
| GET request | `GET(url)` |
| POST request with JSON body | `POST(url, body = list(...), encode = "json")` |
| Add query string parameters | `GET(url, query = list(...))` |
| Add custom headers (auth, content-type) | `add_headers(...)` |
| Read HTTP status code | `status_code(resp)` |
| Parse JSON response body | `content(resp, as = "parsed", type = "application/json")` |
| Raise an R error on non-2xx status | `stop_for_status(resp)` |
| Warn (don't stop) on non-2xx status | `warn_for_status(resp)` |
| Keep a secret out of source code | `Sys.getenv("VAR_NAME")` |

### [09 · Advanced Testing with testthat](level-3/09-testing-advanced.md)

| Task | Tool |
|---|---|
| Replace a function with a fake return value inside the function under test | `mockery::stub(fn, "dependency", mock(value))` |
| Assert a mock was called a specific number of times | `expect_called(m, n)` |
| Assert what arguments a mock was called with | `expect_args(m, call_number, ...)` |
| Create a temp file that auto-cleans after the test | `withr::local_tempfile(lines = ...)` |
| Create a temp dir that auto-cleans after the test | `withr::local_tempdir()` |
| Temporarily set an env var for one test | `withr::local_envvar(VAR = "value")` |
| Mark a test as intentionally not run, with a reason | `skip("reason")` |
| Skip only when a package isn't installed | `skip_if_not_installed("pkgname")` |
| Skip only in CRAN's automated checks | `skip_on_cran()` |

## Level 4 · Master

### [01 · Advanced Statistical Methods](level-4/01-advanced-statistical-methods.md)

| Task | Function |
|---|---|
| Compare 3+ group means at once | `aov(y ~ group, data = df)` |
| Post-hoc pairwise comparisons | `TukeyHSD(fit)` |
| Reduce correlated columns to components | `prcomp(df, scale. = TRUE)` |
| See variance explained per component | `summary(pca)` |
| See how columns load onto components | `pca$rotation` |
| Cluster rows with no labels | `kmeans(df, centers = k, nstart = 25)` |
| Wrap a vector with time metadata | `ts(x, frequency = n)` |
| Test equal variance before ANOVA | `bartlett.test(y ~ group, data = df)` |

### [02 · Production R with plumber](level-4/02-production-plumber.md)

| Task | Annotation / call |
|---|---|
| Define a GET route | `#* @get /path` |
| Define a POST route | `#* @post /path` |
| Access raw request body | function arg `req`, then `jsonlite::fromJSON(req$postBody)` |
| Access response object to set status | function arg `res` |
| Load a plumber file without running it | `plumb("file.R")` |
| Start the server | `pr$run(port = 8199, host = "127.0.0.1")` |
| Return a non-200 status | `res$status <- 400` inside the handler |
| Catch errors inside a route | wrap handler body in `tryCatch()` |

### [03 · Shiny at Scale (Modules)](level-4/03-shiny-at-scale.md)

| Task | Function |
|---|---|
| Define a module's UI half | `function(id) { ns <- NS(id); tagList(...) }` |
| Namespace an ID inside the UI half | `ns("some_id")` |
| Define a module's server half | `moduleServer(id, function(input, output, session) {...})` |
| Instantiate a module in an app | `counterUI("counterA")` / `counterServer("counterA")` |
| Expose a value to the parent | `return()` (or last expression) a reactive from the server function |
| Test a module without a browser | `testServer(counterServer, args = list(...), { session$setInputs(...); expect_equal(...) })` |
| Simulate a user input in a test | `session$setInputs(name = value)` |

### [04 · Testing at Scale &amp; CI](level-4/04-testing-at-scale-ci.md)

| Task | Call |
|---|---|
| Define one test unit | `test_that("description", { ... })` |
| Run one test file | `test_file("path/test-x.R")` |
| Run all tests in a package | `devtools::test()` |
| Run all tests in a directory, failing CI on error | `testthat::test_dir("tests/testthat", stop_on_failure = TRUE)` |
| Check coverage | `covr::package_coverage()` |
| View coverage in browser | `covr::report()` |
| Assert equality with tolerance | `expect_equal(a, b)` |
| Assert exact equality | `expect_identical(a, b)` |
| Assert an error is thrown | `expect_error(expr, "pattern")` |

### [05 · Performance Optimization (Rcpp Intro)](level-4/05-performance-rcpp.md)

| Task | Call |
|---|---|
| Compile an inline C++ function | `Rcpp::cppFunction('...')` |
| Compile a standalone `.cpp` file | `Rcpp::sourceCpp("file.cpp")` |
| Mark a C++ function as callable from R | `// [[Rcpp::export]]` above it |
| R numeric vector type in C++ | `NumericVector` |
| R integer vector type in C++ | `IntegerVector` |
| R character vector type in C++ | `CharacterVector` |
| Vector length in C++ | `x.size()` |
| Benchmark alternatives | `microbenchmark::microbenchmark(a = ..., b = ..., times = n)` |

### [06 · Reproducible Research Practices](level-4/06-reproducible-research.md)

| Task | Call |
|---|---|
| Create a project-local library | `renv::init()` |
| Create one without auto-installing existing deps | `renv::init(bare = TRUE)` |
| Record exact package versions | `renv::snapshot()` |
| Check for drift between code and lockfile | `renv::status()` |
| Reinstall exactly what the lockfile records | `renv::restore()` |
| Install a package into the project library | `renv::install("pkgname")` |
| See where a package is used in the project | `renv::dependencies()` |

### [07 · Data Visualization at Scale](level-4/07-data-viz-at-scale.md)

| Task | Call |
|---|---|
| Bin points into hexagons by density | `geom_hex(bins = n)` |
| Bin points into squares by density | `geom_bin2d()` |
| Random sample of rows for quick iteration | `df[sample(nrow(df), k), ]` |
| Save a plot with explicit dimensions | `ggsave("file.png", p, width = w, height = h, dpi = d)` |
| Reusable custom theme | `theme_x <- function() theme_minimal() + theme(...)` |
| One plot per group, shared scales | `facet_wrap(~group)` |
| Time how long a plot takes to render/save | `system.time({ ... })` |

### [08 · Working with Big Data in R (data.table)](level-4/08-big-data-datatable.md)

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

### [09 · Package Publishing (CRAN Basics)](level-4/09-package-publishing-cran.md)

| Task | Command |
|---|---|
| Generate docs/NAMESPACE from roxygen comments | `roxygen2::roxygenise()` |
| Build the source tarball | `R CMD build mypkg` |
| Run the full CRAN-style check | `R CMD check --as-cran mypkg_x.y.z.tar.gz` |
| Install a local package for testing | `R CMD INSTALL mypkg` |
| Run just the test suite | `R CMD check` (included) or `testthat::test_dir()` directly |
| Declare a runtime dependency | `Imports:` in `DESCRIPTION` |
| Declare a test/dev-only dependency | `Suggests:` in `DESCRIPTION` |

---

Go deeper with the full [R Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
