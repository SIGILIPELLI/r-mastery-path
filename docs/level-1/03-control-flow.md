# 03 · Control Flow

## `if` / `else if` / `else`

```r
score <- 78

if (score >= 90) {
    print("A")
} else if (score >= 80) {
    print("B")
} else if (score >= 70) {
    print("C")
} else {
    print("F")
}
# [1] "C"
```

`if` in R takes a single logical value (length 1). Passing a longer vector
triggers a warning (or an error, in recent R versions) — use `&&`/`||` inside
the condition, not `&`/`|`, and use `ifelse()` (below) when you need to branch
on every element of a vector at once.

## `ifelse()` — vectorized branching

Unlike `if`, which only evaluates one condition, `ifelse()` evaluates a
condition **element-wise** across an entire vector — essential for working
with data frame columns.

```r
scores <- c(95, 82, 61, 74)
grades <- ifelse(scores >= 70, "Pass", "Fail")
grades
# [1] "Pass" "Pass" "Fail" "Pass"
```

This single line replaces what would otherwise be a manual loop — a preview
of R's vectorized style, which you'll rely on constantly once you're working
with real datasets.

## `for` loops

```r
for (i in 1:5) {
    print(i)
}
# [1] 1
# [1] 2
# [1] 3
# [1] 4
# [1] 5
```

`for` in R always iterates over a vector (or list) of values, not a counter —
`1:5` just happens to produce the vector `c(1, 2, 3, 4, 5)`.

```r
fruits <- c("apple", "banana", "cherry")
for (fruit in fruits) {
    cat("I like", fruit, "\n")
}
# I like apple
# I like banana
# I like cherry
```

!!! warning "Loops are a last resort in R"
    R is optimized for **vectorized** operations, not explicit loops.
    `sum(scores)`, `scores * 2`, or `ifelse(...)` all operate on a whole
    vector at once, internally in fast C code — a `for` loop doing the same
    thing element-by-element is typically both slower and less idiomatic.
    Loops are still worth learning first because they make the underlying
    logic explicit; you'll see the vectorized/apply-family alternatives in
    [Level 2](../level-2/03-apply-family-functions.md).

## `while` loops

```r
count <- 1
while (count <= 3) {
    print(count)
    count <- count + 1
}
# [1] 1
# [1] 2
# [1] 3
```

## `repeat` with `break`

R also has `repeat`, an unconditional loop that only stops via `break`:

```r
count <- 1
repeat {
    print(count)
    count <- count + 1
    if (count > 3) {
        break
    }
}
# [1] 1
# [1] 2
# [1] 3
```

## `break` and `next`

`break` exits a loop immediately; `next` (R's equivalent of `continue`)
skips to the next iteration:

```r
for (i in 1:10) {
    if (i == 5) {
        break          # stop the loop entirely once i reaches 5
    }
    print(i)
}
# [1] 1 2 3 4

for (i in 1:6) {
    if (i %% 2 == 0) {
        next           # skip even numbers
    }
    print(i)
}
# [1] 1 3 5
```

## `switch()`

`switch()` matches a value against a set of named cases — useful when an
`if`/`else if` chain would otherwise test the same variable repeatedly:

```r
describe_day <- function(day) {
    switch(day,
        "Mon" = "Start of the work week",
        "Fri" = "Almost the weekend",
        "Sat" = ,
        "Sun" = "Weekend!",
        "Just a regular day"    # default/fallback case
    )
}

describe_day("Fri")
# [1] "Almost the weekend"
describe_day("Sat")
# [1] "Weekend!"
describe_day("Tue")
# [1] "Just a regular day"
```

Leaving a case's right-hand side empty (like `"Sat" = ,`) falls through to the
next case's value — a concise way to group several inputs under one result.

## Loop vs. vectorized cheat sheet

| Task | Loop style | Vectorized style |
|------|------------|-------------------|
| Double every element | `for (i in seq_along(x)) x[i] <- x[i] * 2` | `x <- x * 2` |
| Classify by threshold | `for` loop with `if`/`else` per element | `ifelse(x >= t, "high", "low")` |
| Sum all elements | `total <- 0; for (v in x) total <- total + v` | `sum(x)` |

## Exercise

Write a script that loops over the vector `nums <- c(4, 15, 8, 23, 42, 7)`
with a `for` loop, printing `"even"` or `"odd"` for each number. Then rewrite
the same logic in one line using `ifelse()` and `%%`, and print the result
alongside the loop's output to confirm they match.
