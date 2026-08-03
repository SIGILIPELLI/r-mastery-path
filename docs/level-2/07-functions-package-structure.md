# 07 · Writing Functions & Package Structure

[Level 1](../level-1/04-functions.md) covered the mechanics of writing a
function — parameters, defaults, return values, scope. This module goes one
level up: how to write functions that are safe to hand to someone else (or
your future self), document them properly, and organize a growing
collection of them the way real R projects do — as the beginning of a
**package**, R's standard unit of shareable, reusable code.

## Input validation — fail fast, fail clearly

A function that silently produces a wrong answer on bad input is far worse
than one that stops immediately with a clear message:

```r
bmi <- function(weight_kg, height_m) {
    if (!is.numeric(weight_kg) || !is.numeric(height_m)) {
        stop("weight_kg and height_m must both be numeric")
    }
    if (any(height_m <= 0)) {
        stop("height_m must be positive")
    }
    round(weight_kg / height_m^2, 1)
}

bmi(70, 1.75)
# [1] 22.9

bmi(70, 0)
# Error in bmi(70, 0) : height_m must be positive
```

`stop()` raises an error and halts execution immediately with a message —
the R equivalent of `raise`/`throw` in other languages. Compare this to
what happens *without* the check: `bmi(70, 0)` would silently return `Inf`
(division by zero), which then propagates through every later calculation
that uses it, and the *real* bug — a bad input three steps earlier — only
surfaces as a confusing `Inf` or `NaN` much later, far from its actual
cause.

`warning()` is the softer sibling — it lets execution continue but surfaces
a message the caller should read:

```r
safe_ratio <- function(numerator, denominator) {
    if (denominator == 0) {
        warning("denominator is 0 -- returning NA instead of dividing")
        return(NA)
    }
    numerator / denominator
}
```

## Documenting a function with roxygen2-style comments

Real R packages document functions with a specific comment convention
(`#'`) that the **roxygen2** package turns into proper help pages. Even
before you're building an installable package, writing comments in this
style is worth doing — it's a clear, standard structure for "what does this
function expect, and what does it give back":

```r
#' Calculate Body Mass Index
#'
#' @param weight_kg Numeric. Weight in kilograms.
#' @param height_m Numeric. Height in meters.
#' @return Numeric BMI, rounded to 1 decimal place.
#' @examples
#' bmi(70, 1.75)
bmi <- function(weight_kg, height_m) {
    round(weight_kg / height_m^2, 1)
}
```

`@param` documents each argument, `@return` documents the output, and
`@examples` gives runnable sample usage — the same three things any good
function documentation needs in any language, just with R's specific tag
syntax.

## Organizing functions into files

A small analysis project usually starts as one script, but once you have
more than a handful of related functions, splitting them into purpose-named
files and `source()`-ing them keeps things navigable:

```text
my_analysis/
    R/
        cleaning.R      # clean_names(), remove_outliers(), ...
        stats.R         # summarize_by_group(), ...
    analysis.R          # the main script that uses them
```

```r
# analysis.R
source("R/cleaning.R")
source("R/stats.R")

# now clean_names(), remove_outliers(), summarize_by_group() are all available
```

This `R/` directory naming is not a coincidence — it's exactly the
directory name a real R package expects for its function definitions, which
makes the next step (turning this into an actual package) mostly a matter
of adding metadata, not restructuring code.

## From a folder of scripts to a package skeleton

A minimal installable R package needs only a few pieces, all of which
`usethis::create_package()` generates for you:

```r
usethis::create_package("~/mypackage")
```

```text
mypackage/
    DESCRIPTION     # package name, version, author, dependencies
    NAMESPACE       # which functions are exported (usually auto-generated)
    R/              # your .R files with function definitions go here
```

`DESCRIPTION` is a plain-text metadata file:

```text
Package: mypackage
Title: What This Package Does
Version: 0.0.1
Author: Your Name
Description: A short description of the package's purpose.
License: MIT
```

Once functions in `R/` have roxygen2 comments, running
`devtools::document()` generates the `NAMESPACE` and help files
automatically, and `devtools::load_all()` loads the whole package for
interactive testing — the same workflow used to build every package on
CRAN, just scaled down.

## Why bother, for a personal analysis project?

Even if you never publish to CRAN, structuring code this way buys you:

- **`R CMD check`** — a built-in package validator that catches real bugs
  (undocumented functions, missing dependencies, examples that error) you'd
  otherwise only find by accident.
- **Easy reuse across projects** — `library(mypackage)` instead of copying
  `cleaning.R` into every new analysis folder and letting the copies drift
  apart.
- **Testability** — [Level 2, Module 8](08-testing-testthat.md) covers
  `testthat`, which expects the standard `R/` + `tests/` package layout.

## Namespacing and `::` inside your own functions

Once your functions live inside a package, calling another package's
function explicitly with `::` (introduced in
[Level 1, Module 9](../level-1/09-packages.md)) becomes more than a style
preference — it's how a package declares exactly what it depends on, which
`R CMD check` verifies against the `DESCRIPTION` file's declared
dependencies:

```r
#' @importFrom dplyr filter
clean_active_users <- function(df) {
    dplyr::filter(df, active == TRUE)
}
```

## Function-writing cheat sheet

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

## Exercise

Take the `classify_bmi()` function from
[Level 1, Module 4's exercise](../level-1/04-functions.md) and rewrite it
with: (1) a `stop()` call if the input isn't numeric, (2) a roxygen2-style
comment block with `@param`, `@return`, and `@examples`, and (3) save it
into its own file, `R/bmi.R`, alongside a `bmi()` function in the same
file. Then write a two-line `analysis.R` script that `source()`s
`R/bmi.R` and calls both functions.
