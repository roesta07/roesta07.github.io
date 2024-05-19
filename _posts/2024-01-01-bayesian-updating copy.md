---
title: "Beta-Binomial Model and Multilevel Binomial Model: A Comparison"
permalink: bayesian-beta-binomial
description: Here's a intutive introduction to Bayesian Statistics
last_modified_at: 2022-10-01T16:20:02-05:00
mathjax: true
toc: true
# categories:
#   - Blog
# tags:
#   - Bayesian
#   - probability
#   - standard
excerpt: A moment of realization when the Beta-Binomial Model and the Multilevel Binomial Model produces similar estimates
---

In this article, we will compare two models: the Beta Binomial Model and the Multilevel Binomial Model, to see how they are equivalent and produce similar estimates

## Quick Intro on Beta-Binomial likelihood

Let us assume the random variable S(success) to be binomially distributed, as

$$
S|\theta \sim Binomial(n,\theta)
$$

Note that S is not marginally Binomial; it is conditionally Binomial, given $$\theta$$. If $$\theta$$ varies across trials then the marginal distribution of S is called the [Beta-Binomial distribution](https://en.wikipedia.org/wiki/Beta-binomial_distribution). i.e

$$
S \sim BetaBinomial(n,{\theta})
$$

Thus, these types of likelihood/ models are widely used to account for dispersion in the data as $$\theta$$ varies across trials.

## Data

For demonstration purposes, I have simulated only two variables: the Number of Trials (N), which is fixed for each experiment, and the Number of Successes (S).

<p float="center">
  <img src="/assets/images/posts/bayesian-beta-binomial/data00.png" width=150/>
  <img src="/assets/images/posts/bayesian-beta-binomial/distribution00.png" height =90 width=400 /> 
</p>


We will use Bayesian methods for both models to facilitate comparison of their estimates. The main parameter of interest is the probability of success, denoted as
$$\theta$$

## Full model Specifications

### **Beta Binomial (Model 1)**

$$
S \sim BetaBinomial(n,\theta) \quad Model\ 1
$$

$$
\theta \sim Beta(\alpha,\beta)
$$

$$
\mu \sim Beta(2,2)
$$

$$
k \sim Exponential(1)
$$

### **Multilevel Binomial (Model 1)**

$$
S_{i} \sim Binomial(n,\theta_{i}) \quad Model\ 2
$$

$$
\theta_i \sim Beta(\alpha,\beta)
$$

$$
\alpha = \mu* k
$$

$$
\mu \sim Beta(2,2)
$$

$$
k \sim Exponential(1)
$$

## Prior

We can see that the same prior distributions have been provided for both models. To verify this, let's plot the prior distribution of the probability of success, i.e. $$\theta\$$.

![prior01.png](assets/images/posts/bayesian-beta-binomial/prior01.png)

*Fig: Prior distribution for probability of success - $$\theta$$.*

As observed, we have given a flat prior with some spikes at 0 and 1. This is not ideal; it is generally better to use informative priors. However, for the sake of demonstration, let's keep it as it is.

## Posterior

![posterior02.png](assets/images/posts/bayesian-beta-binomial/posterior02.png)

Figure: Posterior Distribution of $$\theta$$

The plot shows that the model produces similar estimates for $$\theta$$. This is not a coincidence; rather, it is because the underlying specifications of the models are the same under certain assumptions. Both models produce similar estimates because they both account for variability in the data in comparable ways. The Beta Binomial Model incorporates a prior distribution (Beta distribution) to handle variability in the success probability across different trials, while the Multilevel Binomial Model, also known as a hierarchical model, uses random effects to achieve a similar effect by allowing for group-level variation.

Therefore we can say that in the Beta-Binomial model, each observation's likelihood is modeled with a binomial distribution, and its probability parameter ($$\theta$$) is given a unique beta prior with shared parameter alpha/beta for flexibility and uncertainty representation.

## Choosing the Right Model

All these comparison boils down to choosing the right model.Choosing the right model depends on several factors. If you only need the overall probability of success, $$\theta$$, as used here, the Beta-Binomial model (Model 1) is a good choice because it samples faster than the multilevel model. However, if you are interested in the probability of success for each individual observation, $$\theta_{i} $$, you need to use the Multilevel Binomial model (Model 2). Below is the plot for the probability of success for the first 5 observations.

![posterior03.png](assets/images/posts/bayesian-beta-binomial/posterior03.png)

*Figure: Posterior Distribution of $$\theta$$ for first five trials/experiment*

I typically prefer multilevel models because they provide more detailed information, which helps to better understand how the data is generated.

