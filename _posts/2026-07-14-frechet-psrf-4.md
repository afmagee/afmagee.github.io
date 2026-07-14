---
title: "The Fréchet PSRF, part 4"
author_profile: true
layout: single
---

In this final part of what I initially thought would be a single post, I'm going to reiterate the Fréchet generalization of the PSRF worked on in the last parts and then compare it to Whidden and Matsen (2015)'s Gelman-Rubin-like diagnostic.
The math of the comparison is going to change a bit, but the big picture won't.

## Recap

The Fréchet PSRF is

$$
\hat{R}_\mathrm{F} = \sqrt{\frac{\hat{\sigma}_\mathrm{F}^2}{s_\mathrm{F}^2}}
$$

The pooled Fréchet sample variance is

$$
s_\mathrm{F}^2 = \frac{1}{m} \sum_k s_{\mathrm{F}, k}^2
$$

and the per-chain sample variance is

$$
s_{\mathrm{F}, k}^2 = \frac{1}{n (n - 1)} \sum_{j > i} d(x_{ki}, x_{kj})^2
$$

The biased-from-above $\hat{\sigma}\_\mathrm{F}^2$ is

$$
\hat{\sigma^2} = \left( 1 - \frac{1}{n} \right) s_\mathrm{F}^2 + \frac{1}{n}B_\mathrm{F}
$$

and

$$
B_{\mathrm{F}} = \frac{n}{m (m - 1)} \sum_{\ell > k} d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2
$$

where

$$
d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2 = 
\frac{1}{n^2} \sum_i \sum_j d(x_{k i}, x_{\ell j})^2
- \frac{1}{n^2} \sum_{j > i} d(x_{k i}, x_{k j})^2
- \frac{1}{n^2} \sum_{j > i} d(x_{\ell i}, x_{\ell j})^2
$$

