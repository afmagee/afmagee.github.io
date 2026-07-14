---
title: "The Fréchet PSRF, part 3 (of 4)"
author_profile: true
layout: single
---

[Previously](https://afmagee.github.io/frechet-psrf/), I attempted to derive a Fréchet generalization of the PSRF.
[Then](https://afmagee.github.io/frechet-psrf-2/) I stumbled around in the dark and stumbled on a form that checks out.
Here I'm going to actually do the math I should have done in the first place.


## What I am and am not going to do here

All I'm going to do here is fix the $B$ term in the Fréchet PSRF.
I'll then verify it, and look at where I went wrong previously.
I'll stitch this all back together separately to present the entire Fréchet PSRF, and only then compare this all to the Gelman-Rubin-like diagnostic of Whidden and Matsen (2015).

## The key quantity

We have RVs $\Xi$ and $\Upsilon$, and for each we have $n$ samples $\xi\_1, \dots, \xi\_n$ and $\upsilon\_1, \dots, \upsilon\_n$.
These have sample means $\bar{\xi}$ and $\bar{\upsilon}$.
They have (Bessel-corrected) sample variances $s\_\xi^2$ and $s\_\upsilon^2$.
They have (Bessel-corrected) sample covariance $S\_{\xi \upsilon}$.

We want to know what this term is equal to.

$$
\frac{1}{n^2} \sum_i \sum_j (\xi_i - \upsilon_j)^2
$$

So, now we do the math.
More or less one term at a time.
And yes, this time I'm cheating a little because I've more or less worked it out already and I want the math to look slightly less horrific.[^1]
The shape we're aiming for here is a decomposition into two variance terms and a term about the difference in means (we'd accept a covariance term in the mix, but we'll be assuming independence so it will go away).
Most of the math amounts to either finding the terms that you can push outside a sum, or converting between means (times constants) and sums.

I'm going to try to preserve the larger features of what we're doing by relegating some of the "everything ends up canceling" type detours to subsections.
I'm using gratuitous bracketing to try to keep sum terms clear and to make sure it's clear the difference between `\bar{\xi}^2` and `\bar{\xi^2}`.
(Though we use only the former.)

$$
\frac{1}{n^2} \sum_i \sum_j \left[ \xi_i^2 - 2\xi_i \upsilon_j + \upsilon_j^2 \right]
$$

$$
\frac{1}{n} \sum_i \left[ \xi_i^2 \right]
+ \frac{1}{n^2} \sum_i \sum_j \left[ 2\xi_i \upsilon_j \right]
+ \frac{1}{n} \sum_j \left[ \upsilon_j^2 \right]
$$

By [Detour 1](#detour-1), the left- and right-most terms here yield,

$$
\left[ \frac{n - 1}{n} s_\xi^2  + (\bar{\xi})^2 \right]
- \frac{1}{n^2} \sum_i \sum_j \left[ 2\xi_i \upsilon_j \right]
+ \left[ \frac{n - 1}{n} s_\upsilon^2  + (\bar{\upsilon})^2 \right]
$$

By [Detour 2](#detour-2), under the assumption that $\Xi$ and $\Upsilon$ are independent,

$$
\left[ \frac{n - 1}{n} s_\xi^2  + (\bar{\xi})^2 \right]
- \left[ 2 \bar{\upsilon} \bar{\xi} \right]
+ \left[ \frac{n - 1}{n} s_\upsilon^2  + (\bar{\upsilon})^2 \right]
$$

Now we clean up and recognize that we've got a quadratic sitting around,

$$
\frac{n - 1}{n} s_\xi^2 + \frac{n - 1}{n} s_\upsilon^2
- 2 \bar{\upsilon} \bar{\xi}
+ (\bar{\xi})^2 + (\bar{\upsilon})^2
$$

$$
\frac{n - 1}{n} s_\xi^2 + \frac{n - 1}{n} s_\upsilon^2
+ (\bar{\xi} - \bar{\upsilon})^2
$$

## Getting it right, finally

We have now established that,

$$
\frac{1}{n^2} \sum_i \sum_j (\xi_i - \upsilon_j)^2 =
\frac{n - 1}{n} s_\xi^2 + \frac{n - 1}{n} s_\upsilon^2
+ (\bar{\xi} - \bar{\upsilon})^2
$$

For the PSRF, we want the squared difference in means, so we rearrange it,
$$
(\bar{\xi} - \bar{\upsilon})^2 = 
\frac{1}{n^2} \sum_i \sum_j (\xi_i - \upsilon_j)^2
- \frac{n - 1}{n} s_\xi^2 - \frac{n - 1}{n} s_\upsilon^2
$$

The "usual" $B\_{\mathrm{F}}$ generalization is then,

$$
d(\bar{\xi}, \bar{\upsilon})^2 = 
\frac{1}{n^2} \sum_i \sum_j d(\xi_i, \upsilon_j)^2 -
\frac{n - 1}{n} s_\xi^2 - \frac{n - 1}{n} s_\upsilon^2
$$

At this point I hesitate to invoke a new name for the right-most term here, but for independent sets of samples I do think we can call it the (sample) average squared distance between the samples.

Recall that the definition of the Fréchet sample variance is

$$
s_{\text{F}}^2 = \frac{1}{n (n - 1)} \sum_{j > i} d(\xi_i, \xi_j)^2
$$

This means we can write out all the terms as sums over parts of matrices as,

$$
d(\bar{\xi}, \bar{\upsilon})^2 = 
\frac{1}{n^2} \sum_i \sum_j d(\xi_i, \upsilon_j)^2
- \frac{1}{n^2} \sum_{j > i} d(\xi_i, \xi_j)^2
- \frac{1}{n^2} \sum_{j > i} d(\upsilon_i, \upsilon_j)^2
$$



## $B\_{\mathrm{F}}$

To get the $B\_{\mathrm{F}}$ term in the Fréchet PSRF, now we move from $\Xi$ and $\Upsilon$ to MCMC replicates ${X}_{k \cdot}$ and ${X}_{\ell \cdot}$.

$$
d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2 = 
\frac{1}{n^2} \sum_i \sum_j d(x_{k i}, x_{\ell j})^2
- \frac{1}{n^2} \sum_{j > i} d(x_{k i}, x_{k j})^2
- \frac{1}{n^2} \sum_{j > i} d(x_{\ell i}, x_{\ell j})^2
$$

And _this_ is suitable for plugging directly into what we'd otherwise expect the Fréchet PSRF's $B\_{\mathrm{F}}$ term should look like, without any funky counter-correction terms,

$$
\frac{B_{\mathrm{F}}}{n} = \frac{1}{m (m - 1)} \sum_{\ell > k} d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2
$$

## So, what did I do wrong?

The only lesson I'm sure I can extract here is that I should have just done the math for the terms I was actually using.

[The first time out, I went](https://afmagee.github.io/frechet-psrf/), I started with an identity I had [written out previously](https://projecteuclid.org/journals/bayesian-analysis/volume-19/issue-2/How-Trustworthy-Is-Your-Tree-Bayesian-Phylogenetic-Effective-Sample-Size/10.1214/22-BA1339.full).
I then looked at the terms, figured I knew what they meant, and plugged values in.
I can't quite tell if I got mucked up in _population_ vs _sample_ thinking,[^2] or if I didn't quite get the unbiased estimator I was trying for (by dint of trying to match unbiased variance estimators in the usual PSRF equations).
Partly this is because I'm still not quite sure what the mean squared difference terms really are.

My [stumbling around in the dark](https://afmagee.github.io/frechet-psrf-2/) shows the danger of intuition.
I was in the right wheelhouse of the solution, because my original term was a bit too small.
But I started adding extra terms in and over-corrected, so I ended up having to re-correct the overall $B$ by an odd amount.

All in all, though, most of this isn't the most _importantly_ wrong.
My bad estimator in the first post was close enough to right to get something that would be a recognizable estimator of the PSRF.
But it's safer to operate with firm ground under one's feet, so it's good to fix these things.

## Mathematical detours

Feel free to treat this like an appendix and ignore it.

### Detour 1

$$
\frac{1}{n} \sum_i \left[ \xi_i^2 \right]
$$

$$
\frac{1}{n} \sum_i \left[ (\xi_i - \bar{\xi})^2  - (\bar{\xi})^2  + 2 \xi_i \bar{\xi} \right]
$$

$$
\frac{1}{n} \sum_i \left[ (\xi_i - \bar{\xi})^2 \right]  - \frac{1}{n} \sum_i \left[ (\bar{\xi})^2  \right]  + \frac{1}{n} \sum_i \left[  2 \xi_i \bar{\xi} \right]
$$

$$
\frac{1}{n} \sum_i \left[ (\xi_i - \bar{\xi})^2 \right]  - \frac{1}{n} \sum_i \left[ (\bar{\xi})^2  \right]  + \frac{2 \bar{\xi}}{n} \sum_i \left[ \xi_i \right]
$$

$$
\frac{1}{n} \sum_i \left[ (\xi_i - \bar{\xi})^2 \right]  - \frac{1}{n} \sum_i \left[ (\bar{\xi})^2  \right]  + 2 (\bar{\xi} )^2
$$

$$
\frac{1}{n} \sum_i \left[ (\xi_i - \bar{\xi})^2 \right]  - (\bar{\xi})^2  + 2 (\bar{\xi} )^2
$$

$$
\frac{1}{n} \sum_i \left[ (\xi_i - \bar{\xi})^2 \right]  + (\bar{\xi})^2
$$

$$
\frac{n - 1}{n} s_\xi^2  + (\bar{\xi})^2
$$


### Detour 2

Some day I may come back and work out what this is without starting from the assumption of independence.
But since we're trying to work out multi-chain convergence diagnostics, it's a huge time saver just to plug that in.
Since we have _independent_ sample sets, $\xi\_i$ is independent of $\upsilon\_j$, and we can push the sums and averages around.

$$
\frac{1}{n^2} \sum_i \sum_j \left[ 2\xi_i \upsilon_j \right]
$$

$$
\frac{2}{n^2} \sum_i \sum_j \xi_i \upsilon_j
$$

$$
\frac{2}{n^2} \sum_i \xi_i \sum_j \upsilon_j
$$

$$
\frac{2}{n^2} \sum_i \xi_i \left[ n \bar{\upsilon} \right]
$$

$$
\frac{2 \bar{\upsilon}}{n} \sum_i \xi_i
$$

$$
\frac{2 \bar{\upsilon}}{n} \left[ n \bar{\xi} \right]
$$

$$
2 \bar{\upsilon} \bar{\xi}
$$

## Code appendix

As I keep mucking up the math, it proves useful to spot-check that I get it right.

### The fix

This checks the key identity.
(Small sample sizes are better for seeing small off by 1 or (n - 1)/n type errors).

```R
check_it <- function(x, y) {
    n <- length(x)
    stopifnot(length(y) == n)
    ones <- rep(1, n)

    lhs <- (1 / n^2) *
        sum(sapply(x, function(xi) {
            sapply(y, function(yj) {
                (xi - yj)^2
            })
        }))

    rhs <- ((n - 1) / n * var(x)) +
        ((n - 1) / n * var(y)) +
        (mean(x) - mean(y))^2

    return(lhs - rhs)
}

nrep <- 1000
nsamp <- 10

res <- sapply(seq_len(nrep), function(i) {
    x <- rnorm(nsamp)
    y <- rexp(nsamp)
    check_it(x, y)
})

summary(res)
```

### The whole enchilada

This code checks the math for the entire B term.

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

  mle_vars <- sapply(1:m, function(k) {
    idx <- ((k - 1) * n + 1):(k * n)
    sum(d_sq[idx, idx]) / (2 * n^2)
  })

  d_sq_bars <- matrix(0, nrow = m, ncol = m)

  for (k in 1:(m - 1)) {
    k_idx <- ((k - 1) * n + 1):(k * n)
    for (l in (k + 1):m) {
      l_idx <- ((l - 1) * n + 1):(l * n)
      d_sq_bars[k, l] <- 1 /
        (n^2) *
        sum(d_sq[k_idx, l_idx]) -
        mle_vars[k] -
        mle_vars[l]
    }
  }

  n / (m * (m - 1)) * sum(d_sq_bars)
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

[^1]: Yes, this is the nicer version. Not shorter, just nicer.
[^2]: This is part $n+1$ of the series "oh, so _that's_ why they stress that so much in intro stats" in my personal learning journey. Proving once again that a great way to get a handle on what frequentist stats actually means is to do Bayesian statistics and ask, "well, shoot, what do I do with these samples?"
