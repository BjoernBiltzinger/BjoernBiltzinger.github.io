---
layout: post
title:  Profiled Likelihoods in Bayesian Inference 
date:   2022-03-25 00:00:00 +0300
image:  '/images/posts_main/profiled.png'
tags:   [data analysis]
use_math: true
---
# Validating Profiled Likelihood for Bayesian Inference: A Practical Guide

In Bayesian inference, we often face computationally expensive calculations when dealing with nuisance parameters. One common approach to simplify these calculations is using profiled likelihood. While this method is theoretically well-understood (as excellently explained in [this blog post](https://giacomov.github.io/Bias-in-profile-poisson-likelihood/)), its practical validation for specific use cases remains crucial. In this post, we'll explore a simple yet illustrative example to demonstrate how to validate profiled likelihood against the full marginalization approach.

## The Setup: A Simple Detection Problem

Consider a detector with 50 energy bins and a perfect response. We're observing:
1. A source emitting a constant spectrum (rate 'm' per channel)
2. Background radiation (rate 'b' per channel)
3. A separate background-only observation for calibration

This setup gives us:
- Source+Background observation with exposure time $t_{obs}$
- Background-only observation with exposure time $t_{bkg}$
- Counts per energy bin for both observations ($C_{obs}$ and $C_{bkg}$)

Without the background-only observation, disentangling source and background contributions would be impossible. This two-observation approach provides the necessary information for meaningful analysis.

## Implementation

First, let's import the necessary libraries and set up our environment:

```python
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt
from numba import njit, vectorize, int64, float64
from scipy.integrate import quad
from math import lgamma
```

### Data Simulation

Let's create some simulated data to work with:

```python
def get_data(model_rate, exposure, num_simulations):
    return np.random.poisson(model_rate*exposure, size=num_simulations)

def betterstep(bins, y, **kwargs):
    new_x = [a for row in zip(bins[:-1], bins[1:]) for a in row]
    new_y = [a for row in zip(y, y) for a in row]
    ax = kwargs.pop("ax", plt.gca())
    return ax.plot(new_x, new_y, **kwargs)

np.random.seed(1000)
m = 10
b = 10
t_obs = 10
t_bkg = 10
data_obs = get_data(m+b, exposure=t_obs, num_simulations=50)
data_bkg = get_data(b, exposure=t_bkg, num_simulations=50)
```

### Data and Initial Visualization

After simulating our data, let's visualize the counts across energy bins:

```python
fig, ax = plt.subplots()
betterstep(np.arange(len(data_obs)+1), data_obs, ax=ax, label="Sim. obs. data")
betterstep(np.arange(len(data_obs)+1), data_bkg, ax=ax, label="Sim. bkg. obs. data")
ax.set_ylabel("Counts")
ax.set_xlabel("Energy Bin ID")
ax.legend()
```

<div class="gallery-box">
<div class="gallery">
<img src="/images/profiled/fig1.png">
</div>
</div>

We observe natural Poisson noise scattering around the expected values (200 for observation and 100 for background observation).

### The Theory Behind It

Next, we need to discuss the probability theory required for Bayesian analysis of this data. We start with Bayes' Theorem:

$P(θ\|d) = \frac{P(d\|θ) P(θ)}{P(d)}$

Where:
- P(θ\|d) is the posterior distribution
- P(d\|θ) is the likelihood (L) of the data d given parameters θ
- P(θ) is the parameter prior
- P(d) is the evidence that normalizes the posterior

In our specific case, θ is a vector with 51 entries: one for the model rate and 50 for the background rates in each energy bin. We assume no prior knowledge about the background shape, so we can't connect the background rates of different energy bins with a global model (which would be ideal). 

With this many parameters and our interest focused solely on the model rate, the standard Bayesian approach would be to marginalize over the background rates - integrating over all possible background rates (bi>0). While straightforward to write mathematically, this integration lacks an analytic solution and is computationally expensive to solve numerically.

The alternative is using the profiled likelihood approach, as derived in [this blog post](https://giacomov.github.io/Bias-in-profile-poisson-likelihood/). This method leverages the fact that we can find the background rate that maximizes the likelihood for any given model rate. This maximized likelihood serves as an approximation for the marginalization of the background rate. While this might seem counterintuitive, it often works remarkably well, primarily because L(b) typically has a sharp peak at the maximum likelihood background rate.

In this analysis, we'll implement and compare both approaches to verify the validity of this approximation and identify any potential limitations.

## Different Approaches

First, let's define the model rates we'll use for calculations and plots:

```python
model_rates_test = np.linspace(0,20,200)
```

### Profiled Approach

We first have to define a few functions we will need for the calculations. We use [numba](https://numba.pydata.org/) here to speed the calculations up.

```python
@vectorize([float64(int64)], fastmath=True)
def logfactorial(n):
    """
    helper function to calculate the logarithm of the factorial of a number
    """
    return lgamma(n + 1)

@njit
def log_pstat(observed_counts, model_rates, exposure):
    """
    Calculate log likelihood for a given observation and model counts
    """
    if observed_counts>0:
        return observed_counts*np.log(exposure*model_rates)-exposure*model_rates-logfactorial(observed_counts)
    else:
        return -exposure*model_rates-logfactorial(observed_counts)
    
@njit
def log_ppstat(observed_counts, background_counts, m, background_rates, exposure_obs, exposure_bkg, logLmax=0):
    """
    Calcualte the combined log likelihood of the observation and background observation for one given model rate m
    """
    pstat_bkg = log_pstat(background_counts, background_rates, exposure_bkg)
    pstat_obs = log_pstat(observed_counts, background_rates+m, exposure_obs)
    return pstat_bkg+pstat_obs-logLmax

@njit 
def get_profiled_bkg_rate(observed_counts, background_counts, model_rates, exposure_obs, exposure_bkg):
    """
    Get the background rate that maximes the likelihood
    """
    prefact = 1./(2*(exposure_bkg+exposure_obs))
    first_term = background_counts+observed_counts-model_rates*(exposure_bkg+exposure_obs)
    second_term = np.sqrt((background_counts+observed_counts)**2+
                          2*background_counts*model_rates*(exposure_bkg+exposure_obs)-
                          2*observed_counts*model_rates*(exposure_bkg+exposure_obs)+
                          model_rates**2*(exposure_bkg+exposure_obs)**2)
    return prefact*(first_term+second_term)
```

Calculate and plot $logL(m)$ for profiled case

```python
# profiled likelihood
log_likes_profiled = np.zeros_like(model_rates_test)
for i, m_test in enumerate(model_rates_test):
    bkg_rates_profiled = get_profiled_bkg_rate(data_obs, data_bkg, m_test, t_obs, t_bkg)
    for j in range(len(data_obs)):
        log_likes_profiled[i] += log_ppstat(data_obs[j], data_bkg[j], m_test, bkg_rates_profiled[j], t_obs, t_bkg)

fig, ax = plt.subplots()
ax.plot(model_rates_test, log_likes_profiled, color="darkturquoise", label="bkg profiled")
ax.axvline(m, linestyle="--", color="darkred", label="Truth")
ax.set_xlabel("Model rate")
ax.set_ylabel("$logL$")
ax.legend();
```

Let's calculate and plot $logL(m)$ for the profiled case:

```python
fig, ax = plt.subplots()
ax.plot(model_rates_test, log_likes_profiled, color="darkturquoise", label="bkg profiled")
ax.axvline(m, linestyle="--", color="darkred", label="Truth")
ax.set_xlabel("Model rate")
ax.set_ylabel("logL")
ax.legend()
```
<div class="gallery-box">
<div class="gallery">
<img src="/images/profiled/fig2.png">
</div>
</div>

The log likelihood is largest near the true value of the model rate, which is encouraging.

### Marginalized Approach

Again we have to first define a few new functions for the integration over the background rate.

```python
def log_marg_ppstat_one(observed_counts, background_counts, m, exposure_obs, exposure_bkg):
    """
    Calculate the log likelihood when marg. over background rate b for one given model rate m
    """
    logintL = 0
    for observed_count, background_count in zip(observed_counts, background_counts):
        # get the b paramter that maximes the likelihood
        bmax = get_profiled_bkg_rate(observed_count, background_count, m, exposure_obs, exposure_bkg)    
        logLmax = log_ppstat(observed_count, background_count, m, bmax, exposure_obs, exposure_bkg)

        def integrand(b):
            """
            Gives logL(b) for a given m.
            We also normalize L(b) such that the maximum is 1 (=> logL at maximum is 0).
            This makes the integration numerically stable. We correct this factor again after
            the integration.
            """
            return np.exp(log_ppstat(observed_count, background_count, m, b, exposure_obs, exposure_bkg, logLmax))

        # marg. over the background parameter. Use the points variable to make sure the integration is not
        # missing the important part around bmax
        logL = np.log(quad(integrand, 0, bmax*10+10, points=[0,bmax])[0])+logLmax
        logintL += logL
    return logintL

def log_likes_marg_all(model_rates, obs_counts, bkg_counts, exposure_obs, exposure_bkg):
    """
    Calculate the log likelihood when marg. over background rate b for an array of model rates model_rates
    """
    log_likes_marg = np.zeros_like(model_rates)

    for i, m in enumerate(model_rates):
        log_likes_marg[i] = log_marg_ppstat_one(obs_counts, bkg_counts, m, exposure_obs, exposure_bkg)
    return log_likes_marg
```

Let's calculate and plot $logL(m)$ for the marginalized case:

```python
fig, ax = plt.subplots()
ax.plot(model_rates_test, log_likes_marg, color="darkmagenta",label="bkg marg.")
ax.axvline(m, linestyle="--", color="darkred", label="Truth")
ax.set_xlabel("Model rate")
ax.set_ylabel("logL")
ax.legend()
```

<div class="gallery-box">
<div class="gallery">
<img src="/images/profiled/fig3.png">
</div>
</div>

### Comparing Both Approaches

Let's compare both methods directly:

```python
fig, ax = plt.subplots()
ax.plot(model_rates_test, log_likes_profiled, color="darkturquoise", label="bkg profiled")
ax.plot(model_rates_test, log_likes_marg, color="darkmagenta", label="bkg marg.", linestyle="--")
ax.axvline(m, linestyle="--", color="darkred", label="Truth")
ax.set_xlabel("Model rate")
ax.set_ylabel("logL")
ax.legend()
```

<div class="gallery-box">
<div class="gallery">
<img src="/images/profiled/fig4.png">
</div>
</div>

We notice the logL curves are shifted relative to each other. Let's normalize them by subtracting their maximum values:

```python
fig, ax = plt.subplots()
ax.plot(model_rates_test, log_likes_profiled-log_likes_profiled.max(), color="darkturquoise", label="bkg profiled")
ax.plot(model_rates_test, log_likes_marg-log_likes_marg.max(), color="darkmagenta", label="bkg marg.", linestyle="--")
ax.axvline(m, linestyle="--", color="darkred", label="Truth")
ax.set_xlabel("Model rate")
ax.set_ylabel("logL")
ax.legend()
```

<div class="gallery-box">
<div class="gallery">
<img src="/images/profiled/fig5.png">
</div>
</div>

The curves now match remarkably well. The vertical shift between the two logL curves doesn't affect parameter inference, though it would matter for evidence calculations.

Let's examine $L(m)$ (normalized so the integral between 0 and 20 is 1) with a closer view:

```python
fig, ax = plt.subplots()
ax.plot(model_rates_test, np.exp(log_likes_profiled)/int_like_profiled, color="darkturquoise", label="bkg profiled")
ax.plot(model_rates_test, np.exp(log_likes_marg)/int_like_marg, color="darkmagenta", label="bkg marg.", linestyle="--")
ax.axvline(m, linestyle="--", color="darkred", label="Truth")
ax.set_xlabel("Model rate")
ax.set_ylabel("L")
ax.set_xlim(8.5,11.5)
ax.legend()
```

<div class="gallery-box">
<div class="gallery">
<img src="/images/profiled/fig6.png">
</div>
</div>

## Testing Multiple Scenarios

To ensure our findings are robust, let's test several combinations of parameters:

```python
np.random.seed(1000)
m = np.linspace(0.1,10,4)
b = np.linspace(0.1,10,4)
num_simulations=50

# test for 16 different m,b combinations
mm, bb = np.meshgrid(m,b)
t_obs = 10
t_bkg = 10
data_obs_all = np.zeros((len(m), len(b), num_simulations), dtype=int)
data_bkg_all = np.zeros((len(m), len(b), num_simulations), dtype=int)
for i in range(len(m)):
    for j in range(len(b)):
        data_obs_all[i,j] = get_data(mm[i,j]+bb[i,j], exposure=t_obs, num_simulations=num_simulations)
        data_bkg_all[i,j] = get_data(bb[i,j], exposure=t_bkg, num_simulations=num_simulations)
fig, ax = plt.subplots(*mm.shape, figsize=(10,10), sharex=True)
for i in range(len(m)):
    for j in range(len(b)):
        label1 = None
        label2 = None
        if i==0 and j==0:
            label1 = "Sim. obs. data"
            label2 = "Sim. bkg. obs. data"
        betterstep(np.arange(len(data_obs_all[i,j])+1), data_obs_all[i,j], ax=ax[i,j], label=label1)
        betterstep(np.arange(len(data_bkg_all[i,j])+1), data_bkg_all[i,j], ax=ax[i,j], label=label2)
        ax[i,j].set_ylabel("Counts")
        ax[i,j].set_xlabel("Energy Bin ID")
        ax[i,j].set_title(f"m: {round(mm[i,j], 1)}, b: {round(bb[i,j],1)}")
        
fig.tight_layout()
lgd = fig.legend(loc='center left', bbox_to_anchor=(1, 0.5), fontsize='x-large')
```

<div class="gallery-box">
<div class="gallery">
<img src="/images/profiled/fig7.png">
</div>
</div>

And compare the likelihood calculations for all cases:

```python
fig, ax = plt.subplots(*mm.shape, figsize=(10,10))
for i in range(len(m)):
    for j in range(len(b)):
        label1 = None
        label2 = None
        if i==0 and j==0:
            label1 = "bkg profiled"
            label2 = "bkg marg."
        # profiled likelihood
        log_likes_profiled = np.zeros_like(model_rates_test)
        for q, m_test in enumerate(model_rates_test):
            bkg_rates_profiled = get_profiled_bkg_rate(data_obs_all[i,j], data_bkg_all[i,j], m_test, t_obs, t_bkg)
            for k in range(len(data_obs)):
                log_likes_profiled[q] += log_ppstat(data_obs_all[i,j][k], data_bkg_all[i,j][k], m_test, bkg_rates_profiled[k], t_obs, t_bkg)
                
        int_like_profiled = np.trapz(np.exp(log_likes_profiled), model_rates_test)
        ax[i,j].plot(model_rates_test, np.exp(log_likes_profiled)/int_like_profiled, color="darkturquoise", label=label1)
        # marg
        log_likes_marg = log_likes_marg_all(model_rates_test, data_obs_all[i,j], data_bkg_all[i,j], t_obs, t_bkg)
        int_like_marg = np.trapz(np.exp(log_likes_marg), model_rates_test)
        ax[i,j].plot(model_rates_test, np.exp(log_likes_marg)/int_like_marg, color="darkmagenta", label=label2, linestyle="--")

        xlim = mm[i,j]-1
        if xlim<0:
            xlim = 0
        xlim2 = mm[i,j]+1
        ax[i,j].set_xlim(xlim, xlim2)
        ax[i,j].set_title(f"m: {round(mm[i,j], 1)}, b: {round(bb[i,j],1)}")
        ax[i,j].set_xlabel("Model rate")
        ax[i,j].set_ylabel("$L$")
fig.tight_layout()
lgd = fig.legend(loc='center left', bbox_to_anchor=(1, 0.5), fontsize='x-large')
```

<div class="gallery-box">
<div class="gallery">
<img src="/images/profiled/fig8.png">
</div>
</div>

## Conclusions

Our analysis reveals that the profiled likelihood approach is a reliable approximation in most scenarios, with a few important caveats:

1. The approximation works best when there are sufficient counts in each bin
2. For very low background rates, the differences between profiled and marginalized likelihoods become more pronounced
3. The approach breaks down when dealing with bins containing zero counts

Therefore, when using profiled likelihood:
- Ensure your binning strategy results in at least a few counts per bin
- Validate the approach for your specific use case
- Be cautious with low-count scenarios
