# 02 · Variables & Types

## Assignment

R's idiomatic assignment operator is `<-`, though `=` works identically at
the top level (they diverge in edge cases, e.g. inside function-call
arguments, where `=` sets a named argument rather than assigning a variable).

```r
x <- 42          # idiomatic
y = 42           # also works, less idiomatic outside function arguments
42 -> z          # right-to-left assignment also exists, rarely used

x
# [1] 42
```

## Basic types

R has four core atomic types you'll use constantly, plus a fifth for missing
data:

```r
n <- 42L          # integer (the L suffix forces integer, not double)
d <- 3.14         # double (the default for any plain number)
s <- "hello"      # character (R has no separate single-char "char" type)
b <- TRUE         # logical (TRUE/FALSE, or the shorthand T/F)
m <- NA           # "Not Available" -- a missing value, valid in any type

class(n)
# [1] "integer"
class(d)
# [1] "numeric"
class(s)
# [1] "character"
class(b)
# [1] "logical"
class(m)
# [1] "logical"  -- a bare NA defaults to logical unless typed otherwise
```

!!! warning "`42` is a double, not an integer, by default"
    Typing `x <- 42` makes `x` a **double** (`numeric`). To get a true
    integer you must write `42L`. This rarely matters for everyday scripts,
    but it matters for `class()` checks and for interfacing with code that
    expects a specific type.

## Checking and converting types

```r
x <- "123"
is.character(x)    # TRUE
is.numeric(x)      # FALSE

as.numeric(x)      # 123 -- converts the string to a double
as.character(456)  # "456" -- converts a number to a string
as.integer(3.9)    # 3 -- truncates, does not round

as.numeric("abc")
# [1] NA
# Warning message: NAs introduced by coercion
```

Failed numeric conversions produce `NA` with a warning rather than crashing —
a common source of silent data-quality bugs, so it's worth checking for `NA`
after converting user-provided or file-read data.

## `NULL` vs. `NA`

These are easy to confuse:

- **`NA`** means "a value exists conceptually, but it's missing" — e.g. a
  survey respondent skipped a question. It has a type and takes up a slot in
  a vector.
- **`NULL`** means "nothing here at all" — an empty, zero-length object. It
  cannot be an element of a vector; assigning `NULL` to a list element
  *removes* that element.

```r
x <- c(1, NA, 3)
length(x)
# [1] 3   -- NA still occupies a slot

y <- NULL
length(y)
# [1] 0   -- NULL is nothing

is.na(x)
# [1] FALSE  TRUE FALSE
```

## Vectors are the default container

Even a "single value" in R is technically a length-1 vector — there's no
separate scalar type:

```r
x <- 5
length(x)
# [1] 1

x <- c(5)   # identical to just 5
```

Combine values into a longer vector with `c()` ("combine"):

```r
ages <- c(25, 31, 42, 19)
ages
# [1] 25 31 42 19

names_vec <- c("Alice", "Bob", "Carol")
names_vec
# [1] "Alice" "Bob"   "Carol"
```

A vector holds a **single type** — mixing types forces R to coerce
everything to the "widest" common type (logical → integer → double →
character):

```r
mixed <- c(1, "two", TRUE)
mixed
# [1] "1"    "two"  "TRUE"   -- everything became character
class(mixed)
# [1] "character"
```

## Operators

```r
7 + 3     # 10
7 - 3     # 4
7 * 3     # 21
7 / 3     # 2.333333
7 %/% 3   # 2   -- integer division
7 %% 3    # 1   -- modulo (remainder)
7 ^ 2     # 49  -- exponentiation (** also works)

5 == 5    # TRUE
5 != 4    # TRUE
5 > 4     # TRUE
5 >= 5    # TRUE

TRUE && FALSE   # FALSE -- logical AND (scalar, short-circuits)
TRUE || FALSE   # TRUE  -- logical OR (scalar, short-circuits)
!TRUE           # FALSE -- logical NOT

c(TRUE, FALSE) & c(TRUE, TRUE)   # element-wise AND: TRUE FALSE
```

!!! info "`&&`/`||` vs. `&`/`|`"
    Use `&&` and `||` for single logical values (e.g. in `if` conditions) —
    they short-circuit and only look at the first element. Use `&` and `|`
    for element-wise operations across whole vectors, which is what you'll
    do constantly once you're filtering data frames.

## Naming conventions

R style guides (e.g. the tidyverse style guide) favor `snake_case` for
variable and function names:

```r
student_age <- 21        # preferred
studentAge <- 21         # valid, but not idiomatic in R
StudentAge <- 21         # valid, but reads as a class/constructor name
```

Variable names must start with a letter or a dot (not followed by a digit),
and can contain letters, digits, `.`, and `_`. Reserved words like `if`,
`function`, and `TRUE` cannot be used as names.

## Type cheat sheet

| Type | Example | `class()` |
|------|---------|-----------|
| Double | `3.14`, `42` | `"numeric"` |
| Integer | `42L` | `"integer"` |
| Character | `"hello"` | `"character"` |
| Logical | `TRUE`, `FALSE` | `"logical"` |
| Missing | `NA` | type of the containing vector |
| Nothing | `NULL` | `"NULL"` |

## Exercise

Create variables for a person's `name` (character), `age` (integer, using the
`L` suffix), `height_m` (double), and `is_student` (logical). Print each one's
value and its `class()`. Then create a vector `mixed <- c(age, name)` and
print it along with its `class()` — explain in a comment why the numeric
value got converted to a string.
