# 09 · Package Publishing (CRAN Basics)

A package is what turns "a folder of R scripts I reuse" into something
installable with `install.packages()` and citable by version number.
CRAN adds one more layer on top: a strict, machine-checked set of rules
(`R CMD check --as-cran`) that every submission must pass, which is also
just a genuinely good discipline to run even for a package you never
submit.

## Minimum package structure

```
mypkg/
  DESCRIPTION
  NAMESPACE
  LICENSE
  R/
    add_one.R
  man/
    add_one.Rd
  tests/
    testthat.R
    testthat/
      test-add_one.R
```

`DESCRIPTION` is the package's metadata — name, version, author, license,
and dependencies:

```
Package: mypkg
Title: Example Package for CRAN Basics
Version: 0.1.0
Authors@R: person("First", "Last", email = "you@example.com", role = c("aut", "cre"))
Description: A minimal example package used to demonstrate CRAN packaging basics.
License: MIT + file LICENSE
Encoding: UTF-8
Suggests: testthat (>= 3.0.0)
Config/testthat/edition: 3
```

`NAMESPACE` declares what the package exposes:

```
export(add_one)
```

## Documenting a function with roxygen comments

Every exported function needs a help page. Writing the roxygen block
directly above the function (and running `roxygen2::roxygenise()` to
generate the matching `.Rd` file) keeps the docs next to the code
instead of maintained separately:

```r
#' Add one to a number
#'
#' @param x A numeric vector.
#' @return `x` with 1 added to each element.
#' @examples
#' add_one(41)
#' @export
add_one <- function(x) {
  x + 1
}
```

The `@examples` block isn't just documentation — `R CMD check` actually
*runs* every example during a check, so an example that errors fails
the check, which catches stale examples that drifted from the function's
real behavior.

## Building and checking the package

```bash
R CMD build mypkg
```

```
* checking for file 'mypkg/DESCRIPTION' ... OK
* preparing 'mypkg':
* checking DESCRIPTION meta-information ... OK
* checking for LF line-endings in source and make files and shell scripts
* checking for empty or unneeded directories
* building 'mypkg_0.1.0.tar.gz'
```

`R CMD build` produces the source tarball that `install.packages()` or
CRAN itself would consume. Then run the same check CRAN runs on every
submission:

```bash
R CMD check --as-cran mypkg_0.1.0.tar.gz
```

```
* checking whether the package can be unloaded cleanly ... OK
* checking dependencies in R code ... OK
* checking Rd files ... OK
* checking for missing documentation entries ... OK
* checking examples ... OK
* checking tests ...
  Running 'testthat.R'
 OK
* checking PDF version of manual ... WARNING
LaTeX errors when creating PDF version.
* checking PDF version of manual without index ... ERROR
Error in texi2dvi(file = file, pdf = TRUE, clean = clean, quiet = quiet, :
  pdflatex is not available
* checking HTML version of manual ... OK
* checking for non-standard things in the check directory ... NOTE
Found the following files/directories:
  'mypkg-manual.tex'
* checking for detritus in the temp directory ... OK
* DONE

Status: 1 ERROR, 1 WARNING, 2 NOTEs
```

Everything that matters to the package's actual correctness passed:
namespace, examples, and the `testthat` suite all report `OK`. The
`ERROR` here is environmental, not a package problem — this machine has
no `pdflatex` installed, so `R CMD check` can't render the PDF manual.
That's exactly the kind of check-log line worth learning to read
correctly: not every `ERROR`/`WARNING`/`NOTE` means your code is wrong,
but every one needs to be understood well enough to say so, since CRAN
reviewers will ask about anything left unexplained in a real submission.

## What CRAN specifically checks for

Beyond what `R CMD check` reports automatically, CRAN's human reviewers
and policies also require:

- `DESCRIPTION`'s `Version` field bumped on every resubmission.
- No writing to the user's home directory, working directory, or
  anywhere outside `tempdir()` from examples, tests, or vignettes.
- Every dependency in `Imports`/`Suggests` actually used, and every
  package actually used declared there — `R CMD check` catches most of
  this under "checking dependencies in R code."
- Examples and tests that run in a reasonable time (CRAN enforces a
  per-check time budget across all platforms it tests on).
- A `LICENSE` file whose text matches what `DESCRIPTION` declares.

## R-specific traps

**`NAMESPACE` and `.Rd` files are usually machine-generated, not
hand-written**, via `roxygen2::roxygenise()` reading the `#'` comments
above each function — editing `NAMESPACE` directly is fine for a small
manual example like this one, but on any real package a hand-edit gets
silently overwritten the next time `roxygenise()` runs.

**A package that works when loaded via `devtools::load_all()` can still
fail `R CMD check`** — `load_all()` is more forgiving about namespace
declarations and missing documentation than a real build/check cycle,
so "it works in my session" is not the same guarantee as "it passes
check."

**CRAN checks run on multiple platforms you may not have locally**
(Windows, several macOS/Linux flavors, sometimes multiple R versions) —
a clean `R CMD check --as-cran` on your machine is necessary but not
sufficient; a submission can still come back with a platform-specific
NOTE you have no way to reproduce without a service like R-hub.

## Cheat sheet

| Task | Command |
|---|---|
| Generate docs/NAMESPACE from roxygen comments | `roxygen2::roxygenise()` |
| Build the source tarball | `R CMD build mypkg` |
| Run the full CRAN-style check | `R CMD check --as-cran mypkg_x.y.z.tar.gz` |
| Install a local package for testing | `R CMD INSTALL mypkg` |
| Run just the test suite | `R CMD check` (included) or `testthat::test_dir()` directly |
| Declare a runtime dependency | `Imports:` in `DESCRIPTION` |
| Declare a test/dev-only dependency | `Suggests:` in `DESCRIPTION` |

## How It Actually Works

CRAN's submission process isn't a manual review of your code's logic — it
runs your package through **automated `R CMD check --as-cran`** on
multiple platforms (Windows, macOS, several Linux flavors, and both
release and development R versions) in CRAN's own infrastructure, checking
things a human reviewer wouldn't scale to: examples that must run within a
time budget, no writing outside `tempdir()` during checks, correct
`Encoding` declarations, and no undeclared dependencies — because your
package's `NAMESPACE` and `DESCRIPTION` are the *only* things CRAN's
automated tooling can statically verify without executing arbitrary
untrusted code from every submission.

Once accepted, CRAN mirrors your package's source tarball across its
global mirror network and rebuilds binary versions for Windows/macOS
using its own build farm — this is why a source-only submission can take
a day or two to appear as an installable binary on other platforms, and
why a package that only compiles on your machine (an undeclared system
library dependency, e.g.) fails silently for users elsewhere until CRAN's
build farm catches it and the maintainer is notified to fix it.

## Exercise

1. Add a second exported function to the example package (e.g.
   `subtract_one <- function(x) x - 1`) with its own roxygen block and
   `@examples`, then rebuild and re-check the package.
2. Deliberately introduce a documentation mismatch (change a function's
   argument name without updating its `@param` line) and run
   `R CMD check` to see exactly which check catches it.
3. Read through the `Writing R Extensions` CRAN policy page's summary of
   submission requirements and list three rules from it that aren't
   caught automatically by `R CMD check`.
