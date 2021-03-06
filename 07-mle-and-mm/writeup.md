07-mle-mm
================
Shuyang Lin
11/4/2021

# Introduction

To construct a model, we usually use Maximum Likelihood and Method of
Moments.In this blog, we would model (a) Glycohemoglobin and (b) Height
of adult females. The data will be from National Health and Nutrition
Examination Survey 2009-2010 (NHANES), available from the Hmisc package.
We will compare and contrast the two methods in addition to comparing
and contrasting the choice of underlying distribution.

The data is imported as below:

``` r
require(dplyr)
Hmisc::getHdata(nhgh)
d1 <- nhgh %>% 
  filter(sex == "female") %>% 
  filter(age >= 18) %>% 
  select(gh, ht) %>% 
  filter(1:n()<=1000)
```

# Method of Moments

First we will calculate the means and variances:

``` r
# calculate the mean and variance of the sample
xbar_gh <- mean(d1$gh)
xbar_ht <- mean(d1$ht)
var_gh <- var(d1$gh)
var_ht <- var(d1$ht)
```

Then, we could estimate the parameters based on Method of Moments to fit
the data: normal distribution, Gamma distribution, and Weibull
distribution.

## Estimate parameters

-   Normal distribution

If the data follows the normal distribution, then we could assume that
*X* ∼ *N*(*x**b**a**r*, *v**a**r*<sup>0.5</sup>).

Therefore, the parameters of the normal distribution will be the same as
the means and standard errors of the samples.

``` r
# normal distribution
(sd_gh <- sd(d1$gh))
```

\[1\] 1.052246

``` r
(sd_ht <- sd(d1$ht))
```

\[1\] 7.320161

-   Gamma distribution

For Gamma distribution, we could simply calculate the parameters with
the formulas:

<img src="https://render.githubusercontent.com/render/math?math=Shape = \\frac{xbar^2}{var}">

$$ Scale = \\frac{var^2}{xbar}$$

Therefore, the parameters of Gamma Distribution is:

``` r
# gamma distribution
(shape_gh_hat <- xbar_gh^2/var_gh)
```

\[1\] 29.59754

``` r
(scale_gh_hat <- var_gh/xbar_gh)
```

\[1\] 0.1934147

``` r
(shape_ht_hat <- xbar_ht^2/var_ht)
```

\[1\] 482.1886

``` r
(scale_ht_hat <- var_ht/xbar_ht)
```

\[1\] 0.333359

-   Weibull Distribution

Weibull Distribution is a continuous distribution with two parameters, a
scale parameter *λ* and a shape parameter *k*. It’s PDF is given as
below[1]:

