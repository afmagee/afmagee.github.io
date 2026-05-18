---
title: "The Fréchet PSRF"
author_profile: true
layout: single
---

[Gelman and Rubin (1992)](https://projecteuclid.org/journals/statistical-science/volume-7/issue-4/Inference-from-Iterative-Simulation-Using-Multiple-Sequences/10.1214/ss/1177011136.full) proposed the Potential Scale Reduction Factor (PSRF), perhaps most commonly know known as "Rhat"[^1].
This is, give or take the field dropping some terms from the original for convenience, perhaps the most widely used way of comparing two or more MCMC chains for diagnosing MCMC convergence.
[Whidden and Matsen (2015)](https://academic.oup.com/sysbio/article/64/3/472/1632660) proposed what they called a "Gelman-Rubin-like" diagnostic as a generalization to phylogenies.

I was, when I started writing this, pretty sure that at some point five or six years ago I wrote out some math that shows this is a valid Fréchet generalization of the PSRF in much the same way one can generalize the [effective sample size](https://projecteuclid.org/journals/bayesian-analysis/volume-19/issue-2/How-Trustworthy-Is-Your-Tree-Bayesian-Phylogenetic-Effective-Sample-Size/10.1214/22-BA1339.full).
But the more I look at all this math, the more I think that I didn't actually manage to do so.
Or, perhaps, I got extraordinarily lucky then and goofed this time around.

## PSRF

Assume you have $m$ MCMC runs each with $n$ samples.
The form of the PSRF that you usually see written out[^3] is a ratio of standard deviations

$$
\hat{R} = \sqrt{\frac{\hat{\sigma^2}}{s^2}}
$$

where $s^2$ is the average within-chain sample variance and $\hat{\sigma^2}$ is

$$
\hat{\sigma^2} = \left( 1 - \frac{1}{n} \right) s^2 + \frac{1}{n}B
$$

which is constructed to be biased upwards.
Note that while this looks like it is constructed such that $\hat{\sigma^2} \to s^2$ as $n \to \infty$, the definition of $B$ has a cancelling $n$ in it

$$
B = \frac{n}{m - 1}\sum_{k} (\bar{X}_{k \cdot} - \bar{X})^2
$$

So, when all independent MCMC runs are sampling the same density, $B \to 0$ as $n \to \infty$,[^4] so $\hat{R} \to 1$.

### To $B$ or not to $B$

It may prove helpful later[^9] to have some idea what $B$ really is, if we're going to start trying to generalize it and otherwise muck with math in its general vicinity.

To start, let us adopt a mixture distribution view of the situation.
We have $m$ component distributions with equal weight $1/m$ and a categorical variable for which component $k$ a given one of the $mn$ total samples comes from.
Let $Z$ be the component-defining categorical variable.

Now we have to be careful of randomness.
The sample mean of chain $k$, $\bar{X}\_{k \cdot}$, is a random variable, as is the grand mean $\bar{X}$.
They are also _estimates_

$$
\bar{X}_{k \cdot} = \hat{\mathbb{E}}[X \mid Z = k]
$$

and

$$
\bar{X} = \hat{\mathbb{E}}[X] = \hat{\mathbb{E}}[\mathbb{E}[X \mid Z]]
$$


In this light, $B/n$, give or take a Bessel correction, can be written

$$
\frac{B}{n}
= \frac{1}{m - 1} \sum_{k} (\hat{\mathbb{E}}[X \mid Z] - \hat{\mathbb{E}}[\mathbb{E}[X \mid Z]])^2
$$

Now, for the sake of sanity, let's drop the hats.
Clearly, $B/n$ is a sample-based estimate of something, and if we look at the RHS here, with or without hats, we can understand what.

$$
\frac{1}{m - 1} \sum_{k} (\mathbb{E}[X \mid Z] - \mathbb{E}[\mathbb{E}[X \mid Z]])^2
= \mathbb{E}[(\mathbb{E}[X \mid Z] - \mathbb{E}[\mathbb{E}[X \mid Z]])^2]
= \mathrm{Var}(\mathbb{E}[X \mid Z])
$$

So, $B/n$ is a term _estimating_ the variance in the mixture component's means.
It feels worth noting, since we've gone to the trouble to write this all out, that this is one of two components in the [law of total variance](https://en.wikipedia.org/wiki/Law_of_total_variance)

$$
\mathrm{Var}(X) = \mathbb{E}[\mathrm{Var}(X \mid Z]) + \mathrm{Var}(\mathbb{E}[X \mid Z])
$$

For mixture distributions, this tells us that the total (marginal) variance is bigger than the average variance by a factor of how far apart the means of the components are.

## Whidden and Matsen (2015)

Index chains such that $x\_{ki}$ is the $i$th sample from chain $k$.
Let $
d(x,y)$ be a distance function, such as the SPR distance, as Whidden and Matsen (2015) used.
They proposed to quantify convergence with

$$
\hat{R}_{\text{WM}} = \sqrt{\frac{\hat{\sigma^2}_{\text{WM}}}{s_{\text{WM}}^2}}
$$

where $s\_{\text{WM}}^2$ is the average of the $m$ per-chain quantities

$$
s_{\text{WM}, k}^2 = \frac{1}{n (n - 1)} \sum_{i} \sum_{j} d(x_{ki}, x_{kj})^2
$$

and where $\hat{\sigma^2}\_{\text{WM}}$ is calculated analogously to $\hat{\sigma^2}$ but using

$$
B_{\text{WM}} = \frac{1}{(m - 1)m n^2} \sum_k \sum_\ell \sum_i \sum_j (x_{ki}, x_{\ell j})^2
$$

## My memory is made the fool

When I started writing this, I thought I was going to unpack Whidden and Matsen's equation rather directly.
After all, I did it before, right?
Instead, it turned out to be easier to derive the ingredients of a Fréchet-generalized PSRF from scratch, which we'll do in this section, and in the next section we'll compare them.

I like to think of Fréchet generalizations as what you get when you substitute (squared) arbitrary distance functions for (squared) Euclidean distances.
For a distribution on real numbers, the mean is the value minimizing the squared Euclidian distance to all other values, weighting by the distibution's PDF or PMF.
The sample mean is the point minimizing the squared Euclidean distance to the sampled values.
The variance is the average squared Euclidean distance from the mean to the rest of the distribution, weighted by the PDF or PMF.
Give or take a Bessel correction, the sample variance is the average squared Euclidean distance from the sample mean to the sampled values.
Remove the qualifier "Euclidean" and you're in Fréchet land.

Now let's look at the two key terms in the PSRF and see what that looks like in Fréchet land.

### The easy one: $s^2$

The Fréchet sample variance of a single MCMC run is, per Supplementary Equation 15 of [a paper of mine on the ESS of phylogenetic trees](https://projecteuclid.org/journals/bayesian-analysis/volume-19/issue-2/How-Trustworthy-Is-Your-Tree-Bayesian-Phylogenetic-Effective-Sample-Size/10.1214/22-BA1339.full)

$$
s_{\text{F}, k}^2 = \frac{1}{n (n - 1)} \sum_{j > i} d(x_{ki}, x_{kj})^2
$$

### The harder one: $B$

The biased-from-above $\sigma^2$ term, or equivalently $B$, is a bit more complex.
Several initial attempts here led me to failure, which I won't bore the universe with.[^5]
To get there, we take a somewhat more circuitous path, including some helpfully unnumbered equations[^6] from the supplement to my [aforementioned tree ESS paper](https://projecteuclid.org/journals/bayesian-analysis/volume-19/issue-2/How-Trustworthy-Is-Your-Tree-Bayesian-Phylogenetic-Effective-Sample-Size/10.1214/22-BA1339.full) (see the section entitled "The frechetCorrelationESS").

One of these equations states that, for (real-valued) random variables $\xi$ and $\upsilon$,

$$
\mathbb{E}[(\xi - \upsilon)^2] = \mathrm{Var}(\xi) + \mathrm{Var}(\upsilon) - 2\mathrm{Cov}(\xi,\upsilon) + \left( \mathbb{E}[\xi] - \mathbb{E}[\upsilon] \right)^2
$$

In the tree ESS paper, we used this equation to estimate the covariance, because we could separately estimate the variances, and because we could chant "MCMCCLT" and call the last term 0.
But for our purposes here, we have samples from independent MCMC runs, so $\xi$ and $\upsilon$ are $X_{k \cdot}$ and $X_{\ell \cdot}$, and thus their covariance is 0.[^7] 
We can still get the variances separately, so if we rearrange the equation, make the Fréchet substitution of $d(\xi,\upsilon)^2$ for $(\xi - \upsilon)^2$, and use our chain-based variables, we get

$$
\left( \mathbb{E}[X_{k \cdot}] - \mathbb{E}[X_{\ell \cdot}] \right)^2 = \mathbb{E}[(X_{k \cdot} - X_{\ell \cdot})^2] - \mathrm{Var}(X_{k \cdot}) - \mathrm{Var}(X_{\ell \cdot})
$$

The practical use of this equation is _estimating_ the LHS by using the estimates we get for the terms on the RHS.
What estimates, you ask?
The variances are [as above](#the-easy-one).
The remaining term should be estimable as the average of all $X\_{k i}$, $X\_{\ell j}$ comparisons.[^8]
Putting that together, giving it a convenient shorthand for later, and not putting hats over terms that deserve it because I can't make `\widehat` work in markdown LaTeX, we get

$$
d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2 = \left( \mathbb{E}[X_{k \cdot}] - \mathbb{E}[X_{\ell \cdot}] \right)^2 = \\
\frac{1}{n^2} \sum_{i,j} d(X_{k i}, X_{\ell j})^2\\
- \frac{1}{n (n - 1)} \sum_{j > i} d(x_{ki}, x_{kj})^2\\
- \frac{1}{n (n - 1)} \sum_{j > i} d(x_{\ell i}, x_{\ell j})^2
$$

Now, right about now it looks like we've solved the Wrong Problem.
We've got a difference between chain means pairwise, but that's not what $B/n$ is.
But if we look at the equations that gave us the Fréchet variance, we can turn this into what we want.
For $n$ samples of a variable $X$,

$$
\frac{1}{n - 1} \sum_i (X_i - \bar{X})^2 = \frac{1}{n(n - 1)} \sum_{j > i} (X_i - X_j)^2
$$

Instead, we consider the $m$ sample _means_ of the chains, and we get

$$
\frac{1}{m - 1} \sum_i (\bar{X}_k - \bar{X})^2 = \frac{1}{m(m - 1)} \sum_{\ell > k} (\bar{X}_k - \bar{X}_\ell)^2
$$

And there we have it, the LHS is $B/n$ and the RHS we just showed how to compute.

$$
\frac{B_{\mathrm{F}}}{n} = \frac{1}{m (m - 1)} \sum_{\ell > k} d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2
$$

## Wherefore art thou Fréchet

Now we can interpret Whidden and Matsen (2015) in a Fréchet context.

The $1 / (n (n - 1))$ term in both $s\_{\text{F}, k}^2$ and $s\_{\text{WM}, k}^2$ is identical, but the sum for the Fréchet variance is restricted to the upper (or lower) triangular portion of the matrix.
As this is a distance matrix, the diagonal is $\mathbf{0}$, and the two triangular portions are equal.
Thus

$$
s_{\text{WM}, k}^2 = 2 s_{\text{F}, k}^2
$$

To understand $B\_{\text{WM}}$, we need to do some rewriting of what we wrote above.

We'll start by trying to get a birds eye view of where we're going.
We'll use some dubious notation to revisit and slightly tweak an earlier definition of $d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2$, yielding

$$
d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2 = 
\mathbb{E}[d(X_{k \cdot}, X_{\ell \cdot})^2]
- \mathrm{Var}(X_{k \cdot})
- \mathrm{Var}(X_{\ell \cdot})
$$

Plugging this into our compact definition of $B\_{\mathrm{F}}$, we get

$$
B_{\mathrm{F}} = \frac{n}{m (m - 1)} \sum_{\ell > k}
\left[
\mathbb{E}[d(X_{k \cdot}, X_{\ell \cdot})^2]
- \mathrm{Var}(X_{k \cdot})
- \mathrm{Var}(X_{\ell \cdot})
\right]
$$

The variance terms each show up $(m - 1)$ times, one each for each $k \neq \ell$ pair.
Condensing those terms and then cleaning up the leading fractions accordingly, we get

$$
B_{\mathrm{F}} = 
\frac{n}{m (m - 1)} \left[ \sum_{\ell > k} \mathbb{E}[d(X_{k \cdot}, X_{\ell \cdot})^2] \right]
- \frac{n}{m} \left[ \sum_{k} \mathrm{Var}(X_{k \cdot})]
\right]
$$

At this point, we can perhaps finally see where this is going if we think in terms of the matrix of (squared) distances between all $mn$ samples in all chains.
This is a block matrix, including both within-chain ($k = \ell$) terms and between-chain ($k \neq \ell$) terms.
$B\_{\mathrm{F}}$ is the sum of a term proportional to the sum of the upper (or lower, if you'd rather) diagonal blocks and a term proportional to the sum of the diagonal blocks, but we're subtracting the diagonal terms.
To be properly clear, let's put this in more explicit sum form.
So, now we go backwards, and plug in our actual definitions from when we defined $d(\bar{X}_{k \cdot}, \bar{X}_{\ell \cdot})^2$, yielding

$$
B_{\mathrm{F}} = 
\frac{n}{m (m - 1)} \left[ \sum_k \sum_{\ell > k} \sum_i \sum_j \frac{1}{n^2} d(x_{k i}, x_{\ell j})^2 \right]\\
- \frac{n}{m} \left[ \sum_{k} \sum_i \sum_{j > i} \frac{1}{n (n - 1)}  d(x_{ki}, x_{kj})^2]
\right]
$$

Cleaning up, we get

$$
B_{\mathrm{F}} = 
\frac{1}{n m (m - 1)} \left[ \sum_k \sum_{\ell > k} \sum_i \sum_j d(x_{k i}, x_{\ell j})^2 \right]\\
- \frac{1}{(n - 1) m} \left[ \sum_{k} \sum_i \sum_{j > i} d(x_{ki}, x_{kj})^2]
\right]
$$

Now we can make that first term a sum over all off-diagonal blocks, and compensate with a factor of $1/2$ in the leading fraction because distance matrices are symmetric.

$$
B_{\mathrm{F}} = 
\frac{1}{2 n m (m - 1)} \left[ \sum_k \sum_{\ell \neq k} \sum_i \sum_j d(x_{k i}, x_{\ell j})^2 \right]\\
- \frac{1}{(n - 1) m} \left[ \sum_{k} \sum_i \sum_{j > i} d(x_{ki}, x_{kj})^2]\right]
$$

We're almost done.
$B\_{\mathrm{F}}$ is a sum over the entire matrix, diagonals and all, so let's add them back into our leading term (and balance the equation as needed).

$$
B_{\mathrm{F}} = 
\frac{1}{2 n m (m - 1)} \left[ \sum_k \sum_\ell \sum_i \sum_j d(x_{k i}, x_{\ell j})^2 \right]\\
- \frac{1}{2 n m (m - 1)} \left[ \sum_{k} \sum_i \sum_{j > i} d(x_{ki}, x_{kj})^2] \right]\\
- \frac{1}{(n - 1) m} \left[ \sum_{k} \sum_i \sum_{j > i} d(x_{ki}, x_{kj})^2] \right]
$$

And now we clean up.

$$
B_{\mathrm{F}} = 
\frac{1}{2 n m (m - 1)} \left[ \sum_k \sum_\ell \sum_i \sum_j d(x_{k i}, x_{\ell j})^2 \right]\\
- \left(\frac{1}{2 n m (m - 1)}  + \frac{1}{(n - 1) m} \right) \left[ \sum_{k} \sum_i \sum_{j > i} d(x_{ki}, x_{kj})^2] \right]
$$

