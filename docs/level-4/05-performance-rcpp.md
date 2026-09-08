# 05 · Performance Optimization (Rcpp Intro)

R's vectorized functions are already fast — `sum()`, `colSums()`,
matrix multiplication all drop into compiled C under the hood. The
place R gets slow is the code you can't vectorize: an explicit loop
with a data-dependent, iteration-to-iteration state, like a running
simulation or a recursive update. `Rcpp` lets you write that one loop
in C++ and call it from R like any other function, with no separate
build step.

```r
library(Rcpp)
```

## Writing an inline C++ function

`cppFunction()` compiles a C++ snippet on the fly and binds it to an R
name:

```r
cppFunction('
double sum_cpp(NumericVector x) {
  double total = 0;
  int n = x.size();
  for (int i = 0; i < n; i++) {
    total += x[i];
  }
  return total;
}
')

x <- rnorm(1e6)
sum_cpp(x)
sum(x)
```

```
> sum_cpp(x)
[1] 313.242
> sum(x)
[1] 313.242
```

`NumericVector` is Rcpp's C++ wrapper around R's numeric vector type —
it behaves like a `std::vector<double>` with `[]` indexing and `.size()`,
and Rcpp handles converting the R object in and the `double` return
value back out automatically.

## Why the loop is the bottleneck

Compare three ways of summing the same vector: the C++ version above, a
plain R `for` loop, and R's built-in vectorized `sum()`:

```r
slow_sum_r <- function(x) {
  total <- 0
  for (i in seq_along(x)) total <- total + x[i]
  total
}

microbenchmark::microbenchmark(
  cpp          = sum_cpp(x),
  r_loop       = slow_sum_r(x),
  r_vectorized = sum(x),
  times = 20
)
```

```
Unit: microseconds
         expr       min        lq     mean    median        uq       max neval
          cpp   991.708   998.227  1088.15  1060.568  1111.408  1687.273    20
       r_loop 22521.751 22990.607 24134.46 23874.013 24334.668 32545.226    20
 r_vectorized  1647.626  1666.014  1771.28  1782.393  1820.933  2038.479    20
```

The C++ loop beats R's own `for` loop by roughly 20x, and even edges out
R's built-in vectorized `sum()` — R's `for` loop is slow because every
iteration re-dispatches through R's interpreter and re-checks types;
`sum()` avoids that by calling into compiled C internally, and the Rcpp
version does the same thing but for a loop shape `sum()` doesn't offer.

## A case that actually needs a loop: a running cumulative filter

Vectorized R can't express "each output depends on the previous output"
directly (`cumsum()` covers pure addition, but not a general recursive
update). An exponential moving average is a good example:

```r
cppFunction('
NumericVector ema_cpp(NumericVector x, double alpha) {
  int n = x.size();
  NumericVector out(n);
  out[0] = x[0];
  for (int i = 1; i < n; i++) {
    out[i] = alpha * x[i] + (1 - alpha) * out[i - 1];
  }
  return out;
}
')

ema_cpp(c(10, 12, 11, 15, 20), alpha = 0.5)
```

```
> ema_cpp(c(10, 12, 11, 15, 20), alpha = 0.5)
[1] 10.000 11.000 11.000 13.000 16.500
```

Each `out[i]` reads `out[i - 1]`, which is exactly the shape that
resists vectorization in R but costs nothing extra in a compiled loop.

## Standalone `.cpp` files with `sourceCpp()`

For anything longer than a few lines, put the code in its own file with
an `// [[Rcpp::export]]` tag above each function you want callable from
R, and load it with `sourceCpp()`:

```cpp
// ema.cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
NumericVector ema_cpp(NumericVector x, double alpha) {
  int n = x.size();
  NumericVector out(n);
  out[0] = x[0];
  for (int i = 1; i < n; i++) {
    out[i] = alpha * x[i] + (1 - alpha) * out[i - 1];
  }
  return out;
}
```

```r
sourceCpp("ema.cpp")
ema_cpp(c(10, 12, 11, 15, 20), alpha = 0.5)
```

`sourceCpp()` recompiles only when the file changes, so iterating on a
`.cpp` file is nearly as fast as editing an R script once the first
compile is done.

## R-specific traps

**The first call to `cppFunction()` or `sourceCpp()` in a session pays a
real compilation cost** (often a second or more) — don't benchmark that
first call, and don't be alarmed that "Rcpp is slow" the very first time
you run it; every call after that hits the already-compiled shared
object.

**Rcpp vector types are 0-indexed, R vectors are 1-indexed.** `x[0]` in
C++ is `x[1]` in R — this is the single most common source of an
off-by-one bug when porting an R loop to Rcpp, especially when a loop
bound copied straight from R (`for (i in 1:n)`) becomes `for (int i = 1;
i <= n; i++)` in C++ and silently reads one element past the vector.

**A C++ compiler toolchain must be installed and on `PATH`** (Xcode
command line tools on macOS, `r-base-dev` on Debian/Ubuntu, Rtools on
Windows) — without it, `cppFunction()` fails with a linker or "cannot
find gcc/clang" error that has nothing to do with your C++ syntax being
wrong.

## Cheat sheet

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

## How It Actually Works

`Rcpp` lets you write C++ functions callable from R by generating **glue
code**: `cppFunction()`/`sourceCpp()` parse your C++ source, detect the
`// [[Rcpp::export]]` marker, and generate a wrapper that converts R SEXPs
(the same underlying representation from Module 2) into C++ types like
`Rcpp::NumericVector` — which is really a thin C++ class wrapping a
pointer directly into R's own memory, not a copy — compiles everything
with your system's C++ compiler, and dynamically loads the resulting
shared library into the running R session so the exported function becomes
callable like any other R function.

The performance win is mechanical: your loop now runs as genuinely
compiled machine code with no R-evaluator dispatch per iteration at all
(unlike even a vectorized R call, which still pays one dispatch to enter
the C routine) — for tight numeric loops, that difference compounds
across millions of iterations. But because `NumericVector` often wraps R's
memory directly rather than copying it, mutating it in place inside C++
can violate R's copy-on-modify guarantees if you're not careful — which is
why Rcpp idioms favor `clone()`ing input vectors you intend to mutate,
mirroring the exact copy-on-modify discipline R itself enforces at the
R level.

## Exercise

1. Write an Rcpp function `running_max_cpp(x)` that returns a vector
   where each element is the maximum of `x` up to and including that
   position (the "running max"), and confirm it matches
   `cummax(x)` on a random vector.
2. Benchmark your `running_max_cpp()` against an equivalent R `for` loop
   and against `cummax()` with `microbenchmark::microbenchmark()`, and
   note which is fastest and by roughly how much.
3. Move your `running_max_cpp()` into a standalone `.cpp` file with
   `// [[Rcpp::export]]`, load it with `sourceCpp()`, and confirm it
   still gives the same answer as the inline version.
