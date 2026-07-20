# 06 · Data Frames Basics

## What a data frame is

A **data frame** is R's core structure for tabular data — rows are
observations, columns are variables, and each column is a vector (so every
value in a column shares one type, but different columns can differ).
It's the structure you'll spend the most time with in every later module.

```r
students <- data.frame(
    name = c("Alice", "Bob", "Carol"),
    age = c(23, 25, 22),
    passed = c(TRUE, FALSE, TRUE)
)

students
#     name age passed
# 1  Alice  23   TRUE
# 2    Bob  25  FALSE
# 3  Carol  22   TRUE
```

## Inspecting a data frame

```r
str(students)         # STRucture: compact summary of columns and types
# 'data.frame':	3 obs. of  3 variables:
#  $ name  : chr  "Alice" "Bob" "Carol"
#  $ age   : num  23 25 22
#  $ passed: logi  TRUE FALSE TRUE

nrow(students)         # [1] 3
ncol(students)         # [1] 3
dim(students)          # [1] 3 3
names(students)        # [1] "name"   "age"    "passed"
colnames(students)     # same as names() for data frames
head(students, 2)      # first 2 rows
summary(students)      # per-column summary stats
```

`str()` is one of the most useful functions in all of R for a first look at
any unfamiliar object — reach for it constantly.

## Selecting columns

```r
students$name              # a vector: "Alice" "Bob" "Carol"
students[["age"]]           # equivalent to $ -- a vector
students["age"]              # a one-column DATA FRAME, not a vector
students[, "age"]            # a vector -- comma selects "all rows, this column"
students[, c("name", "age")] # a two-column data frame
```

## Selecting rows

```r
students[1, ]                # first row (all columns)
#    name age passed
# 1 Alice  23   TRUE

students[1:2, ]              # first two rows
students[students$age > 22, ]   # filter by condition -- rows where TRUE
#    name age passed
# 1 Alice  23   TRUE
# 2   Bob  25  FALSE
```

The general pattern is `df[rows, columns]` — leaving either side blank means
"all". This is the base-R way to filter; [Level 2](../level-2/01-dplyr-deep-dive.md)
introduces `dplyr::filter()`, which reads more clearly for complex conditions.

## Adding and modifying columns

```r
students$grade <- c("B", "C", "A")
students
#     name age passed grade
# 1  Alice  23   TRUE     B
# 2    Bob  25  FALSE     C
# 3  Carol  22   TRUE     A

students$age <- students$age + 1   # modify an existing column in place
students$age
# [1] 24 26 23
```

## Adding rows

```r
new_student <- data.frame(name = "Dave", age = 30, passed = TRUE, grade = "A")
students <- rbind(students, new_student)
nrow(students)
# [1] 4
```

`rbind()` requires the new data frame to have the same column names and
compatible types; a mismatch throws an error rather than silently corrupting
data.

## Sorting

```r
students[order(students$age), ]           # ascending by age
students[order(-students$age), ]          # descending (negate to reverse)
students[order(students$passed, students$age), ]  # multiple sort keys
```

## Basic aggregation

```r
mean(students$age)
# [1] 25.75

tapply(students$age, students$passed, mean)
# FALSE  TRUE
#    26 23.66667
```

`tapply()` groups a vector by another vector and applies a function to each
group — a preview of the group-by pattern that `dplyr::group_by()` handles
more readably in [Level 2](../level-2/01-dplyr-deep-dive.md).

## `data.frame` cheat sheet

| Task | Syntax |
|------|--------|
| Create | `data.frame(col1 = ..., col2 = ...)` |
| Inspect structure | `str(df)` |
| Dimensions | `nrow(df)`, `ncol(df)`, `dim(df)` |
| Column as vector | `df$col` or `df[["col"]]` or `df[, "col"]` |
| Column as data frame | `df["col"]` |
| Filter rows | `df[df$col > 10, ]` |
| Add/modify column | `df$new_col <- ...` |
| Add row | `rbind(df, new_row_df)` |
| Sort | `df[order(df$col), ]` |
| Group + summarize | `tapply(df$value, df$group, mean)` |

## Exercise

Create a data frame `products` with columns `name` (character), `price`
(numeric), and `category` (character) for 5 made-up products across 2
categories. Print `str(products)`, then filter to only products with
`price > 10`, then add a new logical column `on_sale` of your choosing, and
finally use `tapply()` to print the average price per category.