At this point, we may have forgotten that

$$
B_{\text{WM}} = \frac{1}{(m - 1)m n^2} \sum_k \sum_\ell \sum_i \sum_j (x_{ki}, x_{\ell j})^2
$$

So, we have

<!--
$$
B_{\text{WM}} = \frac{1}{(m - 1)m n^2} \Theta
$$

$$
\Theta = B_{\text{WM}} \times (m - 1)m n^2
$$

$$
B_{\mathrm{F}} = 
\frac{(m - 1)m n^2}{2 n m (m - 1)} B_{\text{WM}}\\
- \left(\frac{1}{2 n m (m - 1)}  + \frac{1}{(n - 1) m} \right) \left[ \sum_{k} \sum_i \sum_{j > i} d(x_{ki}, x_{kj})^2] \right]
$$ -->

$$
B_{\mathrm{F}} = 
\frac{n}{2} B_{\text{WM}}
- \left(\frac{1}{2 n m (m - 1)}  + \frac{1}{(n - 1) m} \right) \left[ \sum_{k} \sum_i \sum_{j > i} d(x_{ki}, x_{kj})^2] \right]
$$

Alternately

$$
B_{\text{WM}} = 
\frac{2}{n} B_{\mathrm{F}} + \left(\frac{1}{n^2 m (m - 1)}  + \frac{2}{n(n - 1) m} \right) \left[ \sum_{k} \sum_i \sum_{j > i} d(x_{ki}, x_{kj})^2] \right]
$$

