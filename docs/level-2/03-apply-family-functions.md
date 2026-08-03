# 03 · Apply Family Functions

R vectorizes most arithmetic automatically (`x + 1` adds 1 to every
element), but plenty of real tasks need to call a *function* once per
element or once per group — parsing a list of strings, running a model per
category, applying a custom rule to every column. The **apply family**
(`sapply`, `lapply`, `vapply`, `mapply`, plus base `apply` for matrices) is
R's idiomatic way to do this without writing an explicit `for` loop. They
aren't faster than a well-written loop in every case, but they're more
concise, harder to get subtly wrong, and instantly recognizable to other R
programmers as "map this function over this thing."

## Why not just use a `for` loop?

You can — R loops work fine. But an apply call reads as one clear
statement of intent ("run this function on every element, collect the
results") instead of manual setup:

```r
nums <- c(4, 9, 16, 25)

# Explicit loop -- works, but verbose and needs a pre-allocated result vector
result <- numeric(length(nums))
for (i in seq_along(nums)) {
    result[i] <- sqrt(nums[i])
}
result
# [1] 2 3 4 5

# Same thing with sapply -- one line, same result
sapply(nums, sqrt)
# [1] 2 3 4 5
```

`seq_along(nums)` (not `1:length(nums)`) is the safe way to index a loop —
if `nums` were empty, `1:length(nums)` becomes `1:0`, which is `c(1, 0)`,
not an empty sequence, silently looping over indices that don't exist.
`seq_along()` correctly produces an empty sequence for an empty input.

## `sapply()` — simplify to a vector (or matrix) when possible

```r
words <- c("apple", "kiwi", "watermelon")

sapply(words, nchar)
#      apple       kiwi watermelon
#          5          4         10
```

`sapply()` tries to **simplify** its result into the friendliest shape — a
named vector here. That's convenient, but it's also `sapply()`'s biggest
trap:

**Trap:** the simplified shape depends on the *data*, not just the code, so
the same call can return a vector sometimes and a list or matrix other
times, which breaks downstream code that assumed one particular shape:

```r
sapply(1:3, function(x) x^2)          # simplifies to a vector
# [1] 1 4 9

sapply(1:3, function(x) 1:x)          # different length each time -- can't
# [[1]]                                # simplify to a vector, so you get a LIST
# [1] 1
# [[2]]
# [1] 1 2
# [[3]]
# [1] 1 2 3
```

If you need a guarantee about the output type, use `vapply()` instead.

## `lapply()` — always returns a list

```r
lapply(words, nchar)
# [[1]]
# [1] 5
# [[2]]
# [1] 4
# [[3]]
# [1] 10
```

`lapply()` is `sapply()` without the simplification step — it *always*
returns a list, one element per input, no exceptions. That predictability
makes it the safer choice inside a function or pipeline where surprising
type changes would cause hard-to-trace bugs. Use `unlist()` afterward if
you're confident the result really is simplifiable:

```r
unlist(lapply(words, nchar))
# [1]  5  4 10
```

## `vapply()` — like `sapply()`, but with a guaranteed return type

```r
vapply(words, nchar, FUN.VALUE = integer(1))
#      apple       kiwi watermelon
#          5          4         10
```

`FUN.VALUE` declares the *shape* of each individual result (here, one
integer). If the function ever returns something that doesn't match — the
wrong length or type — `vapply()` throws an immediate, clear error instead
of silently producing a differently-shaped result three function calls
later:

```r
vapply(1:3, function(x) 1:x, FUN.VALUE = integer(1))
# Error in vapply(1:3, function(x) 1:x, FUN.VALUE = integer(1)) :
#   values must be length 1,
#  but FUN(X[[2]]) result is length 2
```

This is a genuine advantage over `sapply()` for production code: fail loud
and immediately, at the exact call that produced a bad value, rather than
fail quietly and confusingly downstream.

## `mapply()` — apply over multiple vectors in parallel

```r
prices <- c(2.5, 4.0, 1.25)
quantities <- c(3, 2, 10)

mapply(function(p, q) p * q, prices, quantities)
# [1]  7.5  8.0 12.5
```

`mapply()` walks several vectors together, element by element — the
multi-argument counterpart to `sapply()`. Where `sapply(x, f)` calls
`f(x[1])`, `f(x[2])`, ..., `mapply(f, x, y)` calls `f(x[1], y[1])`,
`f(x[2], y[2])`, and so on.

## `apply()` — row-wise or column-wise over a matrix or data frame

```r
scores <- matrix(c(90, 85, 70, 95, 60, 100), nrow = 3, ncol = 2,
                  dimnames = list(c("Alice", "Bob", "Carol"), c("test1", "test2")))
scores
#       test1 test2
# Alice    90    95
# Bob      85    60
# Carol    70   100

apply(scores, 1, mean)   # MARGIN = 1: apply mean() to each ROW
#  Alice    Bob  Carol
#   92.5   72.5   85.0

apply(scores, 2, mean)   # MARGIN = 2: apply mean() to each COLUMN
#    test1    test2
# 81.66667 85.00000
```

`MARGIN = 1` means "collapse across columns, one result per row"; `MARGIN =
2` means "collapse across rows, one result per column" — a distinction
that's easy to mix up. A useful mnemonic: rows are the first dimension in
`dim()`/`matrix()`, so `MARGIN = 1` walks *down* the rows.

## Anonymous functions and the `\(x)` shorthand

Every example above used a named function (`sqrt`, `nchar`) or a full
`function(x) ...` definition. R 4.1+ supports a shorthand for quick,
throwaway functions:

```r
sapply(1:5, function(x) x^2)   # traditional
sapply(1:5, \(x) x^2)          # shorthand, R 4.1+ -- identical result
```

Both are common in real code; the shorthand is popular for short one-liners
passed directly into `sapply`/`lapply`/`mutate`, and the full `function`
keyword is often preferred for anything longer than one expression.

## Apply family cheat sheet

| Function | Input | Output |
|----------|-------|--------|
| `sapply(x, f)` | vector/list | simplified vector/matrix (or list if it can't simplify) |
| `lapply(x, f)` | vector/list | **always** a list |
| `vapply(x, f, FUN.VALUE)` | vector/list | vector, with a guaranteed, declared shape |
| `mapply(f, x, y)` | multiple vectors, walked together | simplified vector/matrix |
| `apply(m, MARGIN, f)` | matrix/data frame | vector/matrix, one result per row (`MARGIN=1`) or column (`MARGIN=2`) |

## When to reach for dplyr instead

For anything operating on a data frame's columns — summarizing a group,
adding a derived column — [`dplyr`](01-dplyr-deep-dive.md)'s
`mutate()`/`summarise()` is usually more readable than `apply()`/`sapply()`
directly on a data frame, and doesn't have `sapply()`'s shape-guessing
problem. The apply family shines on plain vectors, lists, and matrices, or
inside your own functions where dplyr's data-frame-shaped verbs don't fit.

## Exercise

Given `prices <- c(2.5, 4.0, 1.25, 9.99)`, write three versions of "round
every price to the nearest whole dollar": one with an explicit `for` loop,
one with `sapply()`, and one with `vapply()` declaring `FUN.VALUE =
numeric(1)`. Then take `words <- c("R", "python", "javascript")` and use
`vapply()` to build a logical vector marking which words have more than 4
characters — and explain in a comment why `vapply()` is a safer choice than
`sapply()` here even though both would give the same answer.
