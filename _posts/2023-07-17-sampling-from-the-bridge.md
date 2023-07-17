---
title: "Sampling from the Bayesian Bridge (with shrunken-shoulder)"
author_profile: true
layout: single
---

For people not named Andrew Magee, the mission statement of this post is "how do we draw samples from the Bayesian Bridge distribution when applying shrunken-shoulder regularization?"[^1]
I am far too disorganized to have a "series" of posts on any particular topic, but if I weren't, this would be the second in the series on shrinkage priors begun [here](https://afmagee.github.io/shrinkage-priors/).
That is to say, I'm not going to introduce the Bridge from scratch in this post.[^2]

## The Bayesian Bridge with shrunken shoulders
The Bayesian Bridge with shrunken shoulders is a modification of the Bayesian Bridge distribution which bullies the tails into behaving themselves better.
The _way_ it does this is by modifying the global-local mixture representation of the Bayesian Bridge such that, asymptotically, the tails behave as a $\text{Normal}(0,\zeta^2)$ distribution, where $\zeta$ is called the slab width.
This Normal has thinner tails than the mixture otherwise would.

Let us move to the multivariate case, because usually we have multiple Bridge-distributed parameters.
We'll call them $\boldsymbol{\beta}$, following Nishimura and Suchard (2023) who discuss regression models.
Nishimura and Suchard show that we can accomplish the shrunken-shoulder regularization in a convenient way by smuggling in some fake data $\boldsymbol{z}$ with all $z_i = 0$.
They even show what the prior distribution is proportional to in this case.
Conditioned on the fake data $\boldsymbol{z}$ and the global scale $\tau$[^3], the shrunken shoulder distribution on $\beta_i$ is $\text{Pr}(\beta_i \mid \tau, \zeta, z_i = 0) \propto \text{Pr}(\beta_i \mid \tau) \exp(-\beta_i^2 / (2 \zeta^2))$, where $\text{Pr}(\beta_i \mid \tau)$ is the un-shrunk Bridge density.

Given all this, sampling from $\text{Pr}(\beta_i \mid \tau, \zeta, z_i = 0)$ should be easy, right?
We know the density we want to sample from, at least up to a constant.
We know two ways to represent it, a product of a closed-form Bridge density and a Normal or a mixture of Normals.
And using the shrunken-shoulder as a prior is straightforward and works efficiently with Gibbs sampling (that was, after all, the point of Nishimura and Suchard).
Well, as seen below, it took me some thinking to make this work.

### Why do you want to do this at all, you idiot?
A very good question one might want to ask right now is why on Earth I want to be able to sample from this distribution.
Because this seems like a solution in search of a problem.

From a general perspective, I'd say it's important to be able to sample from our priors so as to understand them.
That's trite, but I think there's truth to it.
Prior simulations are very helpful for understanding models, and resorting to MCMC, where you have to worry about convergence and effective sample sizes, is a pain.

I've also been curious about what happens with increasing dimensions of Bridge priors.
I have observed that the cumulative effect of $\boldsymbol{z}$ can apparently get quite strong.[^4]
As I am not mathematically-inclined enough to study this rigorously, so I figured I might try to look at $\text{Pr}(\tau \mid \boldsymbol{z} = 0)$ for increasing dimensions.
But sampling from _that_ is a massive pain in its own way, and the neatest thing I could conceive of was a multi-stage procedure.
First, draw $\tau \sim \text{Pr}(\tau)$.
Then, draw $\boldsymbol{\beta} \sim \text{Pr}(\boldsymbol{\beta} \mid \tau, \zeta, \boldsymbol{z} = 0)$.
Lastly, run one Gibbs update to draw $\tau \sim \text{Pr}(\tau \mid \boldsymbol{\beta})$.
Do this many times and we should marginalize out $\boldsymbol{\beta}$ and get the distribution we (I) want.[^5]

