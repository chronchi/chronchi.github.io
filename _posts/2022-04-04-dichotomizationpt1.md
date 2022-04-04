---
layout: post
title: "Dichotomization and statistical testing: part 1"
categories:
    - statistics
    - null hypothesis
    - dichotomization
description: First part of blog posts that describe the effects of dichotomization
---
    
# Intro

Dichotomization is something widespread in biological sciences. Given a 
continuous variable, a dichotomization process is performed in order to 
make its interpreation easier. Usually this is done in the context of survival
analysis, an example is gene expression levels that are divided into low
and high values or different quantiles. 

This process might be problematic, as we might be inducing effects 
that are non existent when thresholding and dichotomizing continuous variables.
In this blog post I show how associations can arise from randomly 
generated data, just by expectation. 

# Generating the data

The first step is to generate the random data. A random outcome 
vector is first generated and will serve as the base to the associations.
Then, through n iterations, a random vector of covariates is created.
By the definition of p-values, we expect some of these covariates
to be associated with our outcome. The covariates are then dichotomized 
using the median as a threshold. 

# Loading necessary packages


```r
library(dplyr)
library(tidyr)
library(janitor)
library(broom)

library(ggplot2)
options(bitmapType = "cairo")

library(survsim)
library(survival)
```

# Linear regression and association with random covariates 

## Simulation

A total of 5000 simulations are run for a total 
of 100 samples. First the random outcome is generated
and then random normaly distributed covariates are obtained
by sampling the mean and standard deviation from an uniform distribution.


```r
nb_runs <- 5000
nb_samples <- 100
outcome <- rnorm(nb_samples, mean = 0, sd = 2)

results_association <- lapply(
    1:nb_runs,
    function(x, outcome){
        
        mean_normal <- runif(1, min = 0, max = 3)
        sd_normal <- runif(1, min = 0, max = 3)
        random_covariate <- rnorm(nb_samples, mean = mean_normal, sd = sd_normal)
        
        list( 
            results = lm(
                outcome ~ random_covariate
            ) %>% broom::tidy() %>% .[2, ],
            random_covariate = random_covariate
        )
    }, 
    outcome = outcome 
)

results_association_vals <- lapply(
    results_association,
    function(x) x$results
) %>% dplyr::bind_rows()
```

## Checking and plotting the results

Due to the nature of p-values, one can expect to see 5% of the p-values
to be smaller than 0.5. 


```r
color_text <- "blue"
results_association_vals %>% ggplot2::ggplot(aes(x = p.value)) +
    ggplot2::geom_histogram(bins = 20) + 
    ggplot2::geom_vline(xintercept = 0.05, linetype = "dashed", color = color_text) + 
    ggplot2::annotate(
        "text",
        x = 0.02,
        y = 200, 
        label = "0.05",
        color = color_text
    ) + 
    ggplot2::theme_bw()
```

<img src="{{ site.baseurl }}/assets/vanilla_dichotomization_files/figure-html/unnamed-chunk-3-1.png"  />

The histogram above looks like an uniform distribution. There are several
statistically significant results, and the probability of them to be
smaller than 0.05 is:
    

```r
mean(results_association_vals$p.value < 0.05) * 100
```

```
## [1] 4.86
```

as expected from a the uniform distribution of p-values.

## Dichotomizing the continuous variables

In this step the continuous covariates are dichotomized and means
of the two groups are compared. 
Again we expect an uniform distribution of the p-values. The trick used
here is the fact the slope of the linear regression corresponds to the 
difference of means.


```r
results_dichotomy <- lapply(
    results_association,
    function(x, outcome){
        random_covariate <- x$random_covariate
        dichotomy <- random_covariate > median(random_covariate)
        
        lm(outcome ~ dichotomy) %>% 
            broom::tidy() %>% .[2, ]
    },
    outcome = outcome
) %>% dplyr::bind_rows()
```

And below is the plot of the p-value distribution based on the dichotomized
simulation.

```r
color_text <- "blue"
results_dichotomy %>% ggplot2::ggplot(aes(x = p.value)) +
    ggplot2::geom_histogram(bins = 20) + 
    ggplot2::geom_vline(xintercept = 0.05, linetype = "dashed", color = color_text) + 
    ggplot2::annotate(
        "text",
        x = 0.05*2,
        y = 200, 
        label = "0.05",
        color = color_text
    ) + 
    ggplot2::labs(
        title = "p-value distribution of the dichotomized associations"
    ) + 
    ggplot2::theme_bw()
```

<img src="{{ site.baseurl }}/assets/vanilla_dichotomization_files/figure-html/unnamed-chunk-6-1.png"  />

It looks as expected. The plot below shows the relation
between the p-values of the continuous covariate and their dichotmized versions.


```r
final_pvalues <- data.frame(
    continuous = results_association_vals$p.value,
    dichotomy = results_dichotomy$p.value
)

final_pvalues %>% ggplot2::ggplot(aes(x = continuous, y = dichotomy)) + 
    ggplot2::geom_point(size = 0.5) + 
    ggplot2::geom_abline(slope = 1, intercept = 0, color = "red", size = 1) + 
    ggplot2::labs(
        x = "Continuous covariate",
        y = "Dichotomized covariate"
    )
```

<img src="{{ site.baseurl }}/assets/vanilla_dichotomization_files/figure-html/unnamed-chunk-7-1.png"  />

The p-values are not all similar, showing how unstable this process can be.

Let us focus now on the dichotomized covariates that 
are actually statistically significant and analyse the 
p-values from the continuous covariates.


```r
final_pvalues %>% 
    dplyr::filter(dichotomy < 0.05) %>% 
    ggplot2::ggplot(aes(x = continuous, y = dichotomy)) + 
    ggplot2::geom_point(size = 1) + 
    ggplot2::geom_abline(slope = 1, intercept = 0, color = "red", size = 1) +
    ggplot2::labs(
        x = "Continuous covariate",
        y = "Dichotomized covariate"
    )
```

<img src="{{ site.baseurl }}/assets/vanilla_dichotomization_files/figure-html/unnamed-chunk-8-1.png"  />

This plot shows that several dichotomized covariates showed an association
to the outcome, despite the fact that the continuous covariate showed
p-values ranging from 0 to 1. 

# Conclusion

By dichotomizing one can create an effect that is not actually there. This 
process can also be gamed by changing how one thresholds the data. 
Dichotomization should be avoided whenever possible. 

In the next blog post I show a simulation example using survival data. 
Stay tuned!
