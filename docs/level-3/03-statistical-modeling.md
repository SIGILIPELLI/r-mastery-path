# 03 · Statistical Modeling

R's modeling functions all share a common interface — a `formula` (`y ~
x`) and a `data` argument — whether you're fitting a straight line, a
multi-predictor regression, or a logistic classifier. This module covers
`lm()`, `glm()`, `t.test()`, and the `broom` package, which turns R's
often awkward native model objects into tidy tibbles you can pipe like
any other data.

```r
set.seed(42)
library(broom)
```

## Simple linear regression

```r
n <- 100
x <- rnorm(n, mean = 50, sd = 10)
y <- 3 + 2.5 * x + rnorm(n, sd = 15)
df <- data.frame(x = x, y = y)

fit <- lm(y ~ x, data = df)
summary(fit)
# Coefficients:
#             Estimate Std. Error t value Pr(>|t|)
# (Intercept)  -0.3624     6.7564  -0.054    0.957
# x             2.5407     0.1315  19.322   <2e-16 ***
# Multiple R-squared:  0.7921
```

`y ~ x` reads as "model y as a function of x." The fitted intercept
(-0.36) and slope (2.54) are close to the true simulated values (3 and
2.5) — close, not exact, because the data has random noise added; a
p-value near zero for `x` says the slope is very unlikely to be zero by
chance, and `R-squared` of 0.79 says the model explains about 79% of the
variance in `y`.

## Tidying model output with broom

`summary(fit)` is designed for reading in the console, not for further
computation — `broom` extracts the same numbers into tibbles:

```r
tidy(fit)
#   term        estimate std.error statistic  p.value
#   (Intercept)   -0.362     6.76     -0.0536 9.57e-1
#   x              2.54      0.131    19.3    3.41e-35

glance(fit)
#   r.squared adj.r.squared sigma statistic   p.value    AIC   BIC
#       0.792         0.790  13.6      373. 3.41e-35    810.   818.
```

`tidy()` gives one row per coefficient (useful for comparing many models
or building a coefficient plot); `glance()` gives one row of whole-model
summary statistics (useful for comparing model fit across candidates).

## Predicting on new data

```r
newdata <- data.frame(x = c(40, 60))
predict(fit, newdata, interval = "confidence")
#        fit      lwr      upr
# 1 101.267   97.45   105.08
# 2 152.082  148.38   155.78
```

`interval = "confidence"` returns a range for the *average* y at that x;
`interval = "prediction"` (not shown) returns a wider range for a *single
new observation*, since individual points vary more than the average
does — using the wrong one understates uncertainty for individual
predictions.

## Multiple regression

```r
z <- rnorm(n, mean = 5, sd = 2)
y2 <- 3 + 2 * x - 1.5 * z + rnorm(n, sd = 10)
df2 <- data.frame(x = x, z = z, y2 = y2)

fit2 <- lm(y2 ~ x + z, data = df2)
summary(fit2)
# x   2.058   (true: 2)
# z  -1.655   (true: -1.5)
# Multiple R-squared:  0.8657
```

`y ~ x + z` adds `z` as a second predictor in the same model; each
coefficient is then interpreted "holding the other predictor constant" —
the effect of `x` on `y2` after accounting for whatever `z` explains, not
the raw correlation between `x` and `y2` alone.

## Logistic regression with glm()

```r
prob <- 1 / (1 + exp(-(-5 + 0.1 * x)))
y3 <- rbinom(n, 1, prob)
df3 <- data.frame(x = x, y3 = y3)

fit3 <- glm(y3 ~ x, data = df3, family = binomial)
tidy(fit3)
#   term        estimate std.error statistic p.value
#   (Intercept)  -3.99     1.23      -3.24   0.00121
#   x             0.0836   0.0242     3.45   0.000553
```

`family = binomial` is what turns `glm()` into logistic regression instead
of another linear model — the coefficients are on the *log-odds* scale, so
`exp(coef(fit3)["x"])` (not shown) gives the odds ratio per unit of `x`,
which is usually the number worth reporting.

## Comparing two groups: t.test()

```r
group_a <- rnorm(30, mean = 100, sd = 15)
group_b <- rnorm(30, mean = 108, sd = 15)
t.test(group_a, group_b)
# t = -1.3205, df = 53.714, p-value = 0.1923
# 95 percent confidence interval: -13.14  2.70
```

`t.test()` defaults to Welch's t-test (unequal variances assumed), which
is the safer default when you haven't separately verified the two groups
have equal variance — pass `var.equal = TRUE` only if you have a reason
to believe otherwise.

## The factor-as-predictor trap

```r
df$grp <- factor(sample(c("A", "B", "C"), n, replace = TRUE))
fit4 <- lm(y ~ x + grp, data = df)
coef(fit4)
# (Intercept)           x        grpB        grpC
#  -0.07717     2.50926   4.95667    -0.39508
levels(df$grp)
# [1] "A" "B" "C"
```

**Trap:** `lm()` silently drops one level of a factor predictor as the
*reference level* (here `"A"`, because it's first alphabetically) and
reports every other level's coefficient as a difference *relative to
that reference* — there is no `coef` for `grpA` because it's baked into
the intercept. Two consequences catch people off guard: which level
becomes the reference depends on factor level order (alphabetical unless
you set it explicitly, same trap as the ggplot2 module), and comparing
`grpB` vs `grpC` directly requires either releveling with `relevel(df$grp,
ref = "B")` or a follow-up contrast — you cannot read that comparison off
the default output.

## Cheat sheet

| Task | Function |
|---|---|
| Linear regression | `lm(y ~ x, data = df)` |
| Logistic regression | `glm(y ~ x, data = df, family = binomial)` |
| Tidy per-coefficient output | `broom::tidy(fit)` |
| Tidy whole-model summary | `broom::glance(fit)` |
| Predict on new data | `predict(fit, newdata, interval = "confidence")` |
| Compare two group means | `t.test(a, b)` |
| Change a factor's reference level | `relevel(f, ref = "level")` |
| Log-odds → odds ratio | `exp(coef(fit))` |

## Exercise

1. Simulate a predictor `x` and outcome `y` with a known slope, fit
   `lm(y ~ x)`, and confirm the fitted slope is close to the value you
   simulated with. Then add heavier noise (`sd = 50`) and refit — watch
   `R-squared` drop even though the true relationship didn't change.
2. Fit a `glm()` with `family = binomial` on simulated pass/fail data and
   convert the `x` coefficient to an odds ratio with `exp()` — write one
   sentence interpreting what that number means.
3. Create a factor predictor with 4 levels, fit an `lm()` with it, and use
   `relevel()` to change the reference level — confirm the *fitted
   values* are identical before and after, even though the coefficients
   look completely different.