This is almost entirely as laid out in [Part 1](https://afmagee.github.io/frechet-psrf/) (and references therein) except this last equation, which I got wrong there and spent a lot of time fixing.

## Compared to the Gelman-Rubin-like diagnostic

Let's start again with the easy term,

$$
s_{\text{WM}, k}^2 = \frac{1}{n (n - 1)} \sum_{i = 1}^n \sum_{j = 1}^n d(x_{k i}, x_{k j})^2 = \frac{2}{n (n - 1)} \sum_{j > i} d(x_{k i}, x_{k j})^2 = 2 s_{\text{F}, k}^2
$$

That is, summing over the entire (squared) distance matrix yields twice the variance.

The harder term is

$$
B_{\text{WM}} = \frac{1}{(m - 1)m n^2} \sum_k \sum_\ell \sum_i \sum_j d(x_{ki}, x_{\ell j})^2
$$

The sum here is over the (squared) distance matrix of the concatenated chains, so there's a block structure.
The diagonal blocks are clearly variance-only terms, and the off-diagonal blocks are between chain comparison terms (which end up including variance terms too).
Last time I tackled this by splitting it up, but having wrangled with these terms more, I'm not sure we want to.
But, note that, per [Detour 1](#detour-1), our work from [Part 3](https://afmagee.github.io/frechet-psrf-3/) holds for both these blocks.

So, rewriting this we get,

$$
B_{\text{WM}} = \frac{1}{(m - 1)m n^2} \sum_k \sum_\ell \left[ n(n - 1) s_{\mathrm{F}, k}^2 + n(n - 1) s_{\mathrm{F}, \ell}^2
+ n^2 \times d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2
 \right]
$$

Now let's tackle these terms separately.
We get the first two via [Detour 2](#detour-2)

$$
B_{\text{WM}} = \frac{1}{(m - 1)m n^2} \left( 2m^2 n(n - 1) s_{\mathrm{F}}^2 + \sum_k \sum_\ell \left[ n^2 \times d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2 \right] \right)
$$

And by [Detour 3](#detour-3)

$$
B_{\text{WM}} = \frac{1}{(m - 1)m n^2} \left( 2m^2 n(n - 1) s_{\mathrm{F}}^2 + 2 n m (m - 1) B_{\mathrm{F}} \right)
$$

Cleaning up, we get,

$$
B_{\text{WM}} = \frac{2m}{(m - 1) n (n - 1)} s_{\mathrm{F}}^2 + \frac{2}{n} B_{\mathrm{F}}
$$

The factor of 2 in $s\_{\text{WM}}^2$ and $\hat{\sigma}\_{\text{WM}}^2$ cancel, but I don't think we need to finish the math because the overall story is the same as we concluded in Part 1.
As the sample size $n$ goes to infinity, $B\_{\text{WM}}$ goes to 0, so the between-chain component here is asymptotically negligible.

## Detours

Treat this like an appendix and ignore it if you like.

### Detour 1

Synthesizing some equations from Part 3, we have, for chains $k$ and $\ell$,

$$
\frac{1}{n^2} \sum_i \sum_j d(x_{k i}, x_{\ell j})^2 =
\frac{n - 1}{n} s_{\mathrm{F}, k}^2 + \frac{n - 1}{n} s_{\mathrm{F}, \ell}^2
+ d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2
$$

Multiplying through by $n^2$, we get

$$
\sum_i \sum_j d(x_{k i}, x_{\ell j})^2 =
n(n - 1) s_{\mathrm{F}, k}^2 + n(n - 1) s_{\mathrm{F}, \ell}^2
+ n^2 \times d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2
$$

In the case where $k = \ell$, the distance between means is 0, so,

$$
\sum_i \sum_j d(x_{k i}, x_{k j})^2 =
n(n - 1) s_{\mathrm{F}, k}^2 + n(n - 1) s_{\mathrm{F}, k}^2 =
2 n (n - 1) s_{\mathrm{F}, k}^2
$$

We can also get to this identity by starting from our variance identity,

$$
s_{\mathrm{F}, k}^2 = \frac{1}{n (n - 1)} \sum_{j > i} d(x_{ki}, x_{kj})^2
$$

$$
2 n (n - 1) s_{\mathrm{F}, k}^2 = \sum_i \sum_j d(x_{ki}, x_{kj})^2
$$

### Detour 2

$$
\sum_k \sum_\ell n(n - 1) s_{\mathrm{F}, k}^2 = \sum_k m n(n - 1) s_{\mathrm{F}, k}^2
$$

$$
\sum_k \sum_\ell n(n - 1) s_{\mathrm{F}, k}^2 = m^2 n(n - 1) s_{\mathrm{F}}^2
$$

In parallel,

$$
\sum_k \sum_\ell n(n - 1) s_{\mathrm{F}, \ell}^2 = m^2 n(n - 1) s_{\mathrm{F}}^2
$$

So in total,

$$
\sum_k \sum_\ell \left[ n(n - 1) s_{\mathrm{F}, k}^2 + n(n - 1) s_{\mathrm{F}, \ell}^2 \right] = 2m^2 n(n - 1) s_{\mathrm{F}}^2
$$

### Detour 3

We have that,

$$
B_{\mathrm{F}} = \frac{n}{m (m - 1)} \sum_{\ell > k} d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2
$$

But we want to work with all $k$ and $\ell$.
We apply symmetry and the fact that for $k = \ell$ the distance between averages is 0, to note that this simply doubles things.

$$
2 B_{\mathrm{F}} = \frac{n}{m (m - 1)} \sum_k \sum_\ell d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2
$$

so

$$
\sum_k \sum_\ell d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2 = \frac{2 m (m - 1) B_{\mathrm{F}}}{n}
$$

and

$$
\sum_k \sum_\ell \left[ n^2 \times d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2 \right] = 2 n m (m - 1) B_{\mathrm{F}}
$$

## Code check

```R
set.seed(42)

psrf <- function(x) {
  m <- length(x)
  n <- length(x[[1]])

  stopifnot(class(x) == "list", all(lengths(x) == n))

  chain_means <- sapply(x, mean)
  grand_mean <- mean(chain_means)

  b_n <- sum((chain_means - grand_mean)^2) / (m - 1)

  s_sq <- mean(sapply(x, var))
  hat_sigma_sq <- (1 - 1 / n) * s_sq + b_n

  r_hat <- sqrt((hat_sigma_sq) / s_sq)

  r_hat
}

psrf_frechet <- function(x, dist_fun) {
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
        (n^2) *
        sum(d_sq[k_idx, l_idx]) -
        (n - 1) / n * vars[k] -
        (n - 1) / n * vars[l]
    }
  }

  b_n <- sum(d_sq_bars) / (m * (m - 1))

  s_sq <- mean(vars)
  hat_sigma_sq <- (1 - 1 / n) * s_sq + b_n

  r_hat <- sqrt(hat_sigma_sq / s_sq)

  r_hat
}

wm_stat <- function(x, dist_fun) {
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
    sum(d_sq[idx, idx]) / (n * (n - 1))
  })

  b_n <- 1 / ((m - 1) * m * n^3) * sum(d_sq)

  s_sq <- mean(vars)
  hat_sigma_sq <- (1 - 1 / n) * s_sq + b_n

  r_hat <- sqrt((hat_sigma_sq) / s_sq)

  r_hat
}

sim_norms <- function(m, n, means = 1:m, vars = rep(1, m)) {
  lapply(1:m, function(k) {
    rnorm(n, means[k], vars[k])
  })
}


nrep <- 100
nsamp <- 100

four_chain_psrf <- t(sapply(1:nrep, function(idx) {
  x <- sim_norms(4, nsamp)
  c(
    euclidean = psrf(x),
    frechet = psrf_frechet(x, dist),
    wm_2015 = wm_stat(x, dist)
  )
}))

r <- range(four_chain_psrf)

plot(
  four_chain_psrf[, 1:2],
  xlim = r,
  ylim = r,
  xlab = "Standard PSRF",
  ylab = "Frechet comparisons"
)
abline(a = 0, b = 1, col = "red")
legend(
  "topleft",
  legend = c("Fréchet", "WM2015"),
  fill = c("black", "blue"),
  border = NA,
  bty = "n"
)
points(four_chain_psrf[, c(1, 3)], col = "blue")
```