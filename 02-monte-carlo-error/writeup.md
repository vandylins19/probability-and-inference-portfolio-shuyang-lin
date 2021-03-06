Monte Carlo Error
================
Shuyang Lin
9/8/2021

# Introduction

In statistics, simulation is a test or experiment based on a sample to
simulate the population. Specifically, we use several methods to
complete a task called parameter estimation, which requires sample
generation, parameter calculation of the sample like the mean or the
variance, and estimation of the same type of parameters of the
population. Because the sample will not be the same as the population,
some errors will occur between the estimated value and the actual value,
which could not be avoided but could be reduced. One basic but efficient
method is the Monte Carlo simulation. In this blog, we will use this
method to show what factors will affect the error.

## Background

When measuring the error between the estimating value and the actual
value, we introduce two basic concept, the absolute error and the
relative error.

### Two types of errors

**Absolute Error** is the difference between the inferred value of the
index we estimate and its actual value. That means, we do not change the
scale of the value. Instead, we minus it immediately and take the
absolute value.

-   For example, we got an estimating value of 2%, and if the actual
    value is 1%, we have an absolute error of 1%. That seems to be
    small.

**Relative Error** is obtained by dividing the absolute error by the
actual value. It is used to figure out how large the absolute error is.

-   Still, with that example, since the absolute error is 1% and the
    actual value is 1%, the relative error is 1, or in percentage
    format, 100%, which means we deviate from the truth by 100%. That is
    far beyond acceptable.

### Monte Carlo simulation

**Monte Carlo Simulation** is a model used to predict the probability of
a random variable by repeating sampling and calculating the
frequency[1].

As the sampling is repeated sufficient times, the frequency will get
closer and closer to the actual probability. Its proof is given by the
Law of Large Numbers and proved by Chebyshev???s inequality[2].

Monte Carlo method usually contains particular steps[3]:

-   Define the domain

-   Generate inputs randomly from a probability distribution

-   Perform a deterministic computation on the inputs

-   Aggregate the results

The very first usage of Monte Carlo method is to estimate the
probability of a defined random event. As we repeat enough times, the
frequency will be close enough to estimate the actual probability. How
many times should we loop? That is a question able to explore if we
repeat a random distribution whose probability we know. In this article,
we will use a basic distribution, binomial distribution.

### Binominal distribution

**Binomial Distribution** is a random distribution with parameters n and
p.??**n** stands for the number of independent experiments we do, and
**p** stands for the probability that this binary experiment (or called
Bernoulli trial) has a positive result.

**Binary** here means the event has only two results, either positive or
negative. For example, flipping a coin. When flipping a coin, we have a
50% chance to get a positive outcome (head) and a 50% to get a negative
one (tail).

As we have a large n, the average frequency of the binomial distribution
experiment will be similar to the actual probability. That is what we
are talking about in this article.

# Methods

Here, to show the relation between experiment size and the error, we
will perform a 14 X 5 factorial experiment simulation. The size will be
the powers of 2, like 4, 16, 1024, until 32768, at a total of 14
numbers. The probability of the **Binomial Distribution** we talk about
previously will be set as 0.01, 0.05, 0.1, 0.25 and 0.5. Therefore we
got 70 combinations between each size and each probability. Then, we
generate each binomial experiment based on these parameter pairs by
100000 times to show a steady outcome of the errors.

### Code for generating the experiment

Firstly, we set the seed to be a certain number so everytime we open
this markdown file, we get the same result.

``` r
# random seed set
set.seed(1)
```

Here is the library we use:

``` r
library(tidyverse)
library(ggplot2)
```

Then here???s the code:

``` r
# generate the experiment
test_size <- 100000 # parameter for the rbinom function

# generate the vector for sizes
size <- NA 
for(i in 2:15) {
  size[i-1] <- 2**i
}

# generate the vector for probabilities
prob <- c(0.01,0.05,0.1,0.2,0.5)

# generate the empty data frame for recording the inputs and outputs
data <- data.frame(matrix(NA,70,4))
colnames(data) <- c("size","p", "abs_error", "rel_error")

# set a count for experiment and start generation
count <- 1
for(i in 1:14) {
  for(j in 1:5) {
    data[count,1] <- size[i]
    data[count,2] <- prob[j]
    temp <- rbinom(test_size,size[i],prob[j]) #further explanation for it below
    data[count, 3] <- mean(abs(temp/size[i] - prob[j]))
    data[count, 4] <- data[count,3]/prob[j]
    count <- count + 1;
  }
}

data$p <- factor(data$p)
```

