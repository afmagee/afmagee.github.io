---
title: "A tail of two shrinkage priors"
author_profile: true
layout: single
---

## A tail of two shrinkage priors

Two priors, both alike in dignity, in fair Bayesian inference, where we lay our scene...

Sometimes, in Bayesian inference, the way we specify prior information takes a form like "I think that the mutation rate is probably on the order of 1e-8 per site per year."
Other times, we want to specify information on the structure of the model.
In regression contexts, that can be information about the number of covariates which are non-zero.
In Bayesian phylogenetics, Bayesian stochastic search variable selection (BSSVS) is a common form of this sort of prior.
We have some number of variables we think might contribute to, say, geographic dispersal, but we doubt they all do.
The prior on the coefficients in BSSVS is a mixture which breaks down into two parts.
First, a 0/1 component that determines whether the variable has any effect (an indicator variable).
Then, a conditional prior for when the coefficient is not 0.
BSSVS makes use of what are called spike-and-slab priors, which are closely related to the focus here: shrinkage priors.

### What is a shrinkage prior?
Shrinkage priors are ways of saying "I think this parameter has a particular value, but perhaps it's actually different."
For regression models, the fixed value is generally 0, which says the covariate has no effect and essentially removes the variable from the model.
But unlike the mixture approach that spike-and-slab distributions take, shrinkage priors are fully continuous distributions.
Shrinkage priors give up the possibility of setting any parameter to exactly some value (the probability that a continuous random variable $X$ takes any exact value $x$ is 0).
But in return, we have a fully-continuous variable, which means we gain a number of convenient properties.
Most notably, it makes derivatives (gradients) much easier, so we can use gradient-based sampling algorithms, which are quite efficient.

Like a spike-and-slab mixture, though, a good shrinkage prior should be spiky and have reasonably large tails.
The spike is the prior mass which accounts for "the parameter is probably roughly 0 (or other value)."
The tails, meanwhile, allow for the variable to take on a relatively wide range of values if it isn't roughly 0.
(We will discuss later how we can reign in the tails.)

Many shrinkage priors can be represented as a scale mixture of normal distributions (meaning a mixture over the scale/variance parameter rather than the location/mean parameter), which can be convenient, both for understanding the distribution and for sampling.

