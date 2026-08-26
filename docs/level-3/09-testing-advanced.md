# 09 · Advanced Testing with testthat

Level 2 covered basic `testthat` expectations. Real code under test
usually touches something you don't want a test suite actually hitting —
a network call, a paid API, the filesystem — and needs to clean up after
itself regardless of whether the test passes or fails. This module covers
mocking with `mockery`, fixtures with `withr`, and `skip()` for tests that
can't always run.

```r
library(testthat)
library(mockery)
library(withr)
```

## The problem: code that calls something untestable

```r
fetch_price <- function(ticker) {
  # in real life: an HTTP call to a paid market-data API
  stop("network not available in tests")
}

get_discounted_price <- function(ticker, discount) {
  price <- fetch_price(ticker)
  price * (1 - discount)
}
```

Testing `get_discounted_price()` directly would require a real network
call to `fetch_price()` on every test run — slow, flaky, and potentially
costly. The logic actually worth testing (the discount math) doesn't
depend on *how* the price was fetched, only on *what* `fetch_price()`
returned.

## Mocking with mockery

```r
test_that("get_discounted_price applies discount to mocked fetch", {
  m <- mock(100)
  stub(get_discounted_price, "fetch_price", m)

  result <- get_discounted_price("AAPL", 0.1)

  expect_equal(result, 90)
  expect_called(m, 1)
  expect_args(m, 1, "AAPL")
})
```

```text
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 3 ]
```

`mock(100)` creates a fake function that returns `100` every time it's
called, regardless of its arguments. `stub(get_discounted_price,
"fetch_price", m)` temporarily replaces `fetch_price()` *as called from
inside* `get_discounted_price()` with that mock, for the duration of this
one test — the real `fetch_price()` (and its `stop()`) is never reached.
`expect_called(m, 1)` and `expect_args(m, 1, "AAPL")` then verify not just
that the discount math was right, but that `fetch_price()` was called
exactly once, with the ticker you expected — asserting on *how* a
function was used, not only what it returned.

## Fixtures with automatic cleanup

A common testing need is a piece of test data — a temp file, a temp
directory, an environment variable — that must exist for the test and be
cleaned up afterward, even if the test fails partway through:

```r
test_that("uses a temp file fixture that cleans up automatically", {
  tf <- withr::local_tempfile(lines = c("a,b", "1,2"))
  df <- read.csv(tf)
  expect_equal(df$a, 1)
})
```

`withr::local_tempfile()` creates a temp file, writes the given lines to
it, and schedules its deletion for when the *current test* exits — pass
or fail, the file is gone before the next test runs. This is the general
`withr` pattern (`local_tempdir()`, `local_envvar()`, `local_dir()`, and
more): each `local_*()` function sets something up and registers its own
teardown, so you never write a manual cleanup step that a failed
assertion could skip over.

**Trap:** manually creating a temp file with `tempfile()` +
`file.create()` and cleaning it up with `unlink()` at the end of the test
body looks fine until an `expect_*()` earlier in the same test fails — a
failed expectation stops execution of that test block immediately, and a
manual cleanup line written *after* the assertions never runs, leaking
temp files across test runs. `withr`'s `local_*()` functions register
their cleanup to run regardless of how the test exits, which is why they
are the safer default over manual setup/teardown.

## Skipping tests that can't always run

```r
test_that("skipped example", {
  skip("demonstration of skip()")
  expect_true(FALSE)
})
```

```text
══ Skipped ═══════════════════════════════════════
1. skipped example - Reason: demonstration of skip()
```

`skip()` marks a test as intentionally not run, with a reason recorded in
the output — distinct from a passing or failing test. Real uses include
`skip_if_not_installed("some_optional_pkg")`, `skip_on_cran()` for tests
that need network access CRAN's check machinery won't have, or
`skip_if(Sys.getenv("API_KEY") == "")` for tests that need a real
credential not available in every environment. A skipped test is visible
in the report as skipped, which is the honest signal — silently deleting
a test that can't always run hides the fact that coverage has a gap.

## Cheat sheet

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

## Exercise

1. Take a function that calls another function you don't want to
   actually invoke in tests (a `send_email()`, a `charge_card()`, a
   `fetch_price()`-style call) and write a test using `mockery::stub()`
   and `mock()` that verifies the calling function's own logic without
   triggering the real dependency.
2. Write a test with a `withr::local_tempfile()` fixture, deliberately
   put a failing `expect_equal()` *before* the file would be used, and
   confirm (by checking `tempdir()` afterward) that the file was still
   cleaned up despite the failure.
3. Add a `skip_if_not_installed("mockery")` guard to a test file to see
   the message it produces, then remove `mockery` from consideration and
   re-run to see the test reported as skipped rather than failed.
