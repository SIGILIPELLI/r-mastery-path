# 04 · Testing at Scale & CI

A handful of `stopifnot()` calls in a script is fine for a one-off
analysis. A codebase with dozens of functions that other people (or your
future self) depend on needs a real test suite: one command that runs
every check, reports every failure at once, and can be wired into CI so
a broken function never reaches `main`.

```r
library(testthat)
```

## Project layout `testthat` expects

```
myproject/
  R/
    add.R
  tests/
    testthat.R
    testthat/
      test-add.R
```

`R/add.R` holds the functions under test:

```r
add <- function(x, y) x + y

safe_divide <- function(x, y) {
  if (y == 0) stop("division by zero")
  x / y
}
```

`tests/testthat/test-add.R` holds the tests. Each `test_that()` block is
one named unit — a description string plus one or more `expect_*()`
assertions:

```r
library(testthat)
source("../../R/add.R")

test_that("add sums two numbers", {
  expect_equal(add(2, 3), 5)
  expect_equal(add(-1, 1), 0)
})

test_that("safe_divide errors on zero", {
  expect_error(safe_divide(1, 0), "division by zero")
  expect_equal(safe_divide(10, 2), 5)
})

test_that("add is vectorized", {
  expect_equal(add(c(1, 2, 3), c(1, 1, 1)), c(2, 3, 4))
})
```

## Running the suite

Inside an actual package, `devtools::test()` discovers and runs
everything under `tests/testthat/`. Standalone, `test_file()` runs one
file directly:

```r
test_file("test-add.R")
```

```
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 5 ]
```

Five passes: two `expect_equal()` calls in the first block, two
assertions in the second, one in the third. `testthat` counts individual
`expect_*()` calls, not `test_that()` blocks.

A genuine failure looks like this — deliberately asserting `add(2, 2)`
equals `5`:

```r
test_that("intentional failure demo", {
  expect_equal(add(2, 2), 5)
})
```

```
FAILURE: 'test-fail-demo.R:4:3' -------------------
Expected `add(2, 2)` to equal 5.
Differences:
1/1 mismatches
[1] 4 - 5 == -1

[ FAIL 1 | WARN 0 | SKIP 0 | PASS 0 ]
```

`testthat` shows the exact expression, the expected value, and the
numeric difference — no need to add your own `print()` calls to see what
went wrong.

## Common expectations

| Assertion | Checks |
|---|---|
| `expect_equal(a, b)` | numeric/object equality with floating-point tolerance |
| `expect_identical(a, b)` | exact equality, including type |
| `expect_error(expr, "pattern")` | `expr` throws an error matching `pattern` |
| `expect_warning(expr)` | `expr` emits a warning |
| `expect_true(x)` / `expect_false(x)` | a logical condition |
| `expect_length(x, n)` | `length(x) == n` |
| `expect_s3_class(x, "cls")` | `x` has the given S3 class |

## Measuring coverage with covr

`covr::package_coverage()` runs the test suite and reports which lines
of `R/` were actually exercised — useful for finding the branches your
tests never touch (like the `y == 0` guard if every test happened to use
a nonzero divisor):

```r
covr::package_coverage()
covr::report()  # opens an interactive HTML coverage report
```

`report()` highlights untested lines directly in the source, which is
the fastest way to spot a function that's only tested on its happy path.

## Wiring it into CI

A minimal GitHub Actions workflow that runs the suite on every push,
using `r-lib`'s maintained setup actions:

```yaml
# .github/workflows/R-CMD-check.yml
name: R tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: r-lib/actions/setup-r@v2
      - uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::testthat, any::covr
      - name: Run tests
        run: Rscript -e 'testthat::test_dir("tests/testthat", stop_on_failure = TRUE)'
```

`stop_on_failure = TRUE` is what makes this useful as a gate — without
it, `test_dir()` prints failures but still exits successfully, and a
red test suite would merge silently.

## R-specific traps

**`expect_equal()` uses a numeric tolerance by default** (`1.5e-8`), so
`expect_equal(0.1 + 0.2, 0.3)` passes even though the raw doubles differ
in the last bit. Reach for `expect_identical()` when you actually need
exact equality — for example, comparing integer vs. double `1L` vs `1`,
which `expect_equal()` treats as equal but `expect_identical()` does not.

**Tests must `source()` or load the code under test explicitly** —
`testthat` does not know your `R/` directory exists unless you're inside
a real package (where `devtools::test()` loads it automatically via
`devtools::load_all()`). A standalone script with a wrong relative
`source()` path fails with a confusing "could not find function" instead
of a clear "file not found."

**`stop_on_failure = TRUE` is what actually fails a CI run** — a
plain `Rscript -e 'testthat::test_dir(...)'` without it will happily
report `FAIL 3` in the log and still return exit code 0, and GitHub
Actions will mark the job green.

## Cheat sheet

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

## Exercise

1. Write a function `is_palindrome(x)` that checks whether a string reads
   the same forwards and backwards (ignore case), plus a `test_that()`
   block with at least three cases: a true palindrome, a false one, and
   an empty string.
2. Add a test that deliberately fails, run it with `test_file()`, and
   read the diff `testthat` prints to confirm you understand exactly
   what it's telling you — then fix the test.
3. Write a GitHub Actions workflow (or extend the one above) that also
   runs `covr::package_coverage()` and fails the build if coverage drops
   below a threshold you choose.