## A note on the un-shrunken-shoulder Bridge
Without shrunken shoulders, the Bridge is also known as the Generalized Normal Distribution, and you can play around with it in `R` using the package `gnorm`.
You can sample from it with that, or you can check [Wikipedia](https://en.wikipedia.org/wiki/Generalized_normal_distribution) (you want the "symmetric version") for it's quantile function and use inverse transform sampling.
Or you can follow links from Wikipedia or the R package to a very [lovely writeup by Maryclare Griffin that explains multiple ways to sample from it](https://cran.r-project.org/web/packages/gnorm/vignettes/gnormUse.html).

## The obvious solutions

### MCMC
I'm just going to get this out of the way.
Yes, we could sample this with MCMC.
No, I don't want to.
I don't want to worry about MCMC behaving itself to describe something I should be able to do directly.

### Inverse transform sampling
The inverse transform approach is to draw $u ~ \text{Uniform}(0,1)$ then set $x := F^{-1}(u)$ where $F^{-1}$ is the quantile function of the distribution.
There are often more efficient ways to sample from a distribution than it.
I would happily use it, but nobody has worked out the quantile function of the shrunken-shoulder Bridge yet and whoever does won't be me.
So I can't do this either.

## Take 1: Sampling from the scaled mixture of normals
The first thing I considered was just sampling from the representation of the Bridge as a scaled mixture of Normals.
This is, after all, how one painlessly generates samples from the Horseshoe (a common shrinkage prior)
The problem is the mixing distribution.
It's awful.

Let's be a bit more precise.
We can write the shrunken-shoulder Bridge conditionally as,

$\beta_i \mid \tau, \zeta, z_i = 0 \sim \text{Normal}(0,(1/{\zeta^2} + 1/(\tau^2 + \lambda_i^2))^{-1})$.

Where the variable $\lambda_i$ is called the local scale.
The shrunken-shoulder Horseshoe would have $\lambda_i$ drawn from a half-Cauchy.
For the Bridge, it's a tilted one-sided stable distribution.
What is that?
Good question, it's a variable whose distribution includes a $1/x$ factor in front of the one-sided stable distribution $\text{Pr}(x)$.
What's a one-sided stable distribution?
[Something that doesn't in general have an analytical form for any of the things we'd like it to.](https://en.wikipedia.org/wiki/Stable_distribution#One-sided_stable_distribution_and_stable_count_distribution)

If you follow enough papers you might just be able to find an implementation of this, or create one.
But surely there's an easier way, no?

## Take 2: Rejection sampling
The solution I have landed on is to use rejection sampling.
Initially, this was more or less motivated by looking at the (unnormalized) density of the shrunken-shoulder Bridge and saying "hey, that's two distributions multiplied together, what if we sampled according to one and rejected according to the other?"
Let's look at the pieces involved in this: the density at hand and rejection sampling.

### The shrunken-shoulder Bridge, again
Lets follow (more or less) along with Nishimura and Suchard (page 371, around Equation 2.3) so we can start constructing the density we need.[^6]
Or rather, let's do it ourselves since we already stated what it was earlier.
The density of interest is defined by the following model,

$\beta_i \sim \text{Bridge}(\text{location} = 0,\text{scale} = \tau,\text{shape} = \alpha)$

$z_i \sim \text{Normal}(\beta_i,\zeta^2)$.

In addition to the slab width $\zeta$, we have the (global) scale $\tau$ and the shape (called the exponent in some contexts) $\alpha$ parameters of the Bridge.

We can write the joint density on $\beta_i, z_i$ out as the product of a Bridge distribution on $\beta_i$ and a Normal distribution on $z_i$.

$\text{Pr}(\beta_i, z_i \mid \tau) \propto \text{Bridge}(\beta_i; 0,\tau,\alpha)\text{Normal}(z_i; \beta_i,\zeta^2)$.

Dropping Normalizing constants, we get,

$\text{Pr}(\beta_i, z_i \mid \tau) \propto \exp(-\lvert \beta_i/\tau \rvert^{\alpha}) \exp(-(z_i - \beta_i)^2 / (2 \zeta^2))$.

And now conditioning on $z_i = 0$ we're left with,

$\text{Pr}(\beta_i \mid \tau, z_i = 0) \propto \exp(-\lvert \beta_i/\tau \rvert^{\alpha}) \exp(-\beta_i^2 / (2 \zeta^2))$.

### Rejection sampling, a refresher
When rejection sampling, we have a target distribution $f(x)$ which we want samples from (here the shrunken-shoulder Bridge) and a proposal distribution $g(x)$ which we can draw samples from.
We need to be able to make sure that we can appropriately scale $g(x)$ so that $f(x)$ is always within it.
We can rephrase this by saying we need to find some $M$ such that $M \geq f(x) / g(x)$.

Given this, we can implement rejection sampling in a simple loop.
1. Draw $x \sim g(x)$ and $u \sim \text{Uniform}(0,1)$.
2. If $u < f(x)/(M \times g(x))$ return $x$, else go to 1.

<!-- Importantly, $M$ tells us about the efficiency of this procedure.
If $M = 1$ then we're proposing from the distribution we want and we keep all samples.
In general, it will take $M$ draws to return a sample from $f(x)$, so we want $M$ as close to 1 as we can get it. -->

Now, in this case we've worked out $f(x)$ only up to a constant.
But as outlined [here](https://regularize.wordpress.com/2017/02/13/sampling-from-unnormalized-distributions/) constants come out in the wash, so we don't actually need to work with the normalized version of $g(x)$ either, which will be convenient.

### Rejection sampling from either component 
We want samples from $f(\beta_i) \propto \exp(-\lvert \beta_i/\tau \rvert^{\alpha}) \exp(-\beta_i^2 / (2 \zeta^2))$.
This distribution is the product of two things, let's call them the Bridge component and the shoulder component.
That's nice for two reasons.
For one, if we take $g(\beta_i)$ to be either component, we can see that the (unnormalized) densities will satisfy $f(x) \leq M g(x)$.
Further, we know how to draw samples from either component.
We can sample from $g(\beta_i) \propto \exp(-\lvert \beta_i/\tau \rvert^{\alpha})$, which is the (unnormalized density of the) un-shrunk Bridge, or $g(\beta_i) \propto \exp(-\beta_i^2 / (2 \zeta^2))$, which is the (unnormalized density of the) Normal distribution.
<!-- Let's call the two terms in the shrunken-shoulder Bridge distribution the Bridge component and the shoulder component. -->

Let's start with sampling from the un-shrunk Bridge, taking $g(\beta_i) \propto \exp(-\lvert \beta_i/\tau \rvert^{\alpha})$.
We need to get the bound $f/g$, $M = \max(f/g)$.

$M = \max([\exp(-\lvert \beta_i/\tau \rvert^{\alpha}) \exp(-\beta_i^2 / (2 \zeta^2))] / \exp(-\lvert \beta_i/\tau \rvert^{\alpha}))$

$M = \max(\exp(-\beta_i^2 / (2 \zeta^2))) = 1$.

What about doing this the other way, and sampling from a Normal, taking $g(\beta_i) \propto \exp(-\beta_i^2 / (2 \zeta^2))$?

$M = \max([\exp(-\lvert \beta_i/\tau \rvert^{\alpha}) \exp(-\beta_i^2 / (2 \zeta^2))] / \exp(-\beta_i^2 / (2 \zeta^2)))$

$M = \max(\exp(-\lvert \beta_i/\tau \rvert^{\alpha})) = 1$.

Hang on a minute, you might be asking, what gives?
Wikipedia tells us that $M$ can't be 1 because that only happens when we're sampling from the correct distribution in the first place?
What did we do wrong?
As far as I can tell, working with the unnormalized versions of the distributions has obscured the truth of the matter.
It's less that $M$ has come out in the wash and more that we've accidentally smuggled another constant into it, the normalization constant for $f()$.
So, we're not really looking at $M$ here but something like $M/C$ (to use the terminology from [here](https://regularize.wordpress.com/2017/02/13/sampling-from-unnormalized-distributions/)).

Charging boldly forward, let's try these out!
(R code is at the end of the post to implement both of these.)
What we can find playing around with the performance of these two approaches makes some intuitive sense.
When the shoulder component dominates the shrunken-shoulder Bridge density, we do best (get the most efficient results) by proposing from that shoulder (Normal) distribution.
Because most of the target distribution looks like that Normal.
When the Bridge component dominates the shrunken-shoulder Bridge density, the reverse is true.

How much better is better?
When the scale is much much smaller than the slab width, we get almost perfect performance proposing from the Bridge component, but proposing from the Normal requires hundreds of more tries.
When the scale is much much larger than the slab width, the pattern largely reverses.
The point of balance seems to be in favor of the Normal, with the Normal performing better for slab widths within maybe an order of magnitude of the scale.[^7]
At least, that's true when the shape is near the 1/4 that I default to using.
The shape can change the balance of power greatly.

At this point, we might be satisfied and call it a day.
But wouldn't it be nice if we had a general-purpose approach?
I've used a lot of priors on $\tau$ that imply a pretty large range of plausible values, with a lot of mass on small values but the possibility of very large values too.

### Picking a component to propose from
If we could solve for the normalization constant of the shrunken-shoulder Bridge, we could pretty clearly choose between regimes by simply figuring our which proposal has a better $M$.
But that isn't particularly straightforward to do, so we're left contemplating other approaches.

Beyond the simplicity of being able to directly choose based on the rejection rate, the reality of defining the ideal proposal is not trivially simple.
The scale relative to the slab width is clearly important, but so is the shape.
The shape likely plays a role both in terms of the tails of the proposal distribution and the shape of the target.
The bridge can get pretty flat when $\alpha$ gets small, which both means our proposal has way too much mass in the tails (lots of rejecting) and that in the middle the dominant contribution should be from the shoulder.

However, I have a dumb rule to propose.
For a given $\tau,\alpha,\zeta$, determine how much of the unshrunken Bridge mass lies in $[-\zeta,\zeta]$.
If it is a majority, then propose from the Bridge.
If it is not, then propose from the shoulder.
This seems to work far better than it should.
If I fix $\alpha,\zeta$ and sample $\tau$ from the usual prior[^8], it very strongly outperforms fixing either proposal.
It also rarely seems to require all that many attempts.

### Other proposals for rejection sampling
Instead of trying to pick between these two options, we could try a different proposal distribution.
As someone much smarter than me suggested, we could simply sample from a _mixture_ of the Bridge and shoulder components.
Our prior distribution is a mixture, with Bridge-like and Normal-like characteristics, so if we combine those cleverly we could get a proposal that does the same, but which we can sample from.

Consider a standard mixture distribution with proportion $p$ of mass on the Bridge and $(1-p)$ on the shoulder.
The rough intuition for what we'd hope the performance would be like is that at worst we wait, on average, twice as long to draw from the mixture component that works better for this parameter regime.
This should never outperform the "correct" choice, but could mean we don't have to worry about needing hundreds (or thousands) of attempts if we've picked badly.

Since a mixture density requires us to average the two density functions, we'll need to start caring about normalizing constants again, at least partly.

We'll use a 50-50 mixture of the Bridge and shoulder components, such that,

$g(\beta_i) = 1/2 * (\alpha / (2 \times \tau \times \Gamma(1/\alpha)) * \exp(-\lvert \beta_i/\tau \rvert^{\alpha})) + 1/2 * (1 / (\zeta \times \sqrt{2 \pi}) * \exp(-(\beta_i)/(2 \zeta^2))$.

Let's clean this up to make things a bit easier to understand, and use $Z_B$ for the Bridge normalizing constant and $Z_N$ for the Normal.
That gives us,

$g(\beta_i) = 1/2 * (1/Z_B * \exp(-\lvert \beta_i/\tau \rvert^{\alpha})) + 1/2 * (1 / Z_N * \exp(-(\beta_i)/(2 \zeta^2))$.

Again, our target is,

$f(\beta_i) \propto \exp(-\lvert \beta_i/\tau \rvert^{\alpha}) \exp(-\beta_i^2 / (2 \zeta^2))$,

so the ratio $f/g$ is,

<!-- $f(\beta_i)/g(\beta_i) \propto \exp(-\lvert \beta_i/\tau \rvert^{\alpha}) \exp(-\beta_i^2 / (2 \zeta^2)) / (1/2 * (1/Z_B * \exp(-\lvert \beta_i/\tau \rvert^{\alpha})) + 1/2 * (1 / Z_N * \exp(-(\beta_i)/(2 \zeta^2)))$ -->

$f(\beta_i)/g(\beta_i) \propto 2 * \exp(-\lvert \beta_i/\tau \rvert^{\alpha}) \exp(-\beta_i^2 / (2 \zeta^2)) / ((1/Z_B * \exp(-\lvert \beta_i/\tau \rvert^{\alpha})) + (1 / Z_N * \exp(-(\beta_i)/(2 \zeta^2)))$.

Wolfram-Alpha tells us that the derivative (with respect to $\beta_i$) of this function is pretty gnarly, and it can't find a (unique) maximum.
So let's try to find a handwavy approach to a worse bound.

We can rewrite the requirement for $M$ as $f(x) \leq M g(x)$.
We can rewrite $g$ as,

$g(\beta_i) = 1/2 * (1/Z_B * g_B(\beta_i) + 1/2 * (1 / Z_N * g_N(\beta_i))$.

We've already seen above that $g_B(\beta_i)$ and $g_N(\beta_i)$ satisfy the requirements for rejection sampling.
So if we choose either $M = 2 Z_B$ or $M = 2 Z_N$, then we must have that $f(\beta_i) \leq M g(\beta_i)$ because we know that $f$ is less than or equal to one of the components.

Now, we can try this too.
In many cases it appears to avoid the worst-case scenarios of either proposal.
However, it seems to perform much closer to simply using the Bridge as a proposal than splitting the difference.
In some cases (larger values of $\alpha$ and $\tau$) it does worse than either!
And in no case does it beat the simple majority-mass-rules decision for picking the proposal based on the parameters.

<!-- $-2 Z_B Z_N / \zeta^2 (\alpha Z_B \zeta^2 \exp(|\beta_i/\tau|^{\alpha}) + Z_N \beta_i^2 \exp())/()$ -->

## Haven't you written enough?
It is now far, far beyond time to wrap this up.
In summary, the shrunken-shoulder Bridge lacks the closed-form density that makes the usual version somewhat nice, and can be a royal pain to deal with.
Drawing samples from this can be accomplished via rejection sampling from either the unshrunken Bridge, a Normal, or a mixture of both.
A generally efficient approach over a range of parameters appears to be to dynamically choose whether to propose from the Bridge or the Normal based on $\tau$, $\alpha$, and $\zeta$.

&nbsp;

&nbsp;

&nbsp;

&nbsp;
# Appendix: code
I know that this would be slicker if it were linked to actual code on GitHub, or was a downloadable file.
But this is what I have time to do, so it's what I'm doing.

    # Draw from a Shrunken-shoulder Bayesian Bridge by rejection-sampling, where proposal
    #   distribution is a Normal(mean=0,sd=slabWidth)
    drawBBRejectionProposeNormal <- function(scale, shape, slabWidth, return.count=FALSE) {
    value <- NULL
    done <- FALSE
    count <- 0
    while (!done) {
        count <- count + 1
        prop <- rnorm(1,0,slabWidth)
        logU <- log(runif(1,0,1))
        logAR <- -abs(prop/scale)^shape
        if (logU < logAR) {
        value <- prop
        done <- TRUE
        }
    }
    if (return.count) {
        return(count)
    }
    return(value)
    }

    # Draw from a Shrunken-shoulder Bayesian Bridge by rejection-sampling, where proposal
    #   distribution is a BayesianBridge(location=0,scale=scale,shape=shape)
    drawBBRejectionProposeBridge <- function(scale, shape, slabWidth, return.count=FALSE) {
    value <- NULL
    done <- FALSE
    count <- 0
    while (!done) {
        count <- count + 1
        prop <- rgnorm(1,0,scale,shape)
        logU <- log(runif(1,0,1))
        logAR <- -(prop^2)/(2*slabWidth*slabWidth)
        if (logU < logAR) {
        value <- prop
        done <- TRUE
        }
    }
    if (return.count) {
        return(count)
    }
    return(value)
    }

    # Draw from a Shrunken-shoulder Bayesian Bridge by rejection-sampling, where proposal
    #   distribution is either a Bridge or a Normal, decided based on probability mass
    #   of Bridge between +/- slabWidth
    drawBBRejectionProposeDecision <- function(scale, shape, slabWidth, return.count=FALSE) {
    bridge_prob_in_tails <- 2 * (pgnorm(slabWidth,0,scale,shape) - 0.5)
    if ( bridge_prob_in_tails > 0.5 ) {
        return(drawBBRejectionProposeBridge(scale=scale, shape=shape, slabWidth=slabWidth, return.count=return.count))
    } else {
        return(drawBBRejectionProposeNormal(scale=scale, shape=shape, slabWidth=slabWidth, return.count=return.count))
    }
    }

    # unnormalized density for shrunken-shoulder bridge
    dBBunnorm <- function(x, scale, shape, slabWidth,log=FALSE) {
    # page 2, Nishimura and Suchard (2022)
    dens <- -log(scale) - abs(x/scale)^shape
    # add slab effect, eqn just before 2.4, Nishimura and Suchard (2022)
    dens <- dens - (x^2)/(2*slabWidth^2)
    if (log == FALSE){dens <- exp(dens)}
    return(dens)
    }

    # Normalizing constant of un-shrunken-shoulder Bridge (gnorm) with scale scale and shape shape.
    zBBunshrunk <- function(scale,shape,log=FALSE) {
    Z <- -(log(shape) - (log(2*scale) + lgamma(1/shape)))
    if (!log) {
        Z <- exp(Z)
    }
    return(Z)
    }


    # A helper function
    weightedLogSumExp <- function(x,w) {
    m <- max(x)
    x <- x - m
    return(log(sum(exp(x)*w)) + m)
    }

    # A 50-50 mixture of a BayesianBridge(location=0,scale=scale,shape=shape) and a Normal(mean=0,sd=slabWidth)
    mixDens <- function(x, scale, shape, slabWidth, log=FALSE) {
    d <- sapply(x,function(x_){
        l1 <- dgnorm(x_,0,scale,exponent,log=TRUE)
        l2 <- dnorm(x_,0,slab,log=TRUE)
        weightedLogSumExp(c(l1,l2),c(0.5,0.5))
    })
    if (!log) {
        d <- exp(d)
    }
    return(d)
    }

    # Draw from a Shrunken-shoulder Bayesian Bridge by rejection-sampling, where proposal
    #   distribution is a Bridge-Normal mixture
    drawBBRejectionProposeMixture <- function(scale, shape, slabWidth, return.count=FALSE) {
    lnM <- log(2) + min( c(log(slabWidth * sqrt(2*pi)),zBBunshrunk(scale, shape, log=TRUE)) )
    value <- NULL
    done <- FALSE
    count <- 0
    while (!done) {
        count <- count + 1
        prop <- NA
        if (runif(1,0,1) < 0.5) {
        prop <- rgnorm(1,0,scale,shape)
        } else {
        prop <- rnorm(1,0,slabWidth)
        }
        logU <- log(runif(1,0,1))
        logAR <- dBBunnorm(prop, scale, shape, slabWidth, TRUE) - mixDens(prop,scale,shape,slabWidth,log=TRUE) - lnM
        if (logU < logAR) {
        value <- prop
        done <- TRUE
        }
    }
    if (return.count) {
        return(count)
    }
    return(value)
    }

[^1]: For people who are named Andrew Magee, the mission statement is "write up the damn math somewhere you won't forget it this time!"
[^2]: Not that I'm claiming I did a great job in the other post either.
[^3]: Sorry, I know that was $\sigma$ in the other post.
[^4]: At least, strong enough to overwhelm the sort of time-series likelihoods I've worked with.
[^5]: The distribution I think I want, that is. 
[^6]: I am not attempting to be general here, this is only for the Bridge and I am dispensing with local scales, because we know how to analytically integrate them out for the Bridge.
[^7]: Please do not take this as a hard fast rule, I'm making it up as I go along. There is no math and no extensive simualation here.
[^8]: A power-transformed Gamma, or if you'd prefer a Gamma distribution on $\phi^{-\alpha}$. This happens to be conjugate so it's convenient. Matters of spiritual correctness are left to the reader.