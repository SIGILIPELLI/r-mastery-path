# 08 · Basic Plotting

## `plot()` — R's built-in plotting function

Base R ships with a capable plotting system, no packages required. `plot()`
adapts its behavior to the type of data you give it.

```r
ages <- c(23, 25, 22, 34, 28, 41, 19, 30)
scores <- c(88, 92, 79, 65, 85, 70, 95, 81)

plot(ages, scores)
# Opens a plot window (or saves to a device) with a scatter plot:
# ages on the x-axis, scores on the y-axis, one point per pair
```

## Customizing a scatter plot

```r
plot(ages, scores,
     main = "Age vs. Test Score",
     xlab = "Age",
     ylab = "Score",
     col = "steelblue",
     pch = 16)          # pch = point character; 16 is a solid filled circle
```

| Argument | Meaning |
|----------|---------|
| `main` | Plot title |
| `xlab` / `ylab` | Axis labels |
| `col` | Point/line color |
| `pch` | Point shape (0-25 are built-in symbols) |
| `type` | `"p"` points (default), `"l"` line, `"b"` both |
| `xlim` / `ylim` | Axis ranges, e.g. `xlim = c(0, 50)` |

## Line plots

```r
days <- 1:7
temps <- c(15, 17, 16, 19, 22, 21, 18)

plot(days, temps, type = "l", col = "darkred", lwd = 2,
     main = "Temperature Over a Week", xlab = "Day", ylab = "°C")
```

`type = "l"` connects points with a line instead of drawing separate markers;
`lwd` controls line width.

## Bar charts — `barplot()`

```r
sales <- c(120, 90, 150, 80)
names(sales) <- c("Q1", "Q2", "Q3", "Q4")

barplot(sales,
        main = "Quarterly Sales",
        ylab = "Units Sold",
        col = "seagreen")
```

`barplot()` expects a vector (ideally named, as above, so the names become
the bar labels automatically).

## Histograms — `hist()`

```r
set.seed(42)                       # makes the "random" data reproducible
exam_scores <- rnorm(200, mean = 75, sd = 10)   # 200 values from a normal distribution

hist(exam_scores,
     main = "Distribution of Exam Scores",
     xlab = "Score",
     col = "cornflowerblue",
     breaks = 15)                  # roughly how many bars to draw
```

A histogram groups continuous values into bins and shows how many fall into
each — the standard first look at any single numeric column's distribution.

## Boxplots — `boxplot()`

```r
group_a <- c(88, 92, 79, 65, 85)
group_b <- c(70, 95, 60, 55, 80)

boxplot(group_a, group_b,
        names = c("Group A", "Group B"),
        main = "Score Comparison",
        ylab = "Score",
        col = c("lightblue", "lightpink"))
```

A boxplot summarizes a distribution's median, quartiles, and outliers in one
compact shape — useful for comparing two or more groups at a glance.

## Saving a plot to a file

```r
png("scatter.png", width = 800, height = 600)   # open a PNG file device
plot(ages, scores, main = "Age vs. Score")
dev.off()                                        # close the device -- writes the file
```

Every plotting call between opening a device (`png()`, `pdf()`, `jpeg()`) and
`dev.off()` gets drawn into that file instead of the interactive plot window.
Forgetting `dev.off()` is a common mistake — the file stays empty/locked
until you close the device.

## Multiple plots in one figure

```r
par(mfrow = c(1, 2))     # 1 row, 2 columns of plots
plot(ages, scores, main = "Scatter")
hist(exam_scores, main = "Histogram")
par(mfrow = c(1, 1))     # reset back to a single plot per figure
```

## Base plotting cheat sheet

| Function | Use for |
|----------|---------|
| `plot(x, y)` | Scatter plot of two numeric vectors |
| `plot(x, y, type = "l")` | Line plot |
| `barplot(x)` | Bar chart of a (named) vector |
| `hist(x)` | Distribution of one numeric vector |
| `boxplot(x, y, ...)` | Compare distributions across groups |
| `png()` / `dev.off()` | Save a plot to a file |
| `par(mfrow = c(r, c))` | Arrange multiple plots in a grid |

Base R plotting is quick and dependency-free, which is why it's introduced
first. [Level 2](../level-2/04-ggplot2-basics.md) introduces **ggplot2**, the
more expressive and widely used plotting package for anything beyond a quick
look at your data.

## Exercise

Using the `exam_scores` vector generated above (`rnorm(200, mean = 75, sd =
10)` with `set.seed(42)`), produce three plots: a histogram with 20 breaks
and a title, a boxplot with a y-axis label, and a line plot of `sort(exam_scores)`
(the sorted scores) to visualize the distribution's shape. Save the histogram
to a file called `scores_hist.png` using `png()`/`dev.off()`.
