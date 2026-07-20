# 09 · Packages

## What a package is

An R **package** bundles functions, data, and documentation for reuse — the
same idea as a Python package or an npm module. Base R ships with a useful
core (`stats`, `utils`, `graphics`, etc., all loaded automatically), and
**CRAN** (the Comprehensive R Archive Network) hosts tens of thousands more
that you install as needed.

## Installing a package

```r
install.packages("dplyr")
```

This downloads and installs the package (and its dependencies) from CRAN.
It's a one-time step per machine — you don't re-run it every time you use the
package, only when installing for the first time or upgrading.

## Loading a package with `library()`

Installing a package makes it available on disk; `library()` actually loads
it into your current R session so its functions are usable:

```r
library(dplyr)

# Now dplyr's functions are available directly:
starwars |> filter(species == "Human") |> nrow()
```

A common beginner mistake is calling a package's function without
`library()` first and getting `could not find function "..."` — the fix is
almost always a missing `library()` call at the top of the script.

## Checking what's installed

```r
installed.packages()[, "Package"]   # character vector of all installed package names

"dplyr" %in% installed.packages()[, "Package"]
# [1] TRUE
```

A common defensive pattern, especially in scripts shared with others:

```r
if (!require("dplyr", quietly = TRUE)) {
    install.packages("dplyr")
    library(dplyr)
}
```

`require()` behaves like `library()` but returns `FALSE` instead of erroring
if the package isn't installed, which is what makes this "install if missing"
pattern work.

## The `::` operator — calling a function without loading the package

```r
dplyr::filter(starwars, species == "Human")
```

`package::function()` calls a function directly from a package without a
`library()` call first. This is useful for a one-off call, for avoiding name
collisions between two loaded packages that both define a function with the
same name, and for making it obvious in code review exactly where a function
comes from.

## Package name collisions

If two loaded packages export a function with the same name, the
**more recently loaded** one wins by default, and R prints a warning when
this happens:

```r
library(dplyr)
library(MASS)     # MASS also has a function called "select"

# Attaching package: 'MASS'
# The following object is masked from 'package:dplyr':
#     select

select(mtcars, mpg)   # now calls MASS::select, not dplyr::select -- surprising!
dplyr::select(mtcars, mpg)  # unambiguous -- always calls the intended one
```

Reading that "masked" warning when it appears (rather than ignoring it) saves
real debugging time later.

## CRAN Task Views and popular packages

CRAN organizes packages into **Task Views** — curated lists for a domain
(e.g. the "TimeSeries" or "MachineLearning" task views). A few packages
you'll meet throughout this curriculum:

| Package | Purpose |
|---------|---------|
| `dplyr` | Data manipulation (filter, select, mutate, summarize) |
| `ggplot2` | Declarative, layered data visualization |
| `readr` | Fast, predictable file reading |
| `stringr` | Consistent string manipulation |
| `lubridate` | Date/time handling |
| `tidyr` | Reshaping data (pivoting, splitting columns) |
| `testthat` | Unit testing |
| `shiny` | Interactive web apps |

Installing all of them individually is tedious; the **tidyverse** meta-package
installs and loads the most common ones together:

```r
install.packages("tidyverse")
library(tidyverse)   # loads dplyr, ggplot2, readr, tidyr, stringr, and more at once
```

## Updating and removing packages

```r
update.packages()                 # check for and install updates to everything
remove.packages("somepackage")    # uninstall a package
```

## Packages cheat sheet

| Task | Command |
|------|---------|
| Install | `install.packages("pkgname")` |
| Load | `library(pkgname)` |
| Load or error clearly if missing | `require(pkgname)` |
| Call without loading | `pkgname::function_name()` |
| List installed | `installed.packages()[, "Package"]` |
| Update all | `update.packages()` |
| Remove | `remove.packages("pkgname")` |

## Exercise

Write a script that checks whether the `readr` package is installed using
`require()`, installs it if missing, then loads it and uses `readr::read_csv()`
(with the `::` syntax, even though it's already loaded, just to practice the
syntax) to read any CSV file from earlier modules. Print `sessionInfo()` at
the end to see the full list of currently loaded packages.
