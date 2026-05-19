---
title: "The Fréchet PSRF, part 2"
author_profile: true
layout: single
---

[Previously](https://afmagee.github.io/frechet-psrf/), I attempted to derive a Fréchet generalization of the PSRF.
And wouldn't you know it, I was wrong.
Here, I attempt to correct my math.

## What did I test and how wrong was I?

It's always best to start relatively simple, so I started out by testing $B$.
Yes, $s^2$ would have been simpler, but that's already been through peer review, so we'll leave it alone.[^1]
But this is the heart of what we need to be right, and it reduces oh so slightly the number of places an error could creep in compared to jumping straight to $\hat{R}$ or $\hat{\sigma}^2$.

How was I testing $B$? With the code shown [below](#code-appendix), just implementing the math in my previous post.
So, each "chain" is a bunch of IID normals, and chain $k$ has mean $k$.
The comparison, as mentioned last time, is to the regular PSRF for real-valued random variables.
If the generalization is right, in this case the math should collapse and the answers should be identical.

The good news: my $B\_{\mathrm{F}}$ was _close_ to the standard $B$.

The bad news: my $B\_{\mathrm{F}}$ was _stochastically_ wrong, and wrongness varied in both $m$ and $n$.

## Fixing it: plan of attack

Now, assuming that I implemented the math right in code, that is, that this is actually a math error, where to look?
The likely culprit would seem to be that I've got a biased estimator of $\mathbb{E}[d(X_{k \cdot}, X_{\ell \cdot})^2]$.
This doesn't exactly explain the $m$-dependency, which could be a _separate_ error.
But if it's, say, off by a Bessel correction that would explain why it seems to converge as $n \to \infty$.
It's also the only bit of math in $B$ defining a random variable that hasn't previously been vetted.

## Fixing it: no plan survives contact with the enemy

Previously, I said that if we have $n$ samples of a real-valued variable $\xi$ and $n$ independent samples of a real-valued variable $\upsilon$, then

$$
\hat{\mathbb{E}}[d(\xi, \upsilon)^2] = \frac{1}{n^2} \sum_i \sum_j d(\xi_i, \upsilon_j)^2
$$

Going back to Euclidean land, that's saying

$$
\hat{\mathbb{E}}[(\xi - \upsilon)^2] = \frac{1}{n^2} \sum_i \sum_j (\xi_i - \upsilon_j)^2
$$

At this point, I will admit, I took a leap of faith, and assumed that the right answer looked Bessel-corrected.
That is, I assumed the right estimator is

$$
\hat{\mathbb{E}}[(\xi - \upsilon)^2] = \frac{1}{n (n - 1)} \sum_i \sum_j (\xi_i - \upsilon_j)^2
$$

The good news is that this did banish the stochastic part of my being wrong, but it didn't make it go away.
The bad news is now I need to try to figure out why this is The Right Thing To Do.

Going back to the math, which is the same equation we are borne back ceaselessly unto, we have, without hats

$$
\mathbb{E}[(\xi - \upsilon)^2] = \mathrm{Var}(\xi) + \mathrm{Var}(\upsilon) - 2\mathrm{Cov}(\xi,\upsilon) + \left( \mathbb{E}[\xi] - \mathbb{E}[\upsilon] \right)^2
$$

Staring at this isn't proving all that illuminating, so I probably should stare at the Bessel correction's proof instead.
But I'm going to save that for another time and see if I can't flounder my way to figuring out the right answer first, and then prove it later.

What I can say now is that, with this correction, but no other math changed, $B\_{\mathrm{F}} / B$:
- Looked like it stopped being stochastic
- Looked like it became constant in $m$
- Produced some suspiciously informative numbers as a function of $n$

In particular, it looks like with the corrected $\hat{\mathbb{E}}[(\xi - \upsilon)^2]$, $B\_{\mathrm{F}} / B = n / (n - 1)$.

## The right $B\_{\mathrm{F}}$, I hope

My numerical experiments suggest that, to within decent numerical tolerances, The Right Generalization of $B$ is

$$
B_{\mathrm{F}} = \frac{n - 1}{m (m - 1)} \sum_{\ell > k} d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2
$$

where

$$
d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2 = \left( \mathbb{E}[X_{k \cdot}] - \mathbb{E}[X_{\ell \cdot}] \right)^2 = \\
\frac{1}{n (n - 1)} \sum_{i,j} d(X_{k i}, X_{\ell j})^2\\
- \frac{1}{n (n - 1)} \sum_{j > i} d(x_{ki}, x_{kj})^2\\
- \frac{1}{n (n - 1)} \sum_{j > i} d(x_{\ell i}, x_{\ell j})^2
$$

## Next steps

Armed with something that at least checks out computationally, we need to understand why it's right mathematically.
I can allow myself to hope that the average squared distance bit is just a Bessel correction.
But then why did the leading factor of $n$ become a factor of $n - 1$?
Well, time (or someone smarter) will tell eventually.

## Code appendix

```R
set.seed(42)

nrep <- 100
nsamp <- 100

b_classic <- function(x) {
  m <- length(x)
  n <- length(x[[1]])

  stopifnot(class(x) == "list", all(lengths(x) == n))

  chain_means <- sapply(x, mean)
  grand_mean <- mean(chain_means)

  n / (m - 1) * sum((chain_means - grand_mean)^2)
}

b_frechet <- function(x, dist_fun) {
  m <- length(x)
  n <- length(x[[1]])

  stopifnot(
    class(x) == "list",
    all(lengths(x) == n),
    class(dist_fun) == "function"
  )

  d_sq <- as.matrix(dist_fun(do.call(c, x)))^2

  vars <- sapply(1:m, function(k) {
    idx <- ((k - 1) * n + 1):(k * n)
    sum(d_sq[idx, idx]) / (2 * n * (n - 1))
  })

  d_sq_bars <- matrix(0, nrow = m, ncol = m)

  for (k in 1:(m - 1)) {
    k_idx <- ((k - 1) * n + 1):(k * n)
    for (l in (k + 1):m) {
      l_idx <- ((l - 1) * n + 1):(l * n)
      d_sq_bars[k, l] <- 1 /
        (n * (n - 1)) *
        sum(d_sq[k_idx, l_idx]) -
        vars[k] -
        vars[l]
    }
  }

  (n - 1) / (m * (m - 1)) * sum(d_sq_bars)
}

sim_norms <- function(m, n, means = 1:m, vars = rep(1, m)) {
  lapply(1:m, function(k) {
    rnorm(n, means[k], vars[k])
  })
}

two_chain_b <- sapply(1:nrep, function(idx) {
  x <- sim_norms(2, nsamp)
  c(euclidean = b_classic(x), frechet = b_frechet(x, dist))
})
two_chain_ratios <- two_chain_b["frechet", ] / two_chain_b["euclidean", ]
two_chain_diffs <- two_chain_b["frechet", ] - two_chain_b["euclidean", ]

four_chain_b <- sapply(1:nrep, function(idx) {
  x <- sim_norms(4, nsamp)
  c(euclidean = b_classic(x), frechet = b_frechet(x, dist))
})
four_chain_ratios <- four_chain_b["frechet", ] / four_chain_b["euclidean", ]
four_chain_diffs <- four_chain_b["frechet", ] - four_chain_b["euclidean", ]

eight_chain_b <- sapply(1:nrep, function(idx) {
  x <- sim_norms(8, nsamp)
  c(euclidean = b_classic(x), frechet = b_frechet(x, dist))
})
eight_chain_ratios <- eight_chain_b["frechet", ] / eight_chain_b["euclidean", ]
eight_chain_diffs <- eight_chain_b["frechet", ] - eight_chain_b["euclidean", ]

summary(two_chain_ratios)
summary(four_chain_ratios)
summary(eight_chain_ratios)

summary(two_chain_diffs)
summary(four_chain_diffs)
summary(eight_chain_diffs)
```

[^1]: Not that peer review guarantees correctness.
