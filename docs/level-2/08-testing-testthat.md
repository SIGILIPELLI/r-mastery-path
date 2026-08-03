# 08 · Testing with testthat

Manually re-running a function on a couple of examples every time you
change it doesn't scale — and it's easy to forget an edge case you tested
once but never wrote down. **testthat** is R's standard unit-testing
package: you write expectations once, and re-run the whole suite in
seconds any time you change code, immediately seeing exactly which
behavior broke.

```r
library(testthat)
```

## Anatomy of a test file

Real projects put tests in `tests/testthat/test-<name>.R`, mirroring the
function files in `R/`. A single test file typically groups related
expectations inside `test_that()` blocks:

```r
# tests/testthat/test-bmi.R

bmi <- function(weight_kg, height_m) {
    round(weight_kg / height_m^2, 1)
}

test_that("bmi computes the standard formula correctly", {
    expect_equal(bmi(70, 1.75), 22.9)
    expect_equal(bmi(50, 1.6), 19.5)
})
```

`test_that("description", { ... })` groups one or more related checks
under a human-readable description — when a test fails, that description
is exactly what shows up in the output, so write it as a sentence
describing the *behavior* being verified, not the function name.

## Core expectations

```r
test_that("basic expectations work", {
    expect_equal(2 + 2, 4)               # equal (allows small floating-point tolerance)
    expect_identical(2L, 2L)             # exactly identical, including type
    expect_true(5 > 3)
    expect_false(5 < 3)
    expect_null(NULL)
    expect_error(stop("boom"))           # confirms an error IS raised
    expect_warning(warning("careful"))   # confirms a warning IS raised
    expect_length(c(1, 2, 3), 3)
    expect_type(5L, "integer")
})
```

| Expectation | Checks |
|-------------|--------|
| `expect_equal(a, b)` | numerically/structurally equal (small floating-point differences OK) |
| `expect_identical(a, b)` | *exactly* identical, including type (`2L` vs `2` differ) |
| `expect_true(x)` / `expect_false(x)` | a single logical value |
| `expect_error(expr)` | `expr` raises an error |
| `expect_warning(expr)` | `expr` raises a warning |
| `expect_length(x, n)` | `length(x) == n` |
| `expect_type(x, "type")` | `typeof(x) == "type"` |

**Trap: `expect_equal()` vs. `expect_identical()`.** `expect_equal(1L, 1)`
passes (an integer and a double representing the same number are
numerically equal), but `expect_identical(1L, 1)` fails (they're different
underlying types). Use `expect_equal()` for almost everything involving
numbers — floating-point arithmetic rarely produces bit-for-bit identical
results even when mathematically correct — and reserve `expect_identical()`
for when the exact type genuinely matters.

## Testing edge cases, not just the happy path

The real value of a test suite is catching the inputs you'd forget to
re-check by hand every time:

```r
classify_bmi <- function(bmi_value) {
    if (!is.numeric(bmi_value)) stop("bmi_value must be numeric")
    if (bmi_value < 18.5) "Underweight"
    else if (bmi_value < 25) "Normal"
    else if (bmi_value < 30) "Overweight"
    else "Obese"
}

test_that("classify_bmi handles boundary values correctly", {
    expect_equal(classify_bmi(18.4), "Underweight")
    expect_equal(classify_bmi(18.5), "Normal")     # exactly on the boundary
    expect_equal(classify_bmi(24.9), "Normal")
    expect_equal(classify_bmi(25.0), "Overweight") # exactly on the boundary
    expect_equal(classify_bmi(30.0), "Obese")
})

test_that("classify_bmi rejects invalid input", {
    expect_error(classify_bmi("not a number"))
    expect_error(classify_bmi(NA))
})
```

Boundary values (exactly 18.5, exactly 25, exactly 30) are exactly where
off-by-one mistakes hide — an `if (bmi_value <= 18.5)` typo instead of `<`
would pass any test that only checked "clearly underweight" and "clearly
normal" values, but fails one that checks the boundary itself.

## Testing for `NA` and missing-data behavior

Since [Module 2](02-data-cleaning.md) covered how easily `NA` propagates
silently through calculations, it's worth testing that a function handles
missing data the way you intend, rather than discovering it in production:

```r
safe_mean <- function(x) {
    if (all(is.na(x))) {
        return(NA)
    }
    mean(x, na.rm = TRUE)
}

test_that("safe_mean handles missing values as intended", {
    expect_equal(safe_mean(c(1, 2, NA, 4)), 7 / 3)
    expect_true(is.na(safe_mean(c(NA, NA, NA))))
    expect_equal(safe_mean(c(1, 2, 3)), 2)
})
```

## Running tests

Inside an actual package (with the standard `tests/testthat/` layout):

```r
devtools::test()          # run the whole suite, from an R session
```

```bash
Rscript -e 'testthat::test_dir("tests/testthat")'   # from the command line
```

```text
✔ | F W S  OK | Context
✔ |         5 | bmi
✔ |         3 | classify_bmi [0.1s]

══ Results ═══════════════════════════════════
[ FAIL 0 | WARN 0 SKIP 0 PASS 8 ]
```

A clean run shows `FAIL 0` — the moment any expectation fails, testthat
prints exactly which `test_that()` block, which `expect_*()` call, and
what was expected vs. what was actually returned, pointing straight at the
regression instead of leaving you to bisect manually.

## Why write tests before "finishing" a function

A function without tests can look correct after a few manual checks and
still hide a boundary bug, an `NA`-handling gap, or a case broken by a
"harmless" refactor two weeks later. A test suite turns "does this still
work?" from a manual re-check into a single command, and turns "I broke
something" from a mystery into an exact line number.

## testthat cheat sheet

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

## Exercise

Write `test_that()` blocks for the `safe_ratio()` function from
[Module 7](07-functions-package-structure.md#input-validation-fail-fast-fail-clearly):
one test confirming the normal division case, one confirming it returns
`NA` (not an error) when the denominator is 0, and one confirming
`expect_warning()` catches the warning it raises in that zero case. Then
deliberately introduce a bug (e.g. change `<-` to use integer division) and
run your tests to confirm they actually catch it — a test suite that never
fails is a test suite you haven't verified is testing anything real.
