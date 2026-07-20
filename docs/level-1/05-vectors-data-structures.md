# 05 · Vectors & Basic Data Structures

## Creating vectors

```r
nums <- c(10, 20, 30, 40, 50)
nums
# [1] 10 20 30 40 50

seq_vec <- 1:10                  # sequence shorthand
seq_vec
# [1] 1 2 3 4 5 6 7 8 9 10

by_step <- seq(0, 100, by = 25)  # explicit step
by_step
# [1] 0 25 50 75 100

repeated <- rep("x", times = 4)
repeated
# [1] "x" "x" "x" "x"
```

## Indexing — 1-based, not 0-based

R vectors are indexed starting at **1**, not 0 — a frequent source of
off-by-one bugs for people coming from Python, JavaScript, or C.

```r
fruits <- c("apple", "banana", "cherry", "date")

fruits[1]        # "apple"  -- the FIRST element, not the second
# [1] "apple"
fruits[length(fruits)]   # "date" -- the last element
# [1] "date"

fruits[2:3]      # slice: elements 2 through 3
# [1] "banana" "cherry"

fruits[-1]       # negative index: everything EXCEPT element 1
# [1] "banana" "cherry" "date"

fruits[c(1, 3)]  # select specific, non-contiguous indices
# [1] "apple"  "cherry"
```

## Logical (boolean) indexing

Indexing with a logical vector of the same length keeps only the elements
where the condition is `TRUE` — this is the foundation of filtering data
later on:

```r
ages <- c(15, 22, 8, 35, 61, 19)

ages[ages >= 18]
# [1] 22 35 61 19

adults_only <- ages >= 18
adults_only
# [1] FALSE  TRUE FALSE  TRUE  TRUE  TRUE
ages[adults_only]
# [1] 22 35 61 19
```

## Named vectors

```r
prices <- c(apple = 0.5, banana = 0.3, cherry = 2.0)
prices
#  apple banana cherry
#    0.5    0.3    2.0

prices["banana"]
# banana
#    0.3

names(prices)
# [1] "apple"  "banana" "cherry"
```

## Vectorized arithmetic

Operations on vectors apply element-wise, without an explicit loop:

```r
a <- c(1, 2, 3)
b <- c(10, 20, 30)

a + b
# [1] 11 22 33
a * 2
# [1] 2 4 6

# Recycling: the shorter vector repeats to match the longer one's length
c(1, 2, 3, 4) + c(1, 0)
# [1] 2 2 4 4
```

!!! warning "Recycling can hide bugs"
    If the longer vector's length isn't an exact multiple of the shorter
    one's, R still recycles but issues a warning. Silent recycling on
    mismatched lengths is a common source of subtle data bugs — check
    `length()` on both sides if a vectorized result looks wrong.

## Useful vector functions

```r
nums <- c(5, 3, 8, 1, 9, 3)

length(nums)     # 6
sum(nums)        # 29
mean(nums)       # 4.833333
max(nums)        # 9
min(nums)        # 1
sort(nums)       # 1 3 3 5 8 9
rev(nums)        # 3 9 1 8 3 5  -- reversed order
unique(nums)     # 5 3 8 1 9    -- duplicates removed
which(nums > 4)  # 1 3 5 -- INDICES where the condition holds, not values
```

## Lists — mixed-type containers

Unlike a vector (single type), a **list** can hold elements of different
types, including other lists or vectors — R's general-purpose container:

```r
person <- list(name = "Alice", age = 30, scores = c(90, 85, 88))

person$name
# [1] "Alice"
person$scores
# [1] 90 85 88
person[["age"]]     # equivalent to person$age
# [1] 30
```

`$` and `[["..."]]` both extract a named element; `$` is more concise but
only works with literal names (not a variable holding a name), while
`[["..."]]` works with either.

```r
field <- "age"
person[[field]]     # works -- [[ ]] accepts a variable
# [1] 30
# person$field     # would NOT work -- this looks for a literal field called "field"
```

## `[ ]` vs. `[[ ]]` on lists

A crucial distinction: `[ ]` on a list returns a **sub-list** (still a list),
while `[[ ]]` returns the actual element inside:

```r
person["age"]
# $age
# [1] 30
class(person["age"])
# [1] "list"

person[["age"]]
# [1] 30
class(person[["age"]])
# [1] "numeric"
```

## Vector and list cheat sheet

| Task | Syntax |
|------|--------|
| Create a vector | `c(1, 2, 3)` |
| Sequence | `1:10` or `seq(0, 100, by = 25)` |
| First element | `x[1]` (1-based!) |
| Last element | `x[length(x)]` |
| Filter by condition | `x[x > 10]` |
| Named access | `x["name"]` |
| Create a list | `list(a = 1, b = "two")` |
| Get list element (unwrapped) | `lst[["a"]]` or `lst$a` |
| Get list element (still a list) | `lst["a"]` |

## Exercise

Create a named vector `inventory` mapping three item names to their integer
counts (e.g. `c(apples = 10, bananas = 0, cherries = 25)`). Use logical
indexing to print only the items with a count greater than 0. Then create a
list `product <- list(name = "Widget", price = 9.99, in_stock = TRUE)` and
print each field using both `$` and `[[ ]]` syntax to confirm they give the
same result.
