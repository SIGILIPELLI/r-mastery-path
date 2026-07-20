# 04 · Functions

## Defining and calling a function

```r
greet <- function(name) {
    message <- paste("Hello,", name, "!")
    return(message)
}

greet("Alice")
# [1] "Hello, Alice !"
```

A function is just a value bound to a name with `<-`, like anything else in
R. `function(...)` creates the function object; the `{ }` block is its body.

## Implicit return

The last evaluated expression in a function body is returned automatically —
an explicit `return()` is optional and mostly used to exit early:

```r
square <- function(x) {
    x * x     # no "return" needed -- this is the last expression
}

square(5)
# [1] 25
```

```r
safe_divide <- function(a, b) {
    if (b == 0) {
        return(NA)     # early return -- explicit here on purpose
    }
    a / b               # implicit return for the normal case
}

safe_divide(10, 2)
# [1] 5
safe_divide(10, 0)
# [1] NA
```

## Default parameter values

```r
power <- function(base, exponent = 2) {
    base ^ exponent
}

power(5)          # 25 -- uses the default exponent
# [1] 25
power(5, 3)       # 125 -- overrides the default
# [1] 125
power(base = 2, exponent = 10)   # named arguments -- order doesn't matter
# [1] 1024
```

## Named vs. positional arguments

R lets you mix positional and named arguments; named arguments can appear in
any order:

```r
describe_person <- function(name, age, city) {
    paste(name, "is", age, "years old and lives in", city)
}

describe_person("Sam", 30, "Boston")               # positional
describe_person(age = 30, city = "Boston", name = "Sam")   # named, any order
describe_person("Sam", city = "Boston", age = 30)   # mixed
# All three produce: "Sam is 30 years old and lives in Boston"
```

## `...` — variable numbers of arguments

The `...` ("dots") parameter collects any number of extra arguments, which
you can forward to another function:

```r
summarize_all <- function(...) {
    values <- c(...)
    cat("Sum:", sum(values), " Mean:", mean(values), "\n")
}

summarize_all(1, 2, 3, 4, 5)
# Sum: 15  Mean: 3
```

`...` is also how functions like `paste()` and `cat()` themselves accept an
arbitrary number of arguments, and how wrapper functions forward unknown
arguments to an underlying function without listing every one explicitly.

## Functions as values

Functions are ordinary values in R — you can store them in variables, pass
them as arguments, and return them from other functions:

```r
apply_twice <- function(f, x) {
    f(f(x))
}

add_one <- function(x) x + 1

apply_twice(add_one, 5)
# [1] 7   -- add_one(add_one(5)) = add_one(6) = 7
```

## Anonymous (lambda) functions

```r
# Full anonymous function syntax
sapply(c(1, 2, 3), function(x) x ^ 2)
# [1] 1 4 9

# R 4.1+ shorthand: \(x) is equivalent to function(x)
sapply(c(1, 2, 3), \(x) x ^ 2)
# [1] 1 4 9
```

`sapply()` here applies the anonymous function to every element of the
vector — a first look at the apply family, covered fully in
[Level 2](../level-2/03-apply-family-functions.md).

## Scope basics

A function has its own local environment: variables created inside it don't
leak out, and by default a function reads (but does not modify) variables
from the enclosing scope:

```r
x <- 10

modify <- function() {
    x <- 20        # creates a NEW local x -- does not touch the outer one
    print(x)
}

modify()
# [1] 20
print(x)
# [1] 10   -- unchanged
```

To deliberately modify a variable in an enclosing scope (rare, and generally
avoided in favor of returning a new value), R has the `<<-` "superassignment"
operator:

```r
counter <- 0

increment <- function() {
    counter <<- counter + 1   # modifies the outer counter, not a local copy
}

increment()
increment()
print(counter)
# [1] 2
```

!!! warning "Prefer return values over `<<-`"
    Relying on `<<-` to mutate outer variables makes code harder to reason
    about, since a function's effect isn't visible from its return value
    alone. It's useful to know it exists (you'll see it in some counter/cache
    patterns), but idiomatic R almost always prefers a function that takes
    inputs and returns a new value.

## Function basics cheat sheet

| Concept | Syntax |
|---------|--------|
| Define | `f <- function(x, y = 1) { ... }` |
| Call positionally | `f(10, 20)` |
| Call by name | `f(y = 20, x = 10)` |
| Variadic args | `f <- function(...) { c(...) }` |
| Anonymous function | `function(x) x + 1` or `\(x) x + 1` |
| Early exit | `return(value)` |
| Modify outer scope | `x <<- new_value` (avoid unless needed) |

## Exercise

Write a function `bmi(weight_kg, height_m)` that returns the body mass index
(`weight_kg / height_m^2`), rounded to 1 decimal place with `round()`. Give
`height_m` a default of `1.7`. Then write a second function
`classify_bmi(bmi_value)` that returns `"Underweight"`, `"Normal"`,
`"Overweight"`, or `"Obese"` based on standard BMI thresholds (below 18.5,
18.5–24.9, 25–29.9, 30+). Call `classify_bmi(bmi(70, 1.75))` and print the
result.
