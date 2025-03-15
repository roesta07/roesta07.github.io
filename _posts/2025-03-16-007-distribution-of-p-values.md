---
title: "Simulating the Distribution of P-Values"
permalink: uniform_distribution_p_values
description: 
last_modified_at: 2025-03-16T16:20:02-05:00
mathjax: true
toc: false
# categories:
#   - Blog
# tags:
#   - Bayesian
#   - probability
#   - standard
excerpt: Simulating p-values helps in study design, significance testing, and controlling false discoveries. See how they behave in different scenarios!
---
## **Introduction**

P-values are fundamental to statistical hypothesis testing, yet their behavior is often misunderstood. When the null hypothesis ($$H_0$$) is true, p-values follow a **Uniform(0,1)** distribution. However, when the alternative hypothesis ($$H_A$$) is true, p-values tend to cluster near zero. 
Understanding these distributions is essential for designing studies, setting appropriate significance levels, and managing false discovery rates (FDR).
This is possible because of this fascinating probability concept: the [Probability Integral Transform / Universality of the Uniform](https://en.wikipedia.org/wiki/Probability_integral_transform#Proof). Please check those before exploring this


In this blog, we explore:

1. **Case 1:** Null and observed distributions are identical ($$H_0$$ is true).
2. **Case 2:** Null and observed distributions are different ($$H_A$$ is true).
   
## **Step 1: Defining the Distributions**

We define our **null distribution** (which represents data under $$H_0$$) and two **observed distributions**:

- One that is identical to the null (no real effect).
- One that is shifted (indicating a true effect).

```python
import pymc as pm
import numpy as np
import scipy.stats as stats
import matplotlib.pyplot as plt
import seaborn as sns
import pytensor.tensor as pt
import pytensor

# Define the null and observed distributions
null_dist = pm.Normal.dist(0, 4)       # Null Distribution (H0 True)
observed_dist = pm.Normal.dist(0, 4)   # Observed Distribution (H0 True)
observed_dist2 = pm.Normal.dist(2, 4)  # Observed Distribution (H_A True)

```

---

## **Step 2: Computing the Probability Density Functions (PDFs)**

To visualize the distributions, we compute their PDFs using PyTensor:

```python
value = pt.scalar()

# Compute log probabilities and exponentiate to get the PDFs
null_pdf = pm.logp(null_dist, value).exp()
observed_pdf = pm.logp(observed_dist, value).exp()
observed2_pdf = pm.logp(observed_dist2, value).exp()

# Convert into callable functions
get_null_pdf = pytensor.function([value], [null_pdf])
get_observed_pdf = pytensor.function([value], [observed_pdf])
get_observed2_pdf = pytensor.function([value], [observed2_pdf])

```

---

## **Step 3: Visualizing the Distributions**

We now plot the null and observed distributions for both cases.

```python
fig, axs = plt.subplots(1, 2, figsize=(10, 5))
eXs = np.linspace(-10, 10, 1000)

# Compute PDF values
eY_null = np.array([get_null_pdf(eX) for eX in eXs]).squeeze()
eY_observed = np.array([get_observed_pdf(eX) for eX in eXs]).squeeze()
eY_observed2 = np.array([get_observed2_pdf(eX) for eX in eXs]).squeeze()

# Case 1: Null = Observed (H0 True)
axs[0].plot(eXs, eY_null, label="Null Distribution", linestyle="dashed", color="blue")
axs[0].plot(eXs, eY_observed, label="Observed Distribution", linestyle="solid", color="red")
axs[0].fill_between(eXs, eY_null, alpha=0.2, color="blue")
axs[0].fill_between(eXs, eY_observed, alpha=0.2, color="red")
axs[0].set_title("Case 1: H0 is True")
axs[0].legend()

# Case 2: Null ≠ Observed (HA True)
axs[1].plot(eXs, eY_null, label="Null Distribution", linestyle="dashed", color="blue")
axs[1].plot(eXs, eY_observed2, label="Observed Distribution", linestyle="solid", color="red")
axs[1].fill_between(eXs, eY_null, alpha=0.2, color="blue")
axs[1].fill_between(eXs, eY_observed2, alpha=0.2, color="red")
axs[1].set_title("Case 2: H_A is True")
axs[1].legend()

plt.show()

```
<div style="text-align: center;">
  <img src="/assets/images/posts/007/plot1.png" width="1200" class="inline">
</div>

In Case 1, the two distributions overlap perfectly, meaning no real difference between null and observed data.
In Case 2, the observed distribution is shifted, which introduces a real effect.
---

## **Step 4: Simulating P-Values**

We now simulate and compare p-values under both cases.

```python
# Generate 10,000 samples
samples_null = np.random.normal(0, 4, 10000)
samples_obs_same = np.random.normal(0, 4, 10000)  # H0 True
samples_obs_diff = np.random.normal(2, 4, 10000)  # H_A True

# Compute p-values using the null CDF
pval=1-(pm.logcdf(null_dist,observed_dist).exp())
get_pval =pytensor.function([observed_dist],[pval])

pval2=1-(pm.logcdf(null_dist,observed_dist2).exp())
get_pval2 =pytensor.function([observed_dist2],[pval2])
pvals=pm.draw(pval,100000)
pvals2 = pm.draw(pval2,100000)

eX =np.linspace(-5,5,200)
eY = [get_pval(x) for x in eX]
eY2 = [get_pval2(x) for x in eX]
# eY2= [pval_eval(x) for ]
plt.plot(eX,eY)
plt.plot(eX,eY2)
plt.title("Survival Function for \n One sided T-test")

```
<div style="text-align: center;">
  <img src="/assets/images/posts/007/plot2.png" width="1200" class="inline">
</div>

```python
# Plot the p-value distributions
fig,axs =plt.subplots(1,2,figsize=(10,5))
sns.histplot(x=pvals, bins=50, stat='density', color='royalblue', edgecolor='black', ax=axs[0])
sns.histplot(x=pvals2, bins=50, stat='density', color='royalblue', edgecolor='black', ax=axs[1])
axs[0].set(
    xlabel='Values',
    ylabel='Density',
    title='Distribution of p-values \n case1',
    # yticks=np.arange(0, 1.2, 0.1),
    yticks=[0, 0.5, 1],
    # ylim=(0, 2),
    ylim=(0, 1.5)
)

axs[1].set(
    xlabel='Values',
    ylabel='Density',
    title='Distribution of p-values  \n case2',
    xticks=np.arange(0, 1.2, 0.1),
    # xticks=[0, 0.5, 1],
    # ylim=(0, 2),
    xlim=(0, 1)
)

```
<div style="text-align: center;">
  <img src="/assets/images/posts/007/plot3.png" width="1200" class="inline">
</div>

---

## **Conclusion**

Simulating p-values provides valuable insights into statistical testing. By comparing cases where $$H_0$$ is true vs. $$H_A$$ is true, we can:

✅ Verify test calibration.

✅ Understand when and why $$H_0$$ is rejected.

✅ Design more effective studies.