## TL;DR

This has gotten excessively long for a blog post, so let's wrap it up.
Where are we?

If the math I did here is right, and that is not a foregone conclusion, then Whidden and Matsen's proposed Gelman-Rubin-like diagnostic:
- Is Gelmen-Rubin-like. We can see that it's related to a Fréchet generalization of the PSRF.
- Is not a Fréchet generalization of the PSRF. Its $B$ term downweights the between-chain variance and upweights the within-chain variation. Asymptotically in chain length $n$ the within-chain bit goes to zero and $B\_{\text{WM}} \to 2/n \times B\_{\mathrm{F}}$. So we might expect that it is relatively insensitive to between-chain variation.

What still needs to be done?
Obviously I need to check my math.
But that's a lot of math to check, so a test is in order.
If I got the math right, and I implement that math right in code, then for a real-valued variable, if I use the Euclidean distance, I ought to be able to recover the non-Fréchet PSRF with what I claim is the Fréchet PSRF.

But, again, this has gone on long enough, so I'll leave myself in suspense until I get around to doing some implementation and checking.
I'll add a link to a follow-up when I invariably find some of my math errors in this computational testing.

[^1]: Because that's what the Stan output calls it. `coda` calls it the [`gelman.diag`](https://cran.r-project.org/web/packages/coda/refman/coda.html#gelman.diag). But there are lots of variables called "R," and Gelman has proposed other diagnostics[^2], so I go with "PSRF" in text.
[^2]: Not that I ever immediately remember "diag" is short for "diagnostic" and not "diagonal."
[^3]: Which drops a few terms from the original, see Remark 1 in [Vats and Knudson (2021)](https://www.jstor.org/stable/27286459).
[^4]: Wave your hands and chant something about the (MCMC)CLT.
[^5]: Will I regret that some day? Mayhaps. Recording failures can be quite handy later.
[^6]: This reminds me why someone much wiser than me told me once to number every equation in a paper. Live and learn.
[^7]: In other words, if you've got a gnarly starting tree problem, and you use the same tree to initialize all runs, you shouldn't really use this. Though I've seen enough runs go off and fail to converge from the same starting tree to be less paranoid about this than I used to be, as long as the runs are long and the starting tree is more "in the neighborhood of the peak" than "at the peak."
[^8]: I say "should" because I haven't absolutely convinced myself this is correct. It feels like it's right, but when people compute the [Energy Score](https://frazane.github.io/scoringrules/api/energy/), which involves this same average squared difference term disguised as a Norm, it's not actually what people do. Probably because there are estimates of it that don't require $\mathcal{O}(n^2)$ comparisons. But still, one wonders. And one wonders what MCMC-induced autocorrelation does to all this.
[^9]: In the end, I think I could have managed without it, but it helped get me on the right track. It's usually worth going one level deeper of what something _is_ before working with it.