### The Horseshoe
The Horseshoe prior [(Carvalho *et al*, 2009)](http://proceedings.mlr.press/v5/carvalho09a/carvalho09a.pdf) is a commonly-employed shrinkage prior, used for things from sparse regression to [coalescent models](https://onlinelibrary.wiley.com/doi/abs/10.1111/biom.13276). The distribution is characterized by two parameters.

- The location (which we will ignore and assume to be 0)

- The (global) scale parameter, $\sigma$

The Horseshoe does not have a closed form for its probability density function.
Nor indeed for it's cumulative density function.
Instead, we must always appeal to its representation as a mixture of Normal distributions.
Specifically, if $X \sim \text{Horseshoe}(0,\sigma)$, we can instead write the following hierarchical model which is equivalent:

- $\lambda \sim \text{Cauchy}^+(0,1)$

- $X \mid \lambda \sim \text{Normal}(0,\lambda^2\sigma^2).$

Where Cauchy$^+$ is used to mean a half-Cauchy (positive-values-only) distribution.

The variable $\lambda$ is called a local scale variable, while in this representation we call $\sigma$ the global scale.
This language is used because we generally have not one variable $X$ but a vector $X_1,\dots,X_n$ which are iid Horseshoe, $X_i \sim \text{Horseshoe}(0,\sigma)$.
When we write out the mixture distribution, we get $X_i \mid \lambda_i,\sigma_i \sim \text{Normal}(0,\lambda_i^2\sigma^2),$ with $\lambda_i \sim \text{Cauchy}^+(0,1)$.
So, all of these IID variables share the global scale, but each has its own local scale.

The Horseshoe is _very_ spiky, and has _very_ fat tails.
The mixture of normals representation helps make this clear.
A Cauchy distribution has very fat tails, so when you get a very large $\lambda$ you get a very large variance.
These tails mean that the Horseshoe also does not have a defined mean or a finite variance (the variance is either undefined or infinite, but in practice which it is makes little difference).
There is also a non-insignificant probability that $X$ is near 0, in which case we have a variance-0 Normal which has infinite density.
In fact, the Horseshoe has an infinite density at 0 as well, though it is a proper probability distribution and integrates to 1 (isn't math fun?).

The choice of the global scale parameter can be important, but there may not be a lot of information to set it.
As a way around that, we can put a prior on the global scale and infer it.
This gives us additional wiggle room in case we don't get the value quite right.
A convenient form of prior is the Cauchy$^+$ distribution, in which case we can use a Gibbs sampler to update $\boldsymbol{\lambda}$ and $\sigma$.

### The Bayesian Bridge
The Bayesian Bridge [(Polson _et al._ 2011)](https://arxiv.org/abs/1109.2279) is characterized by up to three parameters.

- The location (which we will ignore and assume to be 0)

- The scale parameter, $\sigma$

- The exponent/power parameter, $\alpha$

Unlike the Horseshoe, the Bayesian Bridge has a closed-form probability density.
Assuming the distribution is centered at 0, we get $p(x) \propto \text{exp}(-|x/\sigma|^\alpha)$.
If we have $\alpha = 2$, then this is the Normal distribution, while at $\alpha = 1$ it is the Laplace (or double-exponential) distribution.
When $\alpha$ gets small, however, this distribution gets a decently-sized spike at 0.

While the Bayesian Bridge has a closed form density, inference via MCMC works best when we use a mixture representation.
Specifically (like the Horseshoe), the Bayesian Bridge can be written as a scale mixture of Normal distributions.
However, the distribution on the local scales $\boldsymbol{\lambda}$ is quite ugly.

As with the Horseshoe, we generally want to infer the global scale parameter.
And, like with the Horseshoe, the easy choice is the distribution which allows a convenient Gibbs sampler.
In this case, that means putting a Gamma prior on $\phi = \sigma^{-\alpha}$ rather than directly parameterizing $\sigma$.
Using a Gamma(shape=s,rate=r) distribution on $\phi$ implicitly places a prior on $\sigma$ which is proportional to $\alpha \tau^{-\alpha s -1} e^{-ry^{-\alpha}}$.
In the limit as $r,s \to 0$, this density is proportional to $\sigma^{-1}$ which is the reference prior [(Nishimura and Suchard, 2022)](https://projecteuclid.org/journals/bayesian-analysis/advance-publication/Shrinkage-with-Shrunken-Shoulders--Gibbs-Sampling-Shrinkage-Model-Posteriors/10.1214/22-BA1308.full).

The Bayesian Bridge is also known as the Generalized Normal Distribution, and you can play around with it in `R` using the package `gnorm`.

### Which distribution should I use in practice?
To be clear, both the Bayesian Bridge and Horseshoe are workable shrinkage priors.
But there are reasons one might favor one over the other.
The short version is that the Horseshoe is far spikier and has fatter tails than the Bayesian Bridge.
In some ways, this is a point in favor of the Horseshoe.
On the other hand, those extreme tails can leave imprints on the posterior distribution when there isn't a _lot_ of information in the data.

#### Points in favor of the Horseshoe

The Horseshoe is also very easy to draw IID samples from, as the Cauchy distribution is widely available in software (and not that hard to implement by hand if needed).
Sampling from the Bayesian Bridge is possible, but implementations of the Generalized Normal Distribution are not particularly common, nor are implementations of exponentially-tilted stable distributions.
I think it is also easier to contemplate a $\text{Cauchy}^+$ distribution on the global scale $\sigma$ than a $\text{Gamma}$ on $\sigma^{-\alpha}$.

#### Points in favor of the Bayesian Bridge
When using either the Bayesian Bridge or Horseshoe as a prior, we must consider their behavior in MCMC.
The mixing for the Horseshoe's global scale parameter can be quite painful in practice, regardless of whether you use a Gibbs sampler or Hamiltonian Monte Carlo.
The Bayesian Bridge global scale mixes much more rapidly, possibly because the local scales can be analytically integrated out by using the closed-form representation of the density function.


### Who regularizes the regularizers?
As it turns out, like all good things, there is a limit to how fat you might want the tails of a shrinkage prior.
The sheer prior mass of the tails can be hard to overcome and can leave posterior distributions that are much wider than they need to be.

Shrinkage distributions (and spike-and-slab distributions) are forms of Bayesian regularization.
It turns out that we can regularize our shrinkage priors in order to avoid some of the unwanted tail behavior.
Specifically, we can make it so that the conditional distribution on $x_i$ is Normal with variance,
\[
 \Big(\frac{1}{\xi^2} + \frac{1}{\sigma^2 \lambda_i^2}\Big)^{-1}
\]
where $\xi$ is a parameter known as the slab width.
When $\sigma$ or $\lambda_i$ get large, the variance will tend towards $\xi^2$, and outside of $-\xi < |x_i| < \xi$, the tails of the distribution will look like those of a Normal(0,$\xi^2$).
Thus, $\xi$ defines the width for which this regularized shrinkage distribution acts like the standard version.

Nishimura and Suchard (2022) provide a clever setup for this regularization.
They add a fictitious vector $\boldsymbol{z}$ into the model and define $z_i \sim \text{Normal}(x_i,\xi^2)$.
Setting all $z_i = 0$ then induces a conditional Normal on $x_i$ with the above regularized variance.
By sneaking this in through the likelihood, their approach leaves the Gibbs samplers for $\boldsymbol{\lambda}$ and $\sigma$ intact, which is entirely convenient.
One can also use this fake-data formulation directly in software like stan or RevBayes to tamp down on the tails of your shrinkage prior of choice (so long as you can represent it as a scale mixture of normals).

#### Things that go bump in the night
One oddity does crop up when using these regularized shrinkage priors, and that is how to consider the prior itself.
The fake data $\boldsymbol{z}$ are treated mathematically as part of the likelihood but are in fact part of the prior.
This means that the _effective_ priors on $\boldsymbol{\lambda}$ and $\sigma$ are altered from those which are specified.
The effective prior on $\phi = \sigma^{-\alpha}$ can be quite notably different from the specified Gamma.

This discrepancy between the nominal and effective priors is not a serious issue in practice unless one wishes to estimate marginal likelihoods for the model.
As Gibbs samplers are already potential problems there (they can't handle the heat), one might as well directly use the regularized variance when setting up the model.
Alternately, one should consider fully embracing the philosophy of Bayesian model averaging and Bayesian regularization.
By building a large model which contains all the parameters of interest, and by regularizing that model, there should in general not be a serious need to compare models by marginal likelihoods.
Just look at the posterior distributions on parameters!

And, if you really, really want a Bayes Factor, you can generally obtain one without brute-force computing marginal likelihoods.
For both Bridges and Horseshoes, testing directional hypotheses with Bayes factors is straightforward (though it may require prior samples).
With a Bridge prior you can test any point-null of interest (like, "this parameter is 0" or "the difference between these parameters is 0") with a Savage-Dickey density ratio (and some kernel density estimation).
For the Horseshoe, which has infinite density at 0, testing point nulls at or near 0 will go badly.
Though, in general, one can quickly see the amount of evidence there is for a parameter being non-zero by inspecting the posterior distribution.
If it's a spike at 0, then the parameter is effectively 0.
If there's no spike near 0, you've got strong evidence for the parameter being non-zero.
And if you get a bimodal distribution (which is somewhat common), then there is evidence for a non-zero parameter but it is not overwhelming.
We'll ignore all those posteriors with a spike and a long arm on one side, because I still don't know what to make of them (very weak evidence?).


### All shrinkage is local
I want to close with one last point about these global-local mixture-of-Normals models.
Which is that when we apply the right version of them to time-series models, we end up with a property we call local adaptivity [(Faulkner and Minin, 2017)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5942601/).
Local adaptivity is an ability to capture rapid change in some regions and slower (or no) change in others.
The local scales can really help us see why this happens (or doesn't).

The Horseshoe, with its $\text{Cauchy}^+$ local scales, is locally adaptive.
The Laplace distribution, a special case of the Bayesian Bridge (namely with $\alpha = 1$), is not.
For the Laplace, the distribution on the local scales is the distribution on the [square root of an Exponential random variable](https://rss.onlinelibrary.wiley.com/doi/pdf/10.1111/j.1740-9713.2018.01185.x).
There are perhaps two notable differences between these distributions.
The $\text{Cauchy}^+$ has some pull towards 0, because that is its mode, and it has a very fat tail.
The squareroot-Exponential distribution has a non-zero mode and a relatively mild tail.
The Laplace, then, doesn't have a lot of wiggle-room for the local scales to be a lot bigger or smaller than any other, which means that the overall variability will be determined by the global scale $\sigma$.
But the Horseshoe allows some of those local scales to be much, much bigger than the rest.
Those can then capture regimes of rapid change while elsewhere the global scale $\sigma$ determines the variability.

I am not sure where exactly the Bayesian Bridge shifts from having no local adaptivity to having some.
It's definitely below $\alpha = 1$, and probably somewhere near where the variance (which is $\sigma^2 \, \Gamma(3/\alpha) \, / \, \Gamma(1/\alpha)$) explodes.
I am also not sure how shrunken-shoulder regularization plays into this property either.
It could make for an interesting investigation.