$$
f(x; \\lambda, k) =
\\left\\{
\\begin{array}{lr}
\\frac{k}{\\lambda}(\\frac{x}{\\lambda})^{k-1}e^{-(\\frac{x}{\\lambda})^k} &  & x \\geq 0 \\\\
0 & & x &lt; 0
\\end{array}
\\right.
$$

We mainly use it to predict weathers, wind speed and other similar
random variables.

To estimate its parameters using Method of Moments, we need to form a
group of equations to solve them. We know that when
*X* ∼ *f*(*x*; *λ*, *k*) there are:

$$
\\left\\{
\\begin{array}{lr}
E(X) = \\lambda\\Gamma(1+\\frac{1}{k}) \\\\
Var(X) = \\lambda^2\[\\Gamma(1+\\frac{2}{k})-\\Gamma(1+\\frac{1}{k})^2\]
\\end{array}
\\right.
$$

Thus, we could use the sample mean and *k* to present , and try to solve
the second equation.

``` r
# Weibull distribution
lambda <- function(xbar, k){
  xbar / gamma(1 + 1/k)
}

var_weibull_equation <- function(xbar, k){
  lambda(xbar, k)^2 * (gamma(1+2/k) - gamma(1+1/k)^2)
}
```

We need to know if this variance function has monotonicity:

``` r
(
  ggplot(mapping = aes(x=1:100, y=var_weibull_equation(xbar_gh, 1:100))) + geom_line() + labs(title='Function of Variance of Weibull Distribution of Glycohemoglobin', x='k', y='Inferred Var') + geom_hline(yintercept=var_gh, color='red') + geom_text(x=80, y=var_gh+1, aes(label='Sample Var'), color='red') + theme_classic()
)
```

![](writeup_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
(
  ggplot(mapping = aes(x=10:100, y=var_weibull_equation(xbar_ht, 10:100))) + geom_line() + labs(title='Function of Variance of Weibull Distribution of Height', x='k', y='Inferred Var') + geom_hline(yintercept=var_ht, color='red') + geom_text(x=80, y=var_ht+10, aes(label='Sample Var'), color='red') + theme_classic()
)
```

![](writeup_files/figure-gfm/unnamed-chunk-6-2.png)<!-- -->

Obviously we could get the solutions with function uniroot():

``` r
(k_gh_hat <- uniroot(function(x){var_weibull_equation(xbar_gh, x)-var_gh}, c(1,100))$root)
```

\[1\] 6.353174

``` r
(lambda_gh_hat <- lambda(xbar_gh, k_gh_hat))
```

\[1\] 6.151308

``` r
(k_ht_hat <- uniroot(function(x){var_weibull_equation(xbar_ht, x)-var_ht}, c(1,100))$root)
```

\[1\] 27.45938

``` r
(lambda_ht_hat <- lambda(xbar_ht, k_ht_hat))
```

\[1\] 163.9807

## Estimate medians

As we have the parameters, we could use “q” functions to get the
estimated median.

``` r
# gh

cat("Median of Glycohemoglobin\n")

cat("Sample Median:",median(d1$gh),'\n')

cat("Normal Distribution:",(mnorm_gh <- qnorm(.5,xbar_gh,sd_gh)),'\n')

cat("Gamma Distribution:",(mgamma_gh <- qgamma(.5, shape=shape_gh_hat, scale=scale_gh_hat)),'\n')

cat("Weibull Distribution:",(mweibull_gh <- qweibull(.5, k_gh_hat, lambda_gh_hat)),'\n')

# ht
cat("\nMedian of Height\n")

cat("Sample Median:",median(d1$ht),'\n')

cat("Normal Distribution:",(mnorm_ht <- qnorm(.5,xbar_ht,sd_ht)),'\n')

cat("Gamma Distribution:",(mgamma_ht <- qgamma(.5, shape=shape_ht_hat, scale=scale_ht_hat)),'\n')

cat("Weibull Distribution:",(mweibull_ht <- qweibull(.5, k_ht_hat, lambda_ht_hat)),'\n')
```

    ## Median of Glycohemoglobin
    ## Sample Median: 5.5 
    ## Normal Distribution: 5.7246 
    ## Gamma Distribution: 5.660259 
    ## Weibull Distribution: 5.806483 
    ## 
    ## Median of Height
    ## Sample Median: 160.8 
    ## Normal Distribution: 160.7419 
    ## Gamma Distribution: 160.6308 
    ## Weibull Distribution: 161.8065

## Overlay estimated pdf onto histogram

First one for Glycohemoglobin:

``` r
(
  pdf_gh_mm <- ggplot() +
                geom_histogram(
                  d1,
                  mapping=aes(x=gh,y=..density..),
                  breaks=seq(2,15,0.1),
                  color='black',
                  fill='grey'
                ) +
    geom_vline(xintercept = median(d1$gh),alpha=.5)+
  # norm
                geom_point(
                  mapping = aes(
                    x=seq(2, 15, 0.01), 
                    y=dnorm(
                        seq(2,15,0.01),
                        mean=xbar_gh,
                        sd=var_gh^.5
                      ),
                    color='Normal'
                  ),
                  alpha=.5
                ) +
    geom_vline(xintercept = mnorm_gh,color='orange')+
  # gamma
                geom_point(
                  mapping = aes(
                    x=seq(2,15,0.01),
                    y=dgamma(
                        seq(2,15,0.01),
                        shape=shape_gh_hat,
                        scale=scale_gh_hat
                      ),
                    color='Gamma'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mgamma_gh,color='blue')+
  # weibull
                geom_point(
                  mapping=aes(
                    x=seq(2,15,0.01),
                    y=dweibull(
                      seq(2,15,0.01),
                      k_gh_hat,
                      lambda_gh_hat
                    ),
                    color='Weibull'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mweibull_gh,color='green')+
                scale_color_manual(
                  name='Model',
                  breaks=c('Normal','Gamma','Weibull'),
                  values=c('Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
                labs(
                  title = 'Histogram of Glycohemoglobin and Estimated PDF by MM',
                  x = 'Glycohemoglobin',
                  y = 'Density',
                  color = 'Model'
                ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

The empirical distribution is not axisymmetric, and the density is not
so matched. Thus, the normal distribution seems to fit not so well.
Gamma distribution shares the same peak with the histogram, it might be
better than the Weibull distribution.

``` r
(
  pdf_ht_mm <- ggplot() +
                geom_histogram(
                  d1,
                  mapping=aes(x=ht,y=..density..),
                  breaks=seq(130,200,2),
                  color='black',
                  fill='grey'
                ) + 
    geom_vline(xintercept = median(d1$ht),alpha=.5)+
  # norm
                geom_point(
                  mapping = aes(
                    x=seq(130, 200, 0.1), 
                    y=dnorm(seq(130, 200,0.1),
                    mean=xbar_ht,
                    sd=var_ht^.5
                    ),
                    color='Normal'
                  ),
                  alpha=.5
                ) +
    geom_vline(xintercept = mnorm_ht,color='orange',alpha=.5)+
  # gamma
                geom_point(
                  mapping = aes(
                    x=seq(130, 200, 0.1), 
                    y=dgamma(seq(130, 200,0.1),
                    shape=shape_ht_hat,
                    scale=scale_ht_hat
                    ),
                    color='Gamma'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mgamma_ht,color='blue',alpha=.5)+
  # weibull
                geom_point(
                  mapping=aes(
                    x=seq(130, 200, 0.1), 
                    y=dweibull(seq(130, 200,0.1),
                    shape=k_ht_hat,
                    scale=lambda_ht_hat
                    ),
                    color='Weibull'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mweibull_ht,color='green')+
                scale_color_manual(
                  name='Model',
                  breaks=c('Sample','Normal','Gamma','Weibull'),
                  values=c('Sample'='black','Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
                labs(
                  title = 'Histogram of Height and Estimated PDF by MM',
                  x = 'Height',
                  y = 'Density',
                  color = 'Model'
                ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

The normal distribution and gamma distribution fits quite well! We could
say that the Height is nearly following a normal distribution. Their
estimate median is so close.

## Overlay estimated CDF onto eCDF

Next we try drawing the CDF of the :

``` r
(
  cdf_gh_mm <- ggplot() +
                stat_ecdf(
                  d1,
                  mapping=aes(x=gh),
                  color='black'
                ) + 
    geom_vline(xintercept = median(d1$gh),alpha=.5)+
  # norm
                geom_line(
                  mapping = aes(
                    x=seq(2, 15, 0.01), 
                    y=pnorm(
                        seq(2,15,0.01),
                        mean=xbar_gh,
                        sd=var_gh^.5
                      ),
                    color='Normal'
                  ),
                  alpha=.5
                ) +
    geom_vline(xintercept = mnorm_gh,color='orange')+
  # gamma
                geom_line(
                  mapping = aes(
                    x=seq(2,15,0.01),
                    y=pgamma(
                        seq(2,15,0.01),
                        shape=shape_gh_hat,
                        scale=scale_gh_hat
                      ),
                    color='Gamma'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mgamma_gh,color='blue')+
  # weibull
                geom_line(
                  mapping=aes(
                    x=seq(2,15,0.01),
                    y=pweibull(
                      seq(2,15,0.01),
                      k_gh_hat,
                      lambda_gh_hat
                    ),
                    color='Weibull'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mweibull_gh,color='green')+
                scale_color_manual(
                  name='Model',
                  breaks=c('Sample','Normal','Gamma','Weibull'),
                  values=c('Sample'='black','Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
                labs(
                  title = 'ECDF of Glycohemoglobin and Estimated CDF by MM',
                  x = 'Glycohemoglobin',
                  y = 'Cumulative Posibility',
                  color = 'Model'
                ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

We could still see a gap between the model and the empirical data. This
deliver the same info that the PDF could do.

Next we do the CDF plot of Height:

``` r
(
  cdf_ht_mm <- ggplot() +
                stat_ecdf(
                  d1,
                  mapping=aes(x=ht),
                ) + 
    geom_vline(xintercept = median(d1$ht),alpha=.5)+
  # norm
                geom_line(
                  mapping = aes(
                    x=seq(130, 200, 0.1), 
                    y=pnorm(seq(130, 200,0.1),
                    mean=xbar_ht,
                    sd=var_ht^.5
                    ),
                    color='Normal'
                  ),
                  alpha=.5
                ) +
    geom_vline(xintercept = mnorm_ht,color='orange',alpha=.5)+
  # gamma
                geom_line(
                  mapping = aes(
                    x=seq(130, 200, 0.1), 
                    y=pgamma(seq(130, 200,0.1),
                    shape=shape_ht_hat,
                    scale=scale_ht_hat
                    ),
                    color='Gamma'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mgamma_ht,color='blue',alpha=.5)+
  # weibull
                geom_line(
                  mapping=aes(
                    x=seq(130, 200, 0.1), 
                    y=pweibull(seq(130, 200,0.1),
                    shape=k_ht_hat,
                    scale=lambda_ht_hat
                    ),
                    color='Weibull'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mweibull_ht,color='green')+
                scale_color_manual(
                  name='Model',
                  breaks=c('Sample','Normal','Gamma','Weibull'),
                  values=c('Sample'='black','Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
                labs(
                  title = 'ECDF of Height and Estimated CDF by MM',
                  x = 'Height',
                  y = 'Cumulative Probability',
                  color = 'Model'
                ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

The models except Weibull distribution fit really well! Same conclusion
as previous one by PDF.

## QQ plot (sample vs estimated dist)

We need to generate all the quantiles:

``` r
qqdata_gh_mm <- data.frame(seq(0.05,0.95,0.02))
colnames(qqdata_gh_mm) <- c('p')
qqdata_gh_mm <- qqdata_gh_mm %>% mutate(
  sample = quantile(d1$gh, p),
  norm = qnorm(p, xbar_gh, sd_gh),
  gamma = qgamma(p, shape_gh_hat,scale=scale_gh_hat),
  weibull = qweibull(p, shape=k_gh_hat,scale=lambda_gh_hat)
)

qqdata_ht_mm <- data.frame(seq(0.05,0.95,0.02))
colnames(qqdata_ht_mm) <- c('p')
qqdata_ht_mm <- qqdata_ht_mm %>% mutate(
  sample = quantile(d1$ht, p),
  norm = qnorm(p, xbar_ht, sd_ht),
  gamma = qgamma(p, shape_ht_hat,scale=scale_ht_hat),
  weibull = qweibull(p, shape=k_ht_hat,scale=lambda_ht_hat)
)
```

Then we draw the qqplot of Glycohemoglobin:

``` r
(
  qqplot_gh_mm <- ggplot(qqdata_gh_mm, aes(x=sample)) +
    geom_point(
      aes(y=norm, color='Normal'),alpha=.5
    )+
    geom_point(
      aes(y=gamma, color='Gamma'),alpha=.5
    ) +
    geom_point(
      aes(y=weibull, color='Weibull'),alpha=.5
    ) +
    geom_abline(slope=1,intercept=0)+
    scale_color_manual(
      name='Model',
      breaks=c('Normal','Gamma','Weibull'),
      values=c('Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
    labs(
      title = 'QQ Plot of Glycohemoglobin by MM',
      x = 'Sample Quantile',
      y = 'Theoretical Quantile',
      color = 'Model'
    ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

We may see that all the models show deviation from the line *y* = *x*,
which indicates that the model still needs improvement.

Next qq plot for the Height:

``` r
(
  qqplot_ht_mm <- ggplot(qqdata_ht_mm, aes(x=sample)) +
    geom_point(
      aes(y=norm, color='Normal'),alpha=.5
    )+
    geom_point(
      aes(y=gamma, color='Gamma'),alpha=.5
    ) +
    geom_point(
      aes(y=weibull, color='Weibull'),alpha=.5
    ) +
    geom_abline(slope=1,intercept=0)+
    scale_color_manual(
      name='Model',
      breaks=c('Normal','Gamma','Weibull'),
      values=c('Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
    labs(
      title = 'QQ Plot of Height by MM',
      x = 'Sample Quantile',
      y = 'Theoretical Quantile',
      color = 'Model'
    ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

Weibull distribution fits not well. The other two is good.

# Maximum Likelihood

Now we come to the Maximum Likelihood. To use this method, we need to
create a likelihood function with known values and unknown parameters.
The likelihood function is created by the following formula:

$$ f(x\_0, \\theta) = \\prod\_i^n f(X=x\_i, \\theta) $$

We need to maximize the production of the probability, which means we
need to get the point that $f^\`(x\_0,\\theta) = 0$. We have mle()
function in R to solve this problem. It needs the production of the
probability log-transformed.

## Estimate parameters

-   Normal Distribution

``` r
# normal
norm_ll_gh <- function(mean, sd) {
  p <- dnorm(d1$gh, mean, sd,log=T)
  -sum(p)
}

norm_mle_gh <- coef(mle(
  norm_ll_gh,
  start=list(mean=xbar_gh,sd=sd_gh),
  method='L-BFGS-B',
  lower=c(0,0.01)
))
cat("MLE of normal distribution of Glycohemoglobin\n")
cat("Mean:",norm_mle_gh[1],'\n')
cat("Standard Deviation:",norm_mle_gh[2],'\n\n')


norm_ll_ht <- function(mean, sd) {
  p <- dnorm(d1$ht, mean, sd,log=T)
  -sum(p)
}

norm_mle_ht <- coef(mle(
  norm_ll_ht,
  start=list(mean=xbar_ht,sd=sd_ht),
  method='L-BFGS-B',
  lower=c(0,0.01)
))
cat("MLE of normal distribution of Height\n")
cat("Mean:",norm_mle_ht[1],'\n')
cat("Standard Deviation:",norm_mle_ht[2],'\n')
```

    ## MLE of normal distribution of Glycohemoglobin
    ## Mean: 5.7246 
    ## Standard Deviation: 1.051721 
    ## 
    ## MLE of normal distribution of Height
    ## Mean: 160.7419 
    ## Standard Deviation: 7.3165

-   Gamma Distribution

``` r
# gamma
gamma_ll_gh <- function(shape, scale) {
  p <- dgamma(d1$gh, shape=shape, scale=scale,log=T)
  -sum(p)
}

gamma_mle_gh <- coef(mle(
  gamma_ll_gh,
  start=list(shape=shape_gh_hat,scale=scale_gh_hat),
  method='L-BFGS-B',
  lower=c(0,0.01)
))
cat("MLE of gamma distribution of Glycohemoglobin\n")
cat("Shape:",gamma_mle_gh[1],'\n')
cat("Scale:",gamma_mle_gh[2],'\n\n')


gamma_ll_ht <- function(shape, scale) {
  p <- dgamma(d1$ht, shape=shape, scale=scale,log=T)
  -sum(p)
}

gamma_mle_ht <- coef(mle(
  gamma_ll_ht,
  start=list(shape=shape_ht_hat,scale=scale_ht_hat),
  method='L-BFGS-B',
  lower=c(0,0.01)
))
cat("MLE of gamma distribution of Height\n")
cat("Shape:",gamma_mle_ht[1],'\n')
cat("Scale:",gamma_mle_ht[2],'\n')
```

    ## MLE of gamma distribution of Glycohemoglobin
    ## Shape: 40.7065 
    ## Scale: 0.1406358 
    ## 
    ## MLE of gamma distribution of Height
    ## Shape: 482.1886 
    ## Scale: 0.333359

We could see that the MLE and MM results of Gamma Distribution of
Glycohemoglobin differ from each other. It needs further discussion.

-   Weibull Distribution

``` r
# gamma
weibull_ll_gh <- function(k, lambda) {
  p <- dweibull(d1$gh, shape=k, scale=lambda,log=T)
  -sum(p)
}

weibull_mle_gh <- coef(mle(
  weibull_ll_gh,
  start=list(k=k_gh_hat,lambda=lambda_gh_hat),
  method='L-BFGS-B',
  lower=c(0,0.01)
))
cat("MLE of Weibull distribution of Glycohemoglobin\n")
cat("k:",weibull_mle_gh[1],'\n')
cat("lambda:",weibull_mle_gh[2],'\n\n')


weibull_ll_ht <- function(k, lambda) {
  p <- dweibull(d1$ht, shape=k, scale=lambda,log=T)
  -sum(p)
}

weibull_mle_ht <- coef(mle(
  weibull_ll_ht,
  start=list(k=k_ht_hat,lambda=lambda_ht_hat),
  method='L-BFGS-B',
  lower=c(0,0.01)
))
cat("MLE of Weibull distribution of Height\n")
cat("k:",weibull_mle_ht[1],'\n')
cat("lambda:",weibull_mle_ht[2],'\n')
```

    ## MLE of Weibull distribution of Glycohemoglobin
    ## k: 4.125254 
    ## lambda: 6.173885 
    ## 
    ## MLE of Weibull distribution of Height
    ## k: 21.85398 
    ## lambda: 164.2472

They are not the same as the result of MM.

## Estimate medians

As we have the parameters, we could use “q” functions to get the
estimated median too, as we’ve done before.

``` r
# gh

cat("Median of Glycohemoglobin\n")

cat("Sample Median:",median(d1$gh),'\n')

cat("Normal Distribution:",(mnorm_gh_mle <- qnorm(.5,norm_mle_gh[1],norm_mle_gh[2])),'\n')

cat("Gamma Distribution:",(mgamma_gh_mle <- qgamma(.5, shape=gamma_mle_gh[1], scale=gamma_mle_gh[2])),'\n')

cat("Weibull Distribution:",(mweibull_gh_mle <- qweibull(.5, weibull_mle_gh[1], weibull_mle_gh[2])),'\n')

# ht
cat("\nMedian of Height\n")

cat("Sample Median:",median(d1$ht),'\n')

cat("Normal Distribution:",(mnorm_ht_mle <- qnorm(.5,norm_mle_ht[1],norm_mle_ht[2])),'\n')

cat("Gamma Distribution:",(mgamma_ht_mle <- qgamma(.5, shape=gamma_mle_ht[1], scale=gamma_mle_ht[2])),'\n')

cat("Weibull Distribution:",(mweibull_ht_mle <- qweibull(.5, weibull_mle_ht[1], weibull_mle_ht[2])),'\n')
```

    ## Median of Glycohemoglobin
    ## Sample Median: 5.5 
    ## Normal Distribution: 5.7246 
    ## Gamma Distribution: 5.677983 
    ## Weibull Distribution: 5.64902 
    ## 
    ## Median of Height
    ## Sample Median: 160.8 
    ## Normal Distribution: 160.7419 
    ## Gamma Distribution: 160.6308 
    ## Weibull Distribution: 161.5156

## Overlay estimated pdf onto histogram

First one for Glycohemoglobin:

``` r
(
  pdf_gh_mle <- ggplot() +
                geom_histogram(
                  d1,
                  mapping=aes(x=gh,y=..density..),
                  breaks=seq(2,15,0.1),
                  color='black',
                  fill='grey'
                ) +
    geom_vline(xintercept = median(d1$gh),alpha=.5)+
  # norm
                geom_point(
                  mapping = aes(
                    x=seq(2, 15, 0.01), 
                    y=dnorm(
                        seq(2,15,0.01),
                        mean=norm_mle_gh[1],
                        sd=norm_mle_gh[2]
                      ),
                    color='Normal'
                  ),
                  alpha=.5
                ) +
    geom_vline(xintercept = mnorm_gh_mle,color='orange')+
  # gamma
                geom_point(
                  mapping = aes(
                    x=seq(2,15,0.01),
                    y=dgamma(
                        seq(2,15,0.01),
                        shape=gamma_mle_gh[1],
                        scale=gamma_mle_gh[2]
                      ),
                    color='Gamma'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mgamma_gh_mle,color='blue')+
  # weibull
                geom_point(
                  mapping=aes(
                    x=seq(2,15,0.01),
                    y=dweibull(
                      seq(2,15,0.01),
                      weibull_mle_gh[1],
                      weibull_mle_gh[2]
                    ),
                    color='Weibull'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mweibull_gh_mle,color='green')+
                scale_color_manual(
                  name='Model',
                  breaks=c('Normal','Gamma','Weibull'),
                  values=c('Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
                labs(
                  title = 'Histogram of Glycohemoglobin and Estimated PDF by MLE',
                  x = 'Glycohemoglobin',
                  y = 'Density',
                  color = 'Model'
                ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-20-1.png)<!-- -->

The empirical distribution is not axisymmetric, and the density is not
so matched. Thus, the normal distribution seems to fit not so well
either. Gamma distribution shares the same peak with the weibull
distribution.

``` r
(
  pdf_ht_mle <- ggplot() +
                geom_histogram(
                  d1,
                  mapping=aes(x=ht,y=..density..),
                  breaks=seq(130,200,2),
                  color='black',
                  fill='grey'
                ) + 
    geom_vline(xintercept = median(d1$ht),alpha=.5)+
  # norm
                geom_point(
                  mapping = aes(
                    x=seq(130, 200, 0.1), 
                    y=dnorm(seq(130, 200,0.1),
                    mean=norm_mle_ht[1],
                    sd=norm_mle_ht[2]
                    ),
                    color='Normal'
                  ),
                  alpha=.5
                ) +
    geom_vline(xintercept = mnorm_ht_mle,color='orange',alpha=.5)+
  # gamma
                geom_point(
                  mapping = aes(
                    x=seq(130, 200, 0.1), 
                    y=dgamma(seq(130, 200,0.1),
                    shape=gamma_mle_ht[1],
                    scale=gamma_mle_ht[2]
                    ),
                    color='Gamma'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mgamma_ht_mle,color='blue',alpha=.5)+
  # weibull
                geom_point(
                  mapping=aes(
                    x=seq(130, 200, 0.1), 
                    y=dweibull(seq(130, 200,0.1),
                    shape=weibull_mle_ht[1],
                    scale=weibull_mle_ht[2]
                    ),
                    color='Weibull'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mweibull_ht_mle,color='green')+
                scale_color_manual(
                  name='Model',
                  breaks=c('Sample','Normal','Gamma','Weibull'),
                  values=c('Sample'='black','Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
                labs(
                  title = 'Histogram of Height and Estimated PDF by MLE',
                  x = 'Height',
                  y = 'Density',
                  color = 'Model'
                ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

The normal distribution and gamma distribution fits quite well! We could
say that the Height is nearly following a normal distribution or gamma
distribution too. Their estimate median is so close.

## Overlay estimated CDF onto eCDF

Next we try drawing the CDF of the :

``` r
(
  cdf_gh_mle <- ggplot() +
                stat_ecdf(
                  d1,
                  mapping=aes(x=gh),
                  color='black'
                ) + 
    geom_vline(xintercept = median(d1$gh),alpha=.5)+
  # norm
                geom_line(
                  mapping = aes(
                    x=seq(2, 15, 0.01), 
                    y=pnorm(
                        seq(2,15,0.01),
                        mean=norm_mle_gh[1],
                        sd=norm_mle_gh[2]
                      ),
                    color='Normal'
                  ),
                  alpha=.5
                ) +
    geom_vline(xintercept = mnorm_gh_mle,color='orange')+
  # gamma
                geom_line(
                  mapping = aes(
                    x=seq(2,15,0.01),
                    y=pgamma(
                        seq(2,15,0.01),
                        shape=gamma_mle_gh[1],
                        scale=gamma_mle_gh[2]
                      ),
                    color='Gamma'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mgamma_gh_mle,color='blue')+
  # weibull
                geom_line(
                  mapping=aes(
                    x=seq(2,15,0.01),
                    y=pweibull(
                      seq(2,15,0.01),
                      weibull_mle_gh[1],
                      weibull_mle_gh[2]
                    ),
                    color='Weibull'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mweibull_gh_mle,color='green')+
                scale_color_manual(
                  name='Model',
                  breaks=c('Sample','Normal','Gamma','Weibull'),
                  values=c('Sample'='black','Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
                labs(
                  title = 'ECDF of Glycohemoglobin and Estimated CDF by MLE',
                  x = 'Glycohemoglobin',
                  y = 'Cumulative Posibility',
                  color = 'Model'
                ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-22-1.png)<!-- -->

We could still see a gap between the models and the empirical data. This
deliver the same info that the PDF could do. Gamma distribution is the
closest one.

Next we do the CDF plot of Height:

``` r
(
  cdf_ht_mle <- ggplot() +
                stat_ecdf(
                  d1,
                  mapping=aes(x=ht),
                ) + 
    geom_vline(xintercept = median(d1$ht),alpha=.5)+
  # norm
                geom_line(
                  mapping = aes(
                    x=seq(130, 200, 0.1), 
                    y=pnorm(seq(130, 200,0.1),
                    mean=norm_mle_ht[1],
                    sd=norm_mle_ht[2]
                    ),
                    color='Normal'
                  ),
                  alpha=.5
                ) +
    geom_vline(xintercept = mnorm_ht_mle,color='orange',alpha=.5)+
  # gamma
                geom_line(
                  mapping = aes(
                    x=seq(130, 200, 0.1), 
                    y=pgamma(seq(130, 200,0.1),
                    shape=gamma_mle_ht[1],
                    scale=gamma_mle_ht[2]
                    ),
                    color='Gamma'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mgamma_ht_mle,color='blue',alpha=.5)+
  # weibull
                geom_line(
                  mapping=aes(
                    x=seq(130, 200, 0.1), 
                    y=pweibull(seq(130, 200,0.1),
                    shape=weibull_mle_ht[1],
                    scale=weibull_mle_ht[2]
                    ),
                    color='Weibull'
                  ),
                  alpha=.5
                )+
    geom_vline(xintercept = mweibull_ht_mle,color='green')+
                scale_color_manual(
                  name='Model',
                  breaks=c('Sample','Normal','Gamma','Weibull'),
                  values=c('Sample'='black','Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
                labs(
                  title = 'ECDF of Height and Estimated CDF by MLE',
                  x = 'Height',
                  y = 'Cumulative Probability',
                  color = 'Model'
                ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-23-1.png)<!-- -->

The models except Weibull distribution fit really well too! Same
conclusion as previous one by PDF.

## QQ plot (sample vs estimated dist)

We need to generate all the quantiles:

``` r
qqdata_gh_mle <- data.frame(seq(0.05,0.95,0.02))
colnames(qqdata_gh_mle) <- c('p')
qqdata_gh_mle <- qqdata_gh_mle %>% mutate(
  sample = quantile(d1$gh, p),
  norm = qnorm(p, norm_mle_gh[1], norm_mle_gh[2]),
  gamma = qgamma(p, gamma_mle_gh[1],scale=gamma_mle_gh[2]),
  weibull = qweibull(p, shape=weibull_mle_gh[1],scale=weibull_mle_gh[2])
)

qqdata_ht_mle <- data.frame(seq(0.05,0.95,0.02))
colnames(qqdata_ht_mle) <- c('p')
qqdata_ht_mle <- qqdata_ht_mle %>% mutate(
  sample = quantile(d1$ht, p),
  norm = qnorm(p, norm_mle_ht[1], norm_mle_ht[2]),
  gamma = qgamma(p, gamma_mle_ht[1],scale=gamma_mle_ht[2]),
  weibull = qweibull(p, shape=weibull_mle_ht[1],scale=weibull_mle_ht[2])
)
```

Then we draw the qqplot of Glycohemoglobin:

``` r
(
  qqplot_gh_mle <- ggplot(qqdata_gh_mle, aes(x=sample)) +
    geom_point(
      aes(y=norm, color='Normal'),alpha=.5
    )+
    geom_point(
      aes(y=gamma, color='Gamma'),alpha=.5
    ) +
    geom_point(
      aes(y=weibull, color='Weibull'),alpha=.5
    ) +
    geom_abline(slope=1,intercept=0)+
    scale_color_manual(
      name='Model',
      breaks=c('Normal','Gamma','Weibull'),
      values=c('Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
    labs(
      title = 'QQ Plot of Glycohemoglobin by MLE',
      x = 'Sample Quantile',
      y = 'Theoretical Quantile',
      color = 'Model'
    ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-25-1.png)<!-- -->

We may see that all the models show deviation from the line *y* = *x*,
which indicates that the model still needs improvement.

Next qq plot for the Height:

``` r
(
  qqplot_ht_mle <- ggplot(qqdata_ht_mle, aes(x=sample)) +
    geom_point(
      aes(y=norm, color='Normal'),alpha=.5
    )+
    geom_point(
      aes(y=gamma, color='Gamma'),alpha=.5
    ) +
    geom_point(
      aes(y=weibull, color='Weibull'),alpha=.5
    ) +
    geom_abline(slope=1,intercept=0)+
    scale_color_manual(
      name='Model',
      breaks=c('Normal','Gamma','Weibull'),
      values=c('Normal'='orange','Gamma'='blue','Weibull'='green')
                )+
    labs(
      title = 'QQ Plot of Height by MLE',
      x = 'Sample Quantile',
      y = 'Theoretical Quantile',
      color = 'Model'
    ) + theme_classic() + theme(legend.position='right')
)
```

![](writeup_files/figure-gfm/unnamed-chunk-26-1.png)<!-- -->

Weibull distribution fits not well. The other two is good.

# Conclusion

The distribution of female adults’ Glycohemoglobin doesn’t follow any of
these distributions. The PDF, CDF and QQ plots show gap between the
models and the sample.

The distribution of female adults’ Height seems to follow either normal
or gamma distribution. The PDF, CDF, and QQ plots show nearly the same
points. It doesn’t fit in Weibull distribution.

MM and MLE gives different estimated parameters, while normal
distribution shows nearly same mean and standard deviation. This could
be proved by the definition of MLE. Further proof will not be discussed
in this blog.

# Reference Links

[1] [Weibull
Distribution](https://en.wikipedia.org/wiki/Weibull_distribution)
