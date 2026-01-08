---
title: "Can you bootstrap that?"
author_profile: true
layout: single
---

I like a good crazy idea[^1].
I also, clearly, have a problem with answers in search of questions[^2].
In "a little bit of column A, a lot of column B": bootstrapping branching times in a time-calibrated phylogeny.
This is only a little crazy, because in time-varying phylogenetic birth-death models where all taxa are sampled at the present, [divergence times are IID.](https://academic.oup.com/bioinformatics/article-lookup/doi/10.1093/bioinformatics/btt153).
Why is it an answer in search of a solution?
Let's take a look.

## The good

The good news, such as it is, is that nonparametric bootstrapping of divergence times does seem to work.
I compared it to the parametric bootstrap, which is a similarly widely-scoped hammer for the problem.
As you can see, they both achieve basically the correct coverage in a simple constant-rate birth-death model scenario.

![Coverage of birth and death rates from a constant-rate birth-death model simulation and analysis of trees of 100 tips.](../assets/images/posts/2026-01-08_bt_bootstrap_coverage.png)


In general, the nonparametric bootstrap is probably a bit faster than the parametric bootstrap.
Univariate likelihood profiling could perhaps be faster still, but, well, it's univariate[^3].
[As tested](#code-appendix), I used [TESS](https://github.com/hoehna/TESS), which is very fast for simulating birth-death processes, for the parametric bootstrap, and R's built-in `sample()` for the nonparametric bootstrap.
TESS numerically integrates the CDF of divergence times and then draws from that CDF.
This is, minimally, one more step than simply drawing the indices of the samples for the nonparametric bootstrap.
The number of random numbers needed in both case is the same (other simulation-based approaches would need more).
I can't entirely explain why, in practice, with 250 bootstrap replicates the nonparametric bootstrap was nearly two times faster than the parametric.
It could just be how much effort has gone into R's `sample()` over the years, or that drawing from a numerically-approximated CDF inherently requires more comparisons than drawing uniform random integers.

## The bad

Nonparametric bootstrapping of divergence times, as an approach, is actually rather limited in application scope.
That might sound weird, given how few assumptions it would seem to rely on.
But the assumptions it does rest on are not small.

The explicit assumption is that branching times are IID.
I won't spend too much ink on this, as it is the other, implicit, assumption that is the real death blow.
I do think we can push past models with tips only at the present[^4], and serially-sampled birth death models are important.
I do not have a strong intuition on whether times still be IID if we think there are, say, arbitrary waiting time distributions for births and deaths.
Quite possibly?

The implicit assumption here is that branching times are all we care about.
After all, that's all we're bootstrapping.
But when birth and/or death rates change along the tree, it's the tree that matters, not just the divergence times.
That means trait-dependent models are out, or even just models that let the birth and death rates vary from branch to branch.
Such SSE family models comprise a huge portion of the usefulness of phylogenetic birth-death models[^5] in practice.

It's also worth noting that the nonparametric bootstrap is still an asymptotic approach.
So it's not liable to be particularly helpful if you only have a tree with ten tips.

## The ugly

Nonparametric bootstrapping of divergence times is completely unnecessary in practice.
Roughly, I'd say, the nonparametric bootstrap is useful when it allows us to do something we could not otherwise do satisfactorily.
Many times this is about our ability to remove assumptions from the model.
Other times this is simply that we have no other way to address uncertainty about our estimates.

I can't see ways that this allows us to remove assumptions from our modeling.
In other statistical arenas, for example, it allows us to quantify uncertainty about sample means, or to compare multiple means, without assuming a distribution.
But in phylogenetic contexts, while we may not adore our branching models, we're far more interested in their parameters (we care a lot about things that increase the branching rate) than their sample statistics.
And we can't even address all sample statistics with this approach: we could talk about the mean branching time (which I cannot, for the life of me, fathom a utility for) but not the number of branching events (which, under the right circumstances, we might just care about[^6]).

And for the class of models to which bootstrapping divergence times is applicable, I don't think we have any trouble getting at uncertainty.
Lots of our models have tractable likelihoods, so we can always use Bayesian inference to get at (multivariate) parameter uncertainty.
And I'd argue that most of our models are generative, so even if inference isn't straightforward, we can simulate from them and use a parametric bootstrap.

## TL;DR

Nonparametric bootstrapping of branching times is doable, under strong assumptions, but not particularly useful, unless you happen to have a purely time-dependent model which is not generative and which lacks a tractable likelihood.

## Code appendix

Herein lies the code required to produce the figure above.
Caveat emptor.

```
library(TESS)

set.seed(42)

lambda <- 1.0
mu <- 0.6
nsim <- 250
nboot <- 100
widths <- seq(0.05, 0.95, 0.05)

fitCR <- function(bt) {
  fn <- function(par) {
    -tess.likelihood(bt, exp(par[1]), exp(par[2]))
  }
  r <- log((length(bt) + 1) / 2) / max(bt)
  opts <- optim(log(c(r + 0.1, 0.1)), fn)
  opts$par <- exp(opts$par)
  return(opts)
}

npBootCR <- function(bt, nboot, widths) {
  rates <- sapply(seq.int(nboot), function(idx) {
    fit <- fitCR(sample(bt, replace = TRUE))
    return(c(b = fit$par[1], d = fit$par[2]))
  })
  half_alphas <- (1 - widths) / 2
  apply(rates, 1, function(samps) {
    sapply(half_alphas, function(alpha_2) {
      quantile(samps, c(alpha_2, 1 - alpha_2))
    }, simplify = FALSE)
  })
}

parBootCR <- function(bt, nboot, widths) {
  opts <- fitCR(bt)
  point <- opts$par

  sims <- tess.sim.taxa(n = nboot, nTaxa = length(bt) + 1, lambda = point[1], mu = point[2], max = 100)
  rates <- sapply(seq.int(nboot), function(idx) {
    fit <- fitCR(ape::branching.times(sims[[idx]]))
    return(c(b = fit$par[1], d = fit$par[2]))
  })
  half_alphas <- (1 - widths) / 2
  apply(rates, 1, function(samps) {
    sapply(half_alphas, function(alpha_2) {
      quantile(samps, c(alpha_2, 1 - alpha_2))
    }, simplify = FALSE)
  })
}

extractCoverage <- function(reps, widths, true_par = c(b = lambda, d = mu)) {
  res <- lapply(c("b", "d"), function(par) {
    covered <- sapply(reps, function(sim) {
      sapply(sim[[par]], function(low_high) {
        return(as.integer(low_high[1] < true_par[par] && low_high[2] > true_par[par]))
      })
    })
    dimnames(covered) <- list(widths, NULL)
    return(covered)
  })
  names(res) <- c("b", "d")
  return(res)
}

trees <- tess.sim.age(n = nsim, age = 10, lambda = lambda, mu = mu)
times <- sapply(trees, ape::branching.times)

system.time({
  np_cis <- lapply(times, npBootCR, nboot = nboot, widths = widths)
})

system.time({
  par_cis <- lapply(times, parBootCR, nboot = nboot, widths = widths)
})

np_coverage <- extractCoverage(np_cis, widths = widths)
np_birth_coverage <- cbind(widths, rowMeans(np_coverage$b))
np_death_coverage <- cbind(widths, rowMeans(np_coverage$d))


par_coverage <- extractCoverage(par_cis, widths = widths)
par_birth_coverage <- cbind(widths, rowMeans(par_coverage$b))
par_death_coverage <- cbind(widths, rowMeans(par_coverage$d))

plot(np_birth_coverage, col = "#ffa035", pch = 1, xlab = "Nominal quantile", ylab = "Coverage")
points(np_death_coverage, col = "#00a7b0", pch = 1)
points(par_birth_coverage, col = "#ffa035", pch = 4)
points(par_death_coverage, col = "#00a7b0", pch = 4)
abline(a = 0, b = 1, col = "#66666690")
legend(
  "topleft",
  legend = c("Birth rate", "Death rate", "Nonparametric bootstrap", "Parametric bootstrap"),
  col = c("#ffa035", "#00a7b0", "black", "black"),
  pch = c(16, 16, 1, 4),
  border = NA,
  bty = "n"
)
```

[^1]: As opposed to a crazy-good idea, which everyone loves.
[^2]: There's nothing wrong in general with figuring out the answer to something just because we don't know it. But if we're trying to develop methods to lubricate the wheels of the scientific endeavor, it helps to know that there's anyone out there asking the question. Or that someone _should_ be asking the question. ["What is the dimensionality of treespace?"](https://afmagee.github.io/how-not-to-measure-treespace/) is a question that I have yet to establish has any _practical_ relevance. Even if it's interesting.
[^3]: And we should expect, in general, parameters to be correlated. We know birth-death models are a mess of theoretical and practical identifiability issues. So, we might as well try to get at some of that joint uncertainty. Why not do multivariate likelihood-based confidence intervals? The same reason we use MCMC and not quadrature for Bayesian inference, it's a nonstarter past a handful of dimensions.
[^4]: If we condition on the tip times, then it may well be as simple as sampling, for each tip time, a divergence time at least as old. That is, they may be conditional samples from the same CDF.
[^5]: The majority, I'd hazard. Possibly the vast majority.
[^6]: If we have good enough sampling, or at least a handle on how good our sampling is, then this could be a way to get at the distribution of the total number of infections in an outbreak with fewer assumptions about the branching process model itself.