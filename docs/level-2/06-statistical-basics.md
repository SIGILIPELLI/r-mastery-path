# 06 · Statistical Basics

R was built by statisticians, for statistics — this is the one language
where "run a t-test" or "fit a linear regression" is a single built-in
function call, not a library you bolt on. This module covers the
descriptive and inferential statistics you'll reach for constantly:
summarizing a distribution, testing whether two groups genuinely differ,
measuring relationships between variables, and fitting a basic regression.

```r
set.seed(42)
group_a <- rnorm(30, mean = 75, sd = 8)   # simulated test scores, method A
group_b <- rnorm(30, mean = 70, sd = 9)   # simulated test scores, method B
```

`set.seed()` makes "random" number generation reproducible — every reader
who runs this code gets the exact same simulated numbers, which matters for
following along and for the exercise at the end.

## Descriptive statistics

```r
mean(group_a)      # 75.54869
median(group_a)      # 74.19687
sd(group_a)         # standard deviation -- 10.04022
var(group_a)         # variance (sd^2) -- 100.8061
range(group_a)       # c(min, max)
summary(group_a)     # min, 1st quartile, median, mean, 3rd quartile, max
quantile(group_a, probs = c(0.1, 0.9))  # arbitrary percentiles
```

```text
> summary(group_a)
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
  53.75   71.80   74.20   75.55   83.56   93.29
```

`sd()`/`var()` compute the **sample** standard deviation/variance (dividing
by `n - 1`, not `n`) — the standard choice when your data is a sample used
to estimate a population parameter, which is almost always the case in
practice.

## Visualizing before testing

Always look at a distribution before drawing conclusions from summary
numbers alone — two very different distributions can share the same mean:

```r
boxplot(group_a, group_b, names = c("Method A", "Method B"),
        main = "Score Distribution by Method", ylab = "Score")
```

See [Level 2, Module 4](04-ggplot2-basics.md) for the ggplot2 version of
this comparison, or [Level 1, Module 8](../level-1/08-basic-plotting.md)
for base R plotting.

## Correlation

```r
hours_studied <- c(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
exam_score <- c(52, 58, 60, 65, 68, 74, 78, 85, 88, 94)

cor(hours_studied, exam_score)
# [1] 0.9964304   -- close to 1: a strong positive linear relationship

cor(hours_studied, exam_score, method = "spearman")  # rank-based, robust to outliers
```

**Trap:** correlation measures *linear* association only. Two variables
can be perfectly related in a curved (non-linear) way and still show a
`cor()` near 0 — always plot the relationship (`plot(hours_studied,
exam_score)`) rather than trusting a single correlation number blindly.
Also remember the classic warning: correlation is not causation — a strong
`cor()` never by itself proves that one variable causes the other.

## The t-test — is a difference in means real, or noise?

`t.test()` answers: "given the natural variation in each group, is the
difference between their means bigger than you'd expect from random
chance alone?"

```r
t.test(group_a, group_b)
```

```text
	Welch Two Sample t-test

data:  group_a and group_b
t = 2.64, df = 57.789, p-value = 0.01064
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
  1.606367 11.685375
sample estimates:
mean of x mean of y
 75.54869  68.90282
```

Reading the output:

| Field | Meaning |
|-------|---------|
| `p-value` | probability of seeing a difference this large (or larger) *if the two groups actually had the same true mean* |
| `95 percent confidence interval` | a plausible range for the true difference in means |
| `mean of x` / `mean of y` | the two groups' observed means |

**The p-value trap:** a small p-value (conventionally `< 0.05`) means the
observed difference is unlikely to be pure chance — it does **not** mean
the difference is large, important, or practically meaningful. A tiny,
irrelevant difference can still produce a small p-value if the sample size
is large enough. Always look at the actual effect size (here, `mean of x -
mean of y`, roughly 6.6 points) alongside the p-value, never the p-value
alone.

By default `t.test()` runs **Welch's t-test**, which does not assume the
two groups have equal variance — the safer default. Pass `var.equal = TRUE`
only if you have a specific reason to assume equal variances.

## Paired vs. unpaired

If the same subjects were measured twice (before/after), use a **paired**
test — it's a fundamentally different comparison, testing whether the
*differences* are non-zero rather than comparing two independent groups:

```r
before <- c(70, 65, 80, 75, 60)
after <- c(75, 68, 85, 74, 65)

t.test(before, after, paired = TRUE)
```

Running an unpaired test on paired data throws away information (the
natural pairing) and typically understates your confidence in a real
effect — a subtle but common misapplication.

## Basic linear regression — `lm()`

```r
study_data <- data.frame(hours_studied, exam_score)

model <- lm(exam_score ~ hours_studied, data = study_data)
summary(model)
```

```text
Coefficients:
              Estimate Std. Error t value Pr(>|t|)
(Intercept)    46.9333     0.8538   54.97 1.33e-11 ***
hours_studied   4.5939     0.1376   33.38 7.07e-10 ***
---
Residual standard error: 1.25 on 8 degrees of freedom
Multiple R-squared:  0.9929,	Adjusted R-squared:  0.992
```

`exam_score ~ hours_studied` is R's **formula syntax**: "model `exam_score`
as a function of `hours_studied`." Reading the coefficients: the
`(Intercept)` (~46.9) is the predicted score at 0 hours studied, and the
`hours_studied` coefficient (~4.59) is the predicted score increase per
additional hour studied. `Multiple R-squared` (0.9929) means about 99.3% of
the variation in exam score is explained by hours studied in this
(deliberately clean, simulated) example — real data is rarely this tidy.

```r
predict(model, newdata = data.frame(hours_studied = 12))
#        1
# 102.0606
```

**Trap:** `predict()` will happily extrapolate far outside the range of
your original data (here, well past the observed 1-10 hours) without any
warning that it's doing so — a linear relationship that held for 1-10
hours studied has no guarantee of holding at 12, 20, or 50 hours. Be
skeptical of predictions outside your data's observed range.

## Statistics cheat sheet

| Task | Function |
|------|----------|
| Mean / median / sd / variance | `mean()`, `median()`, `sd()`, `var()` |
| Five-number summary | `summary(x)` |
| Percentiles | `quantile(x, probs = c(...))` |
| Linear correlation | `cor(x, y)` |
| Rank correlation (outlier-robust) | `cor(x, y, method = "spearman")` |
| Compare two group means | `t.test(a, b)` |
| Compare paired before/after | `t.test(before, after, paired = TRUE)` |
| Fit a linear model | `lm(y ~ x, data = df)` |
| Inspect a model's fit | `summary(model)` |
| Predict new values | `predict(model, newdata = ...)` |

## Exercise

Using `group_a` and `group_b` as defined at the top (with `set.seed(42)`
run first so your numbers match): run `t.test(group_a, group_b)` and write
one sentence stating whether the difference is statistically significant
at the conventional 0.05 threshold, quoting the actual p-value. Then fit
`lm(exam_score ~ hours_studied, data = study_data)` using the `study_data`
defined above, and use `predict()` to estimate the score for someone who
studied 6 hours — then compare it to the actual observed value at 6 hours
in the data and note the difference.
