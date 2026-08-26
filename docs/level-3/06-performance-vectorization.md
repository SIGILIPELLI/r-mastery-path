# 06 · Performance & Vectorization

R's most common performance mistake isn't a slow algorithm — it's writing
loops in a language whose operators are already built to work on entire
vectors at once. This module measures the actual cost of that mistake,
covers the specific `c()`-in-a-loop pattern that makes it worse, and shows
`vapply()`'s type-safety advantage over `sapply()`.

## Loop vs. vectorized: a timed comparison

```r
n <- 200000
x <- rnorm(n)

loop_sq <- function(x) {
  out <- numeric(length(x))
  for (i in seq_along(x)) out[i] <- x[i]^2
  out
}
vec_sq <- function(x) x^2

system.time(loop_sq(x))
#    user  system elapsed
#   0.018   0.001   0.019
system.time(vec_sq(x))
#    user  system elapsed
#       0       0       0
```

Both functions compute the same 200,000 squared values and produce
identical results — but `x^2` dispatches to a single vectorized C
routine that operates on the whole array in one call, while the loop pays
R's per-iteration interpreter overhead 200,000 times over. The gap grows
with vector size; at a few hundred elements it's unnoticeable, at
millions it's the difference between milliseconds and minutes.
`x^2`, like nearly all of R's arithmetic and comparison operators, is
**vectorized by design** — reach for the operator directly before
reaching for a loop.

## The pre-allocation trap

A loop is sometimes genuinely necessary — but even then, how you grow the
result matters enormously:

```r
grow_bad <- function(n) {
  out <- c()
  for (i in 1:n) out <- c(out, i^2)   # reallocates + copies every iteration
  out
}
grow_good <- function(n) {
  out <- numeric(n)                   # allocated once, up front
  for (i in 1:n) out[i] <- i^2
  out
}

system.time(grow_bad(5000))
#    user  system elapsed
#   0.021   0.008   0.029
system.time(grow_good(5000))
#    user  system elapsed
#   0.001   0.000   0.001
```

**Trap:** `out <- c(out, i^2)` inside a loop looks harmless but forces R
to allocate a brand-new, one-element-larger vector and copy every
existing element into it on *every single iteration* — an operation whose
total cost grows quadratically with `n`. At `n = 5000` the difference is
already roughly 20x; at `n = 100000` the "growing" version can take
minutes while the pre-allocated version stays near-instant. Whenever you
know (or can bound) the final size of a loop's output ahead of time,
allocate it with `numeric(n)`, `character(n)`, or `vector("list", n)`
before the loop starts, and assign into it by index.

## sapply() vs vapply(): type safety

```r
sapply(1:5, function(i) if (i == 3) "oops" else i)
# [1] "1"    "2"    "oops" "4"    "5"
```

**Trap:** `sapply()` picks the result type by inspecting what actually
came back, silently coercing an entire numeric result to character the
moment a single element doesn't match — here every number got quietly
turned into text because of one string among five numbers, with no
warning at all. `vapply()` requires you to declare the expected return
type up front and raises an error the moment any element doesn't match
it:

```r
vapply(1:5, function(i) if (i == 3) "oops" else i, numeric(1))
# Error in vapply(...):
#   values must be type 'double', but FUN(X[[3]]) result is type 'character'
```

Prefer `vapply()` over `sapply()` in any code you're not immediately
eyeballing the output of — the type declaration (`numeric(1)`,
`character(1)`, `logical(1)`) both documents the expected shape and turns
a silent data-corruption bug into a loud, immediate error.

## Vectorized conditionals with ifelse()

```r
y <- c(-2, 3, -1, 5)
ifelse(y > 0, "pos", "neg")
# [1] "neg" "pos" "neg" "pos"
```

`ifelse()` is the vectorized equivalent of writing an `if/else` inside a
loop over every element — it evaluates the condition, "yes" value, and
"no" value all as full vectors and picks element-wise, avoiding a loop
entirely for simple per-element branching.

## When a loop actually is the right tool

Not everything vectorizes cleanly — logic that depends on a *previous
iteration's result* (a running total with a reset condition, a simulation
where each step depends on the last) often can't be rewritten as a single
vectorized expression. In those cases, pre-allocating the output (as
above) and accepting the loop is the correct choice — don't contort
otherwise-sequential logic into a vectorized form that becomes harder to
read for a performance gain that may not even materialize.

## Cheat sheet

| Situation | Do this |
|---|---|
| Applying arithmetic/comparison across a vector | Use the operator directly (`x^2`, `x > 0`) — never loop |
| Loop's final output size is known ahead of time | Pre-allocate (`numeric(n)`, `vector("list", n)`) before the loop |
| Growing a result inside a loop with `c()` | Stop — this is quadratic; pre-allocate instead |
| Applying a function element-wise, type matters | `vapply(x, f, template)` over `sapply()` |
| Element-wise if/else across a vector | `ifelse(cond, yes, no)` |
| Logic depends on the previous iteration's result | A pre-allocated loop is fine — don't force a vectorized rewrite |

## Exercise

1. Write a loop-based and a vectorized version of a function that
   computes `sqrt(abs(x))` for a numeric vector, time both with
   `system.time()` on a vector of 500,000 elements, and record the ratio.
2. Take a `for` loop that builds up a result with `out <- c(out, ...)`
   and rewrite it to pre-allocate with `numeric()`/`vector("list", ...)`
   instead — time both versions on `n = 10000`.
3. Replace an `sapply()` call in one of your own scripts (or one from an
   earlier module) with the equivalent `vapply()`, and note what template
   type you had to declare.