Highlight the **rbinom** function we use. It is a basic R function for
generate a series of binomial experiments. The second parameter is **n**
we talk about previously and the third is **p**. Here the first
parameter stands for the number of independent binomial experiments we
do.

**Be aware that**, the number of independent binomial experiments and
the number of independent binary experiments in each binomial experiment
is **different**. We do several binary test to complete a binomial test
with a certain size, and we do several binomial tests to estimate **the
average error** differentiating by **the amount of the size**.

After running this code, we will get a data frame named ???data???, which
has 70 rows for each data point, and 4 columns as following statements:

-   Size, the size we set in each binomial experiment.

-   p, the actual probability we set in each binomial experiment.

-   abs\_error, the average absolute error of each pair of sizes and
    probabilities.

-   rel\_error, the relative error calculated based on the abs\_error.

# Results

To see the result more clearly, we will draw some charts:

First one is for the absolute error:

``` r
(g1 <- ggplot(data, aes(x=size,y=abs_error, factor=p)) + geom_line(aes(color=p)) + geom_point(aes(color=p)) + labs(title = 'Absolute error vs. size', y='Absolute error'))
```

![](writeup_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

It is hard to see the detail, so we change the x to log() scale by
applying log function:

``` r
(
  g1 <- ggplot(data,
               aes(
                 x= log(size,2),
                 y=abs_error,
                 factor=p
                 )
        ) +
        geom_line(aes(color=p)) + 
        geom_point(aes(color=p)) + 
        labs(
          title = 'Absolute error vs. size(in log2 scale)',
          x = 'size in log2 scale',
          y = 'Absolute error'
        )
)
```

![](writeup_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

Now we see that the absolute errors first vary among different p.??The
higher the p, the higher the absolute error with the same size.

To see further information, we change the absolute error to log2 scale
too:

``` r
(
  g1_v2 <- ggplot(data,
               aes(
                 x= log(size,2),
                 y= log(abs_error,2),
                 factor=p
                 )
        ) +
        geom_line(aes(color=p)) + 
        geom_point(aes(color=p)) + 
        labs(
          title = 'Absolute error vs. size (both in log2 scale)',
          x = 'size in log2 scale',
          y = 'Absolute error in log2 scale'
        )
)
```

![](writeup_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

We find them similar to linear reduction. As long as the experiment is
valid, we may obtain a hypothesis:

### The larger the experiment size is, the less absolute error we will have.

Now we do the same two charts for relative error:

``` r
(
  g2 <- ggplot(data,
               aes(
                 x= log(size,2),
                 y= rel_error,
                 factor=p
                 )
        ) +
        geom_line(aes(color=p)) + 
        geom_point(aes(color=p)) + 
        labs(
          title = 'Relative error vs. size(in log2 scale)',
          x = 'size in log2 scale',
          y = 'Relative error'
        )
)
```

![](writeup_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

It shares the same trend with chart 1 we previously discussed. However,
we could see that **the less the p, the higher the relative error**. We
may assume that a small value of p will make it easier to **???shake???**, I
mean, differentiating from the actual value.

To see further information, we still change the relative error to log2
scale too:

``` r
(
  g2_v2 <- ggplot(data,
               aes(
                 x= log(size,2),
                 y= log(rel_error,2),
                 factor=p
                 )
        ) +
        geom_line(aes(color=p)) + 
        geom_point(aes(color=p)) + 
        labs(
          title = 'Relative error vs. size (both in log2 scale)',
          x = 'size in log2 scale',
          y = 'Relative error in log2 scale'
        )
)
```

![](writeup_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

It still shares the same trend with absolute error???s and **the less the
p, the higher the relative error** still works for this chart.

# Conclusions

By doing these experiments, we use the action to prove or being strict,
simulate that:

### The larger size we generate, the lower error we will get.

This common sense is used quite widely in statistics, machine learning,
and deep learning. Repeating usually makes the error small.

And we know that the absolute error sometimes won???t be the best
measurement when comparing between different p, and relative error will
be a good index for improving a certain model. But do remember, each
unit of the absolute error in the real world means certainly each 1% of
the probability increased!

# Refer links

[1] <https://en.wikipedia.org/wiki/Monte_Carlo_method>

[2] <https://en.wikipedia.org/wiki/Law_of_large_numbers#Proof_using_Chebyshev's_inequality_assuming_finite_variance>

[3] <https://en.wikipedia.org/wiki/Monte_Carlo_method>
