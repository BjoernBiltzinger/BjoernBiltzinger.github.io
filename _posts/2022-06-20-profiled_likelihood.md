---
layout: single
title: Profiled Likelihoods in Bayesian Inference 
category: blog
preview_text: Profiled likelihoods are often used for Bayesian inference in physics, even though we know that they are only exact for maximum likelihood approaches. So in this test I check whether it is an acceptable approximation for Bayesian inference. 
---

In this notebook we want to check if the profiled likelihood is a valid approximation for Bayesian Inference. The concept of profiled likelihood is nicely derived in this [blog post](https://giacomov.github.io/Bias-in-profile-poisson-likelihood/). Here we will check for a very simple case if it gives similar results compared to the non-approximate, but computational way more expensive, solution of marginalizing. If you are interested to use profiled likelihoods for a more complicated case it would be a good idea to run this kind of test for you special case, too.

Let's start with defining our models and observations. In this case we assume to have a detector with 50 equally spaced energy bins and a perfect response (so no response folding is needed). The source we are interested in emits a constant spectrum with a rate of m in every energy channels. In addition there is background, that is luckily also a constant spectrum, which produces a rate of b in every energy channel. We have a model predicting the constant source spectrum but we have no clue what the background spectrum is. With this information there would be no way to constrain the source model contribution, because we are missing information that can disentangle background and source. Therefore we do a second observation of an emtpy area of the sky, in which the background is the only contribution to the measured signal. These two observations give us enough information to disentangle background and source quiet well, like also shown [here](https://giacomov.github.io/Bias-in-profile-poisson-likelihood/). The variable names we use are listed in the following:

1. $m$; model rate (counts per second)
2. $b$; background rates (counts per second)
3. $t_{obs}$; exposure of the observation
4. $t_{bkg}$; exposure of the background observation
5. $C_{obs}$; counts per energy bin for the observation
6. $C_{bkg}$; counts per energy bin for the background observation


# Imports

```python
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt
from numba import njit, vectorize, int64, float64
from scipy.integrate import quad
from math import lgamma
plt.style.use("dark_background")
```

<!-- #region tags=[] -->
# Simulate data
<!-- #endregion -->

First we simulate some data and have a look at it. We are doing a counting experiment here, therefore the appropriate likelihood is a Poisson likelihood.

```python
def get_data(model_rate, exposure, num_simulations):
    return np.random.poisson(model_rate*exposure, size=num_simulations)

def betterstep(bins, y, **kwargs):
    new_x = [a for row in zip(bins[:-1], bins[1:]) for a in row]
    new_y = [a for row in zip(y, y) for a in row]
    ax = kwargs.pop("ax", plt.gca())
    return ax.plot(new_x, new_y, **kwargs)
```

```python
np.random.seed(1000)
m = 10
b = 10
t_obs = 10
t_bkg = 10
data_obs = get_data(m+b, exposure=t_obs, num_simulations=50)
data_bkg = get_data(b, exposure=t_bkg, num_simulations=50)
fig, ax = plt.subplots()
betterstep(np.arange(len(data_obs)+1), data_obs, ax=ax, label="Sim. obs. data")
betterstep(np.arange(len(data_obs)+1), data_bkg, ax=ax, label="Sim. bkg. obs. data")
ax.set_ylabel("Counts")
ax.set_xlabel("Energy Bin ID")
ax.legend()
```
<center>
<img src="{{site.baseurl}}/assets/images/sim1.png">
<br>
</center>
We see some natural scattering around the expected values (200 for observation and 100 for background observation), due to Poisson noise. 
Next we have to discuss the probabilty theory we need to analyse this data in the Bayesian framework. We start with Bayes Theorem: $P\left( \vec{\theta}|\vec{d} \right) = \frac{P\left( \vec{d}|\vec{\theta} \right) P\left( \vec{\theta} \right) }{P\left( \vec{d} \right) }$, where $P\left( \vec{\theta}|\vec{d} \right)$ is the posterior distribution, $P\left( \vec{d}|\vec{\theta}\right)$ the likelihood (in the following $\mathcal{L}$) of the data $\vec{d}$ for given parameters $\vec{\theta}$, $P\left( \vec{\theta} \right)$ the prior for the parameters and $P\left( \vec{d} \right)$ the evidence, that normalizes the posterior. Good introduction into the topic of Bayesian inference can be found here: .

In our case $\vec{\theta}$ is a vector with 51 entries: one for the model rate and 50 for the background rates in each energy bin. Recall that we assume here that we have no idea what the background looks like, so we can also not connect the background rates of the different energy bins with a global model (which would be the ideal case). These are a lot of parameters and we are actually not interested in what the background rate in each bin is. Therefore the Bayesian way of dealing with this would be to marginalize over the background rates, i.e. simply integrating over all possible background rates ($b_i$>0). But even though it is easy to write down there is no analytic solution for this integral and it is difficult to solve numerically (computationally expensive). The alternative is to use the profiled likelihood like derived [here](https://giacomov.github.io/Bias-in-profile-poisson-likelihood/). It uses the fact that we can find the background rate that maximize the likelihood for a given model rate. This maximized likelihood can be used as an approximation of the marginalization of the background rate. This sounds strange but usually works very well, mostly due to the fact that $\mathcal{L}(b)$ is usually very strongly peaked at the background rate that maximizes the likelihood. But here we want to check how well it works and if it breaks down in some extreme cases. For this we will calculate $\mathcal{L}(m)$ for both approaches and compare them.


# Different Approaches


Define the model rates to use for the calculation and plots

```python
model_rates_test = np.linspace(0,20,200)
```

## Profiled


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

Calculate and plot $log\mathcal{L}(m)$ for profiled case

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
ax.set_ylabel("$log\mathcal{L}$")
ax.legend();
```
<center>
<img src="{{site.baseurl}}/assets/images/logL_prof.png">
<br>
</center>
Looks reasonable. The log likelihood is largest close to the true value of the model rate. 


## Marginalized


Next we test the marginalization approach. Again we have to first define a few new functions for the integration over the background rate.

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

Calculate and plot $log\mathcal{L}(m)$ for marginalized case

```python
# marg. likelihood
log_likes_marg = log_likes_marg_all(model_rates_test, data_obs, data_bkg, t_obs, t_bkg)

fig, ax = plt.subplots()
ax.plot(model_rates_test, log_likes_marg, color="darkmagenta",label="bkg marg.")
ax.axvline(m, linestyle="--", color="darkred", label="Truth")
ax.set_xlabel("Model rate")
ax.set_ylabel("$log\mathcal{L}$")
ax.legend();
```
<center>
<img src="{{site.baseurl}}/assets/images/logL_marg.png">
<br>
</center>
Looks again reasonable.


## Compare both


Let's plot it together in one plot.

```python
fig, ax = plt.subplots()
ax.plot(model_rates_test, log_likes_profiled, color="darkturquoise", label="bkg profiled")
ax.plot(model_rates_test, log_likes_marg, color="darkmagenta", label="bkg marg.", linestyle="--")
ax.axvline(m, linestyle="--", color="darkred", label="Truth")
ax.set_xlabel("Model rate")
ax.set_ylabel("$log\mathcal{L}$")
ax.legend();
```
<center>
<img src="{{site.baseurl}}/assets/images/logL_compare.png">
<br>
</center>
We see that the $log\mathcal{L}$ curves are shifted with respect to each other. But the shift looks more or less constant. Let's test it by subtracting the maximum of the functions.

```python
fig, ax = plt.subplots()
ax.plot(model_rates_test, log_likes_profiled-log_likes_profiled.max(), color="darkturquoise", label="bkg profiled")
ax.plot(model_rates_test, log_likes_marg-log_likes_marg.max(), color="darkmagenta", label="bkg marg.", linestyle="--")
ax.axvline(m, linestyle="--", color="darkred", label="Truth")
ax.set_xlabel("Model rate")
ax.set_ylabel("$log\mathcal{L}$")
ax.legend();
```
<center>
<img src="{{site.baseurl}}/assets/images/logL_compare_norm.png">
<br>
</center>
Now the curves match very nicely. That the two $log\mathcal{L}$ curves are shifted with respect to each other is not important for parameter inference. Only if we would be interested in the evidence it would be a problem. Let's also have a look $\mathcal{L}(m)$ (normalized such that the integral between 0 and 20 is 1) and zoom in.

```python
int_like_profiled = np.trapz(np.exp(log_likes_profiled), model_rates_test)
int_like_marg = np.trapz(np.exp(log_likes_marg), model_rates_test)


fig, ax = plt.subplots()
ax.plot(model_rates_test, np.exp(log_likes_profiled)/int_like_profiled, color="darkturquoise", label="bkg profiled")
ax.plot(model_rates_test, np.exp(log_likes_marg)/int_like_marg, color="darkmagenta", label="bkg marg.", linestyle="--")
ax.axvline(m, linestyle="--", color="darkred", label="Truth")
ax.set_xlabel("Model rate")
ax.set_ylabel("$\mathcal{L}$")
ax.set_xlim(8.5,11.5)
ax.legend();
```
<center>
<img src="{{site.baseurl}}/assets/images/L_compare.png">
<br>
</center>
We see that the distributions are not identical, but the profiled likelihood is shifted slightly to higher model rates. But the difference is really marginal an most likely in any real application subdominant to other assumptions. Therefore the profiled likelihood seems to be a valid approximation in this case.


# Test several simulations


Let's test the same we did for one simulation for several simulation. This way we test different scenarios.

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
<center>
<img src="{{site.baseurl}}/assets/images/many_sim.png">
<br>
</center>
Calculate the $\mathcal{L}(m)$ for all simulations with the profiled and the marginalized approach.

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
        ax[i,j].set_ylabel("$\mathcal{L}$")
fig.tight_layout()
lgd = fig.legend(loc='center left', bbox_to_anchor=(1, 0.5), fontsize='x-large')
```
<center>
<img src="{{site.baseurl}}/assets/images/L_compare_many_sim.png">
<br>
</center>
We see that the profiled likelihood seems to be a reasonable approximation in all tested combinations. But for very low background rates the difference becomes larger. This is due to the fact, that the profiled likelihood approach breaks down if there are bins with 0 counts, which was shown in this [blog post](https://giacomov.github.io/Bias-in-profile-poisson-likelihood/). The more bins have 0 counts the worse the profiled likelihood gets. Therefore always make sure to bin your data to have at least a few counts in every bin, if you want to use a profiled likelihood!


# Summary


We have seen that, for the easy example here, the profile likelihood is a good approximation also for Bayesian Inference, as long as there are no bins with zero counts. Tests like these should always be done before using a profiled likelihood in a real application.
