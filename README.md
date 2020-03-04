
<!-- README.md is generated from README.Rmd. Please edit that file -->

# MNLpred - Simulated Predictions From Multinomial Logistic Models

<!-- badges: start -->

[![Travis build
status](https://travis-ci.org/ManuelNeumann/MNLpred.svg?branch=master)](https://travis-ci.org/ManuelNeumann/MNLpred)
[![License: GPL
v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/MNLpred)](https://cran.r-project.org/package=MNLpred)
[![downloads](http://cranlogs.r-pkg.org/badges/MNLpred)](https://CRAN.R-project.org/package=MNLpred)
<!-- badges: end -->

This package provides functions that make it easy to get plottable
predictions from multinomial logit models. The predictions are based on
simulated draws of regression estimates from their respective sampling
distribution.

At first I will present the theoretical and statistical background,
before using sample data to demonstrate the functions of the package.

## The multinomial logit model

For the statistical and theoretical background of the multinomial logit
regression please refer to the vignette or sources like [these lecture
notes by Germán
Rodríguez](https://data.princeton.edu/wws509/notes/c6s2).

Due to the inconvenience of integrating math equations in the README
file, this is not the place to write comprehensively about it.

These are the important characteristics of the model:

  - The multinomial logit regression is used to model nominal outcomes.
    It provides the opportunity to assign specific choices a
    probability, based on a set of independent variables.
  - The model needs an assigned baseline category to be identifiable.
    All other choices are evaluated in contrast to this reference.
  - The model returns a set of coefficients for each choice category.
  - Like all logit models, the multinomial logit model returns log-odds
    which are difficult to interpret in terms of effect sizes and
    uncertainties.

This package helps to interpret the model in meaningful ways.

## Using the package

### Installing

The package can be both installed from CRAN or the github repository:

``` r
# Uncomment if necessary:

# install.packages("MNLpred")
# devtools::install_github("ManuelNeumann/MNLpred")
```

### How does the function work?

As we have seen above, the multinomial logit can be used to get an
insight into the probabilities to choose one option out of a set of
alternatives. We have also seen that we need a baseline category to
identify the model. This is mathematically necessary, but does not come
in handy for purposes of interpretation.

It is far more helpful and easier to understand to come up with
predicted probabilities and first differences for values of interest
(see e.g., King, Tomz, and Wittenberg 2000 for approaches in social
sciences). Based on simulations, this package helps to easily predict
probabilities and their uncertainty in forms of confidence intervals for
each choice category over a specified scenario.

The procedure follows the following steps:

1.  Estimate a multinomial model and save the coefficients and the
    variance covariance matrix (based on the Hessian-matrix of the
    model).
2.  To simulate uncertainty, make n draws of coefficients from a
    simulated sampling distribution based on the coefficients and the
    variance covariance matrix.
3.  Predict probabilities by multiplying the drawn coefficients with a
    specified scenario (the observed values).
4.  Take the mean and the quantiles of the simulated predicted
    probabilities.

The presented functions follow these steps. Additionally, they use the
so called observed value approach. This means that the “scenario” uses
all observed values that informed the model. Therefore the function
takes these more detailed steps:

1.  For all (complete) cases n predictions are computed based on their
    observed independent values and the n sets of coefficients.
2.  Next, the predicted values of all observations for each simulation
    are averaged.
3.  Take the mean and the quantiles of the simulated predicted
    probabilities (same as above).

For first differences, the simulated predictions are subtracted from
each other.

To showcase these steps, I present a reproducible example of how the
functions can be used.

### Example

The example is based on [this UCLA R data analysis
example](https://stats.idre.ucla.edu/r/dae/multinomial-logistic-regression/).

The data is an example dataset, including the career choice of 200 high
school students and their respective performance indicators. We want to
predict the probability of the students to choose either an academic,
general, or vocational program.

For this task, we need the following packages:

``` r
# Reading data
library(foreign)

# Required packages
library(magrittr) # for pipes
library(nnet) # for the multinom()-function
library(MASS) # for the multivariate normal distribution

# The package
library(MNLpred)

# Plotting the predicted probabilities:
library(ggplot2)
library(scales)
```

Now we load the data:

``` r
# The data:
ml <- read.dta("https://stats.idre.ucla.edu/stat/data/hsbdemo.dta")
```

As we have seen above, we need a baseline or reference category for the
model to work. With the function `relevel()` we set the category
`"academic"` as the baseline. Additionally, we compute a numeric dummy
for the gender variable to include it in the model.

``` r
# Data preparation:

# Set "academic" as the reference category for the multinomial model
ml$prog2 <- relevel(ml$prog, ref = "academic")

# Computing a numeric dummy for "female" (= 1)
ml$female2 <- as.numeric(ml$female == "female")
```

The next step is to compute the actual model. The function of the
`MNLpred` package is based on models that were estimated with the
`multinom()`-function of the `nnet` package. The `multinom()` function
is convenient because it does not need transformed datasets. The syntax
is very easy and resembles the ordinary regression functions. Important
is that the Hessian matrix is returned with `Hess = TRUE`. The matrix is
needed to simulate the sampling distribution.

``` r
# Multinomial logit model:
mod1 <- multinom(prog2 ~ female2 + read + write + math + science,
                 Hess = TRUE,
                 data = ml)
#> # weights:  21 (12 variable)
#> initial  value 219.722458 
#> iter  10 value 189.686272
#> final  value 168.079235 
#> converged
```

The results show the coefficients and standard errors. As we can see,
there are two sets of coefficients. They describe the relationship
between the reference category and the choices `general` and `vocation`.

``` r
summary(mod1)
#> Call:
#> multinom(formula = prog2 ~ female2 + read + write + math + science, 
#>     data = ml, Hess = TRUE)
#> 
#> Coefficients:
#>          (Intercept)   female2        read       write        math
#> general     4.314585 0.2180419 -0.05466370 -0.03863058 -0.09931014
#> vocation    8.592285 0.3618313 -0.05535549 -0.07165604 -0.12226602
#>             science
#> general  0.09386869
#> vocation 0.06388337
#> 
#> Std. Errors:
#>          (Intercept)   female2       read      write       math    science
#> general     1.444954 0.4368366 0.02816753 0.03088249 0.03307516 0.03007196
#> vocation    1.553752 0.4595495 0.03060286 0.03142334 0.03598240 0.03020753
#> 
#> Residual Deviance: 336.1585 
#> AIC: 360.1585
```

A first rough review of the coefficients shows that higher math scores
lead to a lower probability of the students to choose a general or
vocational track. It is hard to evaluate whether the effect is
statistically significant and how the probabilities for each choice look
like. For this it is helpful to predict the probabilities for certain
scenarios and plot the means and confidence intervals for visual
analysis.

Let’s say we are interested in the relationship between the math scores
and the probability to choose one or the other type of track. It would
be helpful to plot the predicted probabilities for the span of the math
scores.

``` r
summary(ml$math)
#>    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
#>   33.00   45.00   52.00   52.65   59.00   75.00
```

As we can see, the math scores range from 33 to 75. Let’s pick this
score as the x-variable (`xvari`) and use the `mnl_pred_ova()` function
to get predicted probabilities for each math score in this range.

The function needs a multinomial logit model (`model`), data (`data`),
the variable of interest `xvari`, the steps for which the probabilities
should be predicted (`by`). Additionally, a `seed` can be defined for
replication purposes, the numbers of simulations can be defined
(`nsim`), and the confidence intervals (`probs`).

If we want to hold another variable stable, we can specify so with
`scennname`and `scenvalue`. See also the `mnl_fd_ova()` function below.

``` r
pred1 <- mnl_pred_ova(model = mod1,
                      data = ml,
                      xvari = "math",
                      by = 1,
                      seed = "random", # default
                      nsim = 100, # faster
                      probs = c(0.025, 0.975)) # default
```

The function returns a list with several elements. Most importantly, it
returns a `plotdata` data set:

``` r
pred1$plotdata %>% head()
#> # A tibble: 6 x 5
#>    math prog2     mean  lower upper
#>   <dbl> <fct>    <dbl>  <dbl> <dbl>
#> 1    33 academic 0.153 0.0497 0.315
#> 2    34 academic 0.165 0.0577 0.324
#> 3    35 academic 0.178 0.0669 0.331
#> 4    36 academic 0.191 0.0776 0.341
#> 5    37 academic 0.206 0.0897 0.354
#> 6    38 academic 0.221 0.103  0.366
```

As we can see, it includes the range of the x variable, a mean, a lower,
and an upper bound of the confidence interval. Concerning the choice
category, the data is in a long format. This makes it easy to plot it
with the `ggplot` syntax. The choice category can now easily be used to
differentiate the lines in the plot by using `linetype = prog2` in the
`aes()`. Another option is to use `facet_wrap()` or `facet_grid()` to
differentiate the predictions:

``` r
ggplot(data = pred1$plotdata, aes(x = math, y = mean,
                                  ymin = lower, ymax = upper)) +
  geom_ribbon(alpha = 0.1) + # Confidence intervals
  geom_line() + # Mean
  facet_grid(prog2 ~., scales = "free_y") +
  scale_y_continuous(labels = percent_format(accuracy = 1)) + # % labels
  theme_bw() +
  labs(y = "Predicted probabilities",
       x = "Math score") # Always label your axes ;)
```

![](man/figures/README-prediction_plot1-1.png)<!-- -->

If we want first differences between two scenarios, we can use the
function `mnl_fd2_ova()`. The function takes similar arguments as the
function above, but now the values for the scenarios of interest have to
be supplied. Imagine we want to know what difference it makes to have
the lowest or highest `math` score. This can be done as follows:

``` r
fdif1 <- mnl_fd2_ova(model = mod1,
                     data = ml,
                     xvari = "math",
                     value1 = min(ml$math),
                     value2 = max(ml$math),
                     nsim = 100)
```

The first differences can then be depicted in a graph.

``` r
ggplot(fdif1$plotdata_fd, aes(categories, y = mean,
                              ymin = lower, max = upper)) +
  geom_pointrange() +
  geom_hline(yintercept = 0) +
  scale_y_continuous(labels = percent_format()) +
  theme_bw()
```

![](man/figures/README-static_fd_plot-1.png)<!-- -->

We are often not only interested in the static difference, but the
difference across a span of values, given a difference in a second
variable. This is especially helpful when we look at dummy variables.
For example, we could be interested in the effect of `female`. With the
`mnl_fd_ova()` function, we can predict the probabilities for two
scenarios and subtract them. The function returns the differences and
the confidence intervals of the differences. The different scenarios can
be held stable with `scenname` and the `scenvalues`. `scenvalues` takes
a vector of two numeric values. These values are held stable for the
variable that is named in `scenname`.

``` r
fdif2 <- mnl_fd_ova(model = mod1,
                    data = ml,
                    xvari = "math",
                    by = 1,
                    scenname = "female2",
                    scenvalues = c(0,1),
                    nsim = 100)
```

As before, the function returns a list, including a data set that can be
used to plot the differences.

``` r
fdif2$plotdata_fd %>% head()
#> # A tibble: 6 x 5
#>    math prog2       mean  lower  upper
#>   <dbl> <fct>      <dbl>  <dbl>  <dbl>
#> 1    33 academic -0.0347 -0.129 0.0438
#> 2    34 academic -0.0368 -0.132 0.0483
#> 3    35 academic -0.0389 -0.136 0.0531
#> 4    36 academic -0.0411 -0.140 0.0577
#> 5    37 academic -0.0433 -0.143 0.0613
#> 6    38 academic -0.0454 -0.147 0.0638
```

Since the function calls the `mnl_pred_ova()` function internally, it
also returns the output of the two predictions in the list element
`Prediction1` and `Prediction2`. The plot data for the predictions is
already bound together row wise to easily plot the predicted
probabilities.

``` r
ggplot(data = fdif2$plotdata, aes(x = math, y = mean,
                                ymin = lower, ymax = upper,
                                linetype = as.factor(female2))) +
  geom_ribbon(alpha = 0.1) +
  geom_line() +
  facet_grid(prog2 ~., scales = "free_y") +
  scale_y_continuous(labels = percent_format(accuracy = 1)) + # % labels
  scale_linetype_discrete(name = "Female") +
  theme_bw() +
  labs(y = "Predicted probabilities",
       x = "Math score") # Always label your axes ;)
```

![](man/figures/README-prediction_plot2-1.png)<!-- -->

As we can see, the differences between `female` and not-`female` are
minimal. So let’s take a look at the differences:

``` r
ggplot(data = fdif2$plotdata_fd, aes(x = math, y = mean,
                                  ymin = lower, ymax = upper)) +
  geom_ribbon(alpha = 0.1) +
  geom_line() +
  geom_hline(yintercept = 0) +
  facet_grid(prog2 ~., scales = "free_y") +
  scale_y_continuous(labels = percent_format(accuracy = 1)) + # % labels
  theme_bw() +
  labs(y = "Predicted probabilities",
       x = "Math score") # Always label your axes ;)
```

![](man/figures/README-first_differences_plot-1.png)<!-- -->

We can see that the differences are in fact minimal and at no point
statistically significant from 0.

## Conclusion

Multinomial logit models are important to model nominal choices. They
are restricted however by being in need of a baseline category.
Additionally, the log-character of the estimates makes it difficult to
interpret them in meaningful ways. Predicting probabilities for all
choices for scenarios, based on the observed data provides much more
insight. The functions of this package provide easy to use functions
that return data that can be used to plot predicted probabilities. The
function uses a model from the `multinom()` function and uses the
observed value approach and a supplied scenario to predict values over
the range of fitting values. The functions simulate sampling
distributions and therefore provide meaningful confidence intervals.
`mnl_pred_ova()` can be used to predict probabilities for a certain
scenario. `mnl_fd_ova()` can be used to predict probabilities for two
scenarios and their first differences.

## Acknowledgment

My code is inspired by the method courses in the [Political Science
master’s program at the University of
Mannheim](https://www.sowi.uni-mannheim.de/en/academics/prospective-students/ma-in-political-science/)(cool
place, check it out\!). The skeleton of the code is based on a tutorial
taught by [Marcel Neunhoeffer](https://www.marcel-neunhoeffer.com/)
(lecture: “Advanced Quantitative Methods” by [Thomas
Gschwend](http://methods.sowi.uni-mannheim.de/thomas_gschwend/)).

## References

<div id="refs" class="references">

<div id="ref-king2000">

King, Gary, Michael Tomz, and Jason Wittenberg. 2000. “Making the Most
of Statistical Analyses: Improving Interpretation and Presentation.”
*American Journal of Political Science* 44 (2): 341–55.

</div>

</div>
