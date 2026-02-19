---
title: "The distribution of the standard deviation of split frequencies"
author_profile: true
layout: single
---

Before we dive in too far, it's worth making sure we know what the standard deviation of split frequencies (SDSF) is.
A split is an edge in a phylogeny, dividing the tips into two sets.
Split probabilities then tell us about support for groupings on the tree.
(We really want clade probabilities, but in unrooted land we have to make sacrifices, and we can apply the same math to clades when we have them.)
Given multiple MCMC chains for a phylogenetic analysis, we can compare split probabilities by looking at their standard deviation across the chains.
Hence, the SDSF.

Let us track the frequency of a split across $m$ chains (each containing $n$ samples), as $f_i$.
The average across all chains is $\bar{f}$.
(These are our per-chain and pooled estimates of the split's probability.)
The SDSF is either,
$$\text{SDSF} = \sqrt{ \frac{1}{m} \sum_{i=1}^{m} (f_i - \bar{f})^2 }$$
or $\sqrt{m / (m - 1)}$ times that if you want to use the Bessel correction and the unbiased sample SD.

I've written in more depth about split frequencies [here](https://projecteuclid.org/journals/bayesian-analysis/volume-19/issue-2/How-Trustworthy-Is-Your-Tree-Bayesian-Phylogenetic-Effective-Sample-Size/10.1214/22-BA1339.full)[^1], and as with many things in phylogenetics, you can find it in the [MrBayes manual](https://github.com/NBISweden/MrBayes/blob/develop/doc/manual/Manual_MrBayes_v3.2.pdf).


## But what do we do with the SDSF?
A phylogenetic posterior distribution has many splits.[^2]
If we want to assess topological convergence, we then have many SDSF to compare.
Commonly, we average them and compare to the arbitrary thresholds of 0.01 and 0.05.
There are many examples of where that's maybe not great.[^3]
Some people suggest using the maximum, and this is indeed better.
Often times rare splits are removed because they tend to have low SDSF and thus drag the average down.

## Defining bad, better
For a pair of chains, [Fabreti and Hoehna (2021)](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.13727)[^5] proposed an approach based on the sampling distribution of the absolute difference in split frequencies.
[I have proposed](https://projecteuclid.org/journals/bayesian-analysis/volume-19/issue-2/How-Trustworthy-Is-Your-Tree-Bayesian-Phylogenetic-Effective-Sample-Size/10.1214/22-BA1339.full) looking at the [distribution of the signed difference](https://onlinelibrary.wiley.com/doi/abs/10.1002/(SICI)1097-0258(19980430)17:8%3C873::AID-SIM779%3E3.0.CO;2-I).
Either way, the distribution depends on the sample size, which turns out to be a potentially useful degree of freedom.[^6]

Fabreti and Hoehna suggested that we define "bad" in terms of absolute differences larger than the 95th percentile of the sampling distribution.
As to the sample size, they fixed it to a threshold ESS, essentially the target ESS that we want our chains to exceed.
Thus, as we run our chains longer, all the absolute differences should come under this threshold.
They specifically suggested computing the exact sampling distribution, and provided this pre-computed for several choices of sample size as it is somewhat computationally intensive for even moderate sample sizes.

I proposed defining "bad" in terms of a difference in frequencies which falls outside a confidence interval.[^7]
I took the sample size to be the effective sample size (of the tree, defining and computing which was the actual point of the paper).
If splits were independent[^8] and we could perfectly estimate the relevant ESS[^9], we'd expect this to mean that we'd always have some splits failing the diagnostic.

## The sampling distribution of the SDSF
Now, neither of those is the SDSF, so why are we talking about them?
Partly to warm up to the idea of sampling distributions involving split frequencies.
Partly to motivate why we might want something different.
Both the signed and absolute difference in split frequencies are natural summaries for comparing two chains.
Neither of them readily generalizes to having three or more chains.
The SDSF does, however, and so it may be a useful starting point to think about generalizations we can use beyond pairwise comparisons.

We could instead think about saying that $m$ chains' split frequencies are overly divergent if the SDSF is too large, measured by the sampling distribution.
So, how do we get that?

Exact computation is right out.
For two chains and Fabreti and Hoehna's focal ESS of 625, exact computation requires computing all $625^2$ pairwise probabilities.
That's nearly 400,000, which is a lot, but not as bad as $625^4$ for 4 chains, which is over 150 billion.

We're left with approximation, so let's do some math and get to something we know how to approximate.[^10]
For the moment, let us forget that we're working with MCMC samples, and just think about $m$ samples of size $n$.
Under the null (that all chains are sampling the same split frequency), the distribution of _counts_ of times we see the split in any chain is identical, and all chains are independent.
Substituting $\bar{f}$ for the true mean, the count in chain $i$ is $n f_i$ which has mean $n \bar{f}$.
What is the variance?
If the samples in each chain are independent (as we typically assume in convergence diagnostics), it is $n \bar{f} (1 - \bar{f})$.
Therefore, we can turn to our old friend the central limit theorem to assert that,
$$\frac{n f_i - n \bar{f}}{\sqrt{n \bar{f} (1 - \bar{f})}}$$
is asymptotically Normal(0,1).


Armed with this,
$$
\begin{align*}
\text{SDSF} &= \sqrt{ \frac{1}{m} \sum_{i=1}^{m} (f_i - \bar{f})^2 }\\
&= \sqrt{ \frac{1}{m} \sum_{i=1}^{m} \frac{(n f_i - n \bar{f})^2}{n^2} }\\
&= \sqrt{ \frac{\bar{f} (1 - \bar{f})}{mn} \sum_{i=1}^{m} \frac{(n f_i - n \bar{f})^2}{n \bar{f} (1 - \bar{f})} }
\end{align*}
$$

We can see that the term in the summand is the square of a variable which is asymptotically Normal.
Therefore the sum is asymptotically $\chi^2$ with $m - 1$ degrees of freedom.[^11]
We lose a degree of freedom because the average, along with any $m - 1$ of the squared deviations, determines the remaining one.

However, we're not _quite_ done yet, because we actually have the square root of a scaled $\chi^2$ variable.
The scaling is easy enough.
A $\chi^2(m)$ distribution is a Gamma(shape = $m/2$, scale = $2$) distribution, and [we can scale those by, well, scaling the scale parameter](https://en.wikipedia.org/wiki/Gamma_distribution#Scaling).
So the term inside the square root is a Gamma($m / 2$, $2 (\bar{f} (1 - \bar{f})) / mn$).
The square root of a Gamma-distributed variable turns out to be known as a [Nakagami distribution](https://en.wikipedia.org/wiki/Nakagami_distribution).

Thus, for $m$ sets of $n$ independent samples, the distribution of the SDSF is asymptotically Nakagami with shape parameter $(m - 1) / 2$, and spread parameter[^12] $(\bar{f} (1 - \bar{f})) / n$.
Or, if you want the Bessel version, shape parameter $(m - 1) / 2$ and spread parameter $(\bar{f} (1 - \bar{f}) (m - 1)) / mn$.

### Reckoning with MCMC

The actual amount that $f_i$ varies from $\bar{f}$ depends not on the number of samples we take, but the effective sample size of (that split in) chain $i$.
If we simply wish to generalize the approach of Fabreti and Hoehna to more than two chains, this isn't a deal breaker, we simply put the target ESS (e.g. 625) in for $n$.
Then, as our actual ESS gets large enough, all of the SDSF will eventually decrease below the 95th percentile of their specified Nakagami distributions.

On the other hand, if we want to use this to generalize confidence interval-based detection of abnormally deviant splits, we have to reckon with the fact that each chain has its own effective sample size.
That is, there is not really one $n$ but there are actually $m$, one $n_i$ per chain.

Mathematically, we can do this just fine.
The derivation is very similar, as the logic is otherwise identical.

$$
\begin{align*}
\text{SDSF} &= \sqrt{ \frac{1}{m} \sum_{i=1}^{m} (f_i - \bar{f})^2 }\\
&= \sqrt{ \sum_{i=1}^{m} \left( \frac{1}{m} \right) \frac{(n_i f_i - n_i \bar{f})^2}{n_i^2} }\\
&= \sqrt{ \sum_{i=1}^{m} \left( \frac{\bar{f} (1 - \bar{f})}{mn_i} \right) \frac{(n_i f_i - n_i \bar{f})^2}{n_i \bar{f} (1 - \bar{f})} }
\end{align*}
$$

This is the square root of the sum of $m$ independent scaled $\chi^2(1)$ distributions.
A $\chi^2(1)$ distribution is a Gamma($1/2$, $2$) distribution, and while each scaling is different, each term in the sum is Gamma($1/2$, $2 \bar{f} (1 - \bar{f}) / mn_i$).
Unfortunately, while Gammas with the same scale parameter can be summed, Gamma distributions with the same shape parameter but different scales cannot.

Are we out of luck?
Sort of.
Still, we've come this far, and there are at least a handful of options which present themselves.

#### Turtles all the way down

We are already approximating things, so what's one more, we might ask?
The potentially least-approximate approximation is brute force simulation.
But as we would have to, for some large-ish number of replicates, simulate $m$ Gammas for each split, this quickly becomes intractable.
So, we could do something more approximate,[^13] but it if we don't find an astonishingly good one it seems unlikely to produce a particularly performant result.[^14]

What would be more approximate?
We could use $\min(n_i)$ for all $n_i$ and proceed with the equal sample sizes case to aim for a lenient test.
We could use $\max(n_i)$ instead for a strict test.
If the ESS doesn't vary too much across chains, then we might not do terribly by using the mean instead.[^15]
Lastly, we could try to match variances.
That is, each term inside the sum has variance $2 \left(\bar{f} (1 - \bar{f}) / mn_i \right)^2$, and, since chains are independent,[^16] the variance of the sum is the sum of the variances.
So, if we assume the sum looks Gamma-ish, with shape parameter $(m - 1)/2$, the scale parameter is $\sqrt{2 \sigma^2 / (m - 1)}$.

I don't get the sense that any of these are particularly good options unless the ESS are pretty consistent across the chains.

#### Reframe the problem

We could also reframe the problem.
We've been asking about the sampling distribution of the SDSF.
But what if we asked about the sampling distribution of a related but rescaled and thus slightly different quantity?

$$
\begin{align*}
\text{rSDSF} &= \sqrt{ \frac{1}{m} \sum_{i=1}^{m} (f_i - \bar{f})^2 \left( \frac{\bar{f} (1 - \bar{f})}{n_i} \right)^{-1}}\\
&= \sqrt{ \frac{1}{m} \sum_{i=1}^{m} \frac{(n_i f_i - n_i \bar{f})^2}{n_i \bar{f} (1 - \bar{f})} }
\end{align*}
$$

If we rescale the squared deviations of split frequencies, we can remove the sample size dependency.
Thus, as in the equal sample sizes case, we get a tractable Nakagami distribution.
One might note that, in essence, we've cheated, and changed the summary statistic to counts of splits.
The important thing to note is that these aren't the actual counts of times we've seen any given split in any given chain, though.
Since the $n_i$ are the per-chain ESS, these are hypothetical counts.

One potential advantage here is that, since all splits will end up with the same (asymptotic) distribution of the rSDSF, we can look at the overall performance of the chain in a single summary statistic.

## Fin

How well any of this works, if indeed it works at all, I leave for future work.
I should note, following along from the final cheat to working with split counts, instead of frequencies, that there's a somewhat conceptually simpler option presenting itself here.
If we correct for the fact that each chain has a different effective sample size, we can simply use a chi-squared test on the counts of samples with (and without) the split in each chain.
We could also use a chi-squared test in the Fabretti Hoehna style, fixing the target ESS and letting the observed test statistics decrease below it as we add more samples.


[^1]: It's my party, and I'll self-cite if I want to.
[^2]: Sometimes many, many, many. Sometimes an absurd number. Sometimes more than that.
[^3]: In [this paper](https://academic.oup.com/sysbio/article/69/2/209/5555782), we use a stricter summary and even still values of 0.03 aren't great. Figure S11 of [this paper](https://projecteuclid.org/journals/bayesian-analysis/volume-19/issue-2/How-Trustworthy-Is-Your-Tree-Bayesian-Phylogenetic-Effective-Sample-Size/10.1214/22-BA1339.full) is a real doozy.[^4]
[^4]: Yes there are good examples in other peoples' work. But I know (some of) what I've written so these were handy. Besides, the crusade against self-citation is overly broad. Wanton self-citation is one thing, as is citing yourself to the exclusion of other relevant work. But it is not, by itself, a bad thing.
[^5]: Well I guess I can cite other people.
[^6]: But not _that_ sort.
[^7]: Yes, I used the 95% CI in practice.
[^8]: They're not. Minimally because many splits are incompatible with each other and so the presence of one means the absence of the others
[^9]: We appear to be able to do far better than we have any right to expect, but ESS is still an estimator.
[^10]: 10 internet points if you can guess where this is headed already.
[^11]: Good job to everyone who saw this coming.
[^12]: Yes it's an awkward name for a parameter.
[^13]: It took me a very long time to figure out why the favored language when describing MCMC was "we approximate the posterior via..." And it's the same reason I have to say that this is an approximation: even when MCMC works, and actually samples from the posterior, that's all we have, _samples_. We don't have the distribution itself. So, while we could get a pretty good idea of the posterior mean or 95th quantile, it's still an approximation to the true mean or quantile. And to get a good approximation to quantiles out in the tail, we need a lot of samples. Far more than common the sort of ESS you commonly see in phylogenetics.
[^14]: The approximation for all sample sizes equal is not bad. But in limited testing it appears to be outperformed by a more standard chi-squared approximation using a contingency table of the presence/absence of the split across all chains. So a further approximation is unlikely to be outstandingly performant.
[^15]: It seems plausible, if not indeed likely, that all of these approximations would only work reasonably in the case where the ESS are all relatively similar, anyways.
[^16]: The more I work with MCMC convergence math, the more I get why this gets put in like a mantra in theoretical papers about MCMC. In practice, we violate this all the time because initializing good trees is hard. We need some way to have our cake and eat it too, but I don't think it's been invented yet.