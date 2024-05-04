---
title: "An interpretable single-number summary of split-based convergence"
author_profile: true
layout: single
---

Inter-run convergence of phylogenetic MCMC has long been diagnosed by comparing the probabilities of splits between chains.[^1]
I think there's a lot to like about this approach, not the least of which is that this works well regardless of how diffuse the posterior distribution is on trees.[^2]
Now, in any given run we'll sample many splits, so what we actually do to compare them across chains is a fair question.
While numeric summaries are attractive, and important for any hope of automated checking (not to mention clearly the point of this post), I've often found that there's no real substitute for plotting these probabilities in two runs against each other.[^3]

The primary summaries used for comparing chains based on split probabilities are the average and maximum of the standard deviations of split probabilities.
That is, for each split we take the standard deviation in its probability across chains.
Then we average that, or pick the biggest one.[^4]
Standard deviations are perfectly sensible summaries of variability, but how bad is _bad_?
Is 0.05 really acceptable?
Is 0.01?

[Fabreti and Hoehna (2021)](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.13727) instead propose to look at the somewhat more straightforward absolute difference in probabilities.
This is, I think, much more interpretable.
An absolute difference of 0.05 is pretty bad, actually, and I'm not thrilled with a difference of 0.01 either, but I'd be pretty okay with 0.001.
They also provide a means to interpret these values in terms of sampling distributions, which is handy.
But the approach doesn't scale beyond a pair of chains.[^5]

When you get right down to it, what we're really talking about are a bunch of observations of a binary variable.
We're usually busy fixating on the proportions, but with binary variables it's the actual counts are where we get statistical context.
It's perfectly expected for split probabilities to be off by 0.1 if we have very short chains[^6].
It's rather more worrisome if we've been running the chains for half a billion generations.

The nice thing about counts is that we've got a bunch of well-studied tools for working with them.
For example, we have the [G-test](https://en.wikipedia.org/wiki/G-test), which has the general form,

$G = 2 \sum_i O_i \ln \left( \frac{O_i}{E_i} \right)$

And the nice thing about $G$ is that it's asymptotically $\chi^2$-distributed.
Well, one nice thing, anyways.
Another nice thing about this statistic, as opposed to the perhaps more commonly encountered $\chi^2$ statistic for count data, is that it doesn't break down when everything is in one category.
We take $0 \ln(0)$ to be 0.


For our case,[^7] we can write this as,
$$G = 2 \sum_i \left[ n f_i \ln \left( \frac{n f_i}{n \bar{f}} \right) + n (1 - f_i) \ln \left( \frac{n (1 - f_i)}{n (1 - \bar{f})} \right) \right]$$
While this form is admittedly a bit uglier, it shows us what we're actually doing: looking at the presence and absence of splits across chains.
We sum over chains, each of which has split frequency $f_i$ and thus $n f_i$ observations of the split, and $n (1 - f_i)$ observations where it was absent.
The average over all chains is $\bar{f}$, which gives us our expected number of times we should see it (and not see it).
The statistic $G$ has a $\chi^2(m - 1)$ distribution, and since it works even when we don't have any observations, it doesn't break down when a split is fixed across all chains.[^8]

Armed with this, we _could_ ask if split probabilities are worryingly different across chains the normal way.
That is, for a sample size, we can compute $\text{Pr}(G \geq G_{\text{obs}})$ and say the split probabilities are worryingly different if the tail probability is below some threshold $\alpha$ we choose, like 0.05.
Or we could get creative.
I suggest that we ask the question "what is the largest possible sample size for which this observed difference is not worrisome?"
That is, if we assume the null hypothesis is true (the split frequencies are the same), we ask how big of a sample we could have taken before the observed differences in frequencies become implausible.

Let $G_{\text{crit}}$ be the critical value of $G$ under the $\chi^2(m - 1)$ distribution for our given $\alpha$ (that is, $\text{Pr}(G \geq G_{\text{crit}}) = \alpha$).
Now, notice that the actual sample size $n$ only enters into the statistic once,
$$G = n \times 2 \sum_i \left[ f_i \ln \left( \frac{f_i}{\bar{f}} \right) + (1 - f_i) \ln \left( \frac{1 - f_i}{1 - \bar{f}} \right) \right] = n \tilde{G}$$
where we define $\tilde{G}$ so we can stop lugging those ugly terms around.
We are interested in a hypothetical sample size $n_{\text{sup}}$ such that $n_{\text{sup}} \tilde{G}$ is not concerningly large.
Just to belabor the point, because we're doing something a bit counterintuitive, we take the per-chain _frequencies_ to be fixed, and we try to imagine how many counts those would have to represent before we started saying "yes, that's too different, the frequencies are not the same among the chains."

We hit "concerningly different" if $G_{\text{obs}} = n \tilde{G}_{\text{obs}} > G_{\text{crit}}$.[^9]
So, if 
$$n_{\text{sup}} \leq \frac{G_{\text{crit}}}{\tilde{G}_{\text{obs}}}$$
then
$$n_{\text{sup}} \tilde{G}_{\text{obs}} \leq G_{\text{crit}}$$

We might term this something like the maximum permissible sample size under the null (where the null is "split probabilities are the same across all chains").
Each split gives us one of these values, and we can then take the minimum as our convergence summary.
I think the scale here is relatively intuitive.
If you've got a difference in split probabilities which could only possibly arise from truly equal probabilities if the sample size was 3, that's very, very bad.
If it's 10, that's still very bad.
50 isn't great either.
On the other hand, if all your $n_{\text{sup}}$ are in the upper hundreds (or higher), you're certainly in the clear.

Basically, since we're already somewhat used to thinking about effective sample sizes, I think the scale of this approach is pretty interpretable.
And it can be contextualized in the light of your actual effective sample size too.
If the minimum $n_{\text{sup}}$ is 100 and the ESS is 200, while neither are great, you're probably talking about a problem that will go away with a longer run.
If the minimum $n_{\text{sup}}$ is 20 and the ESS is 2000, you've probably got chains in different modes.[^10]

[^1]: Sorry, I'm assuming a fair bit of background knowledge of phylogenetics here. For an introduction from a statistical perspective, I wrote more in [this paper](https://projecteuclid.org/journals/bayesian-analysis/volume-19/issue-2/How-Trustworthy-Is-Your-Tree-Bayesian-Phylogenetic-Effective-Sample-Size/10.1214/22-BA1339.full)
[^2]: I see the fundamental folk theorem of phylogenetics as something like, "splits are easy, trees are hard." For a diffuse-enough posterior distribution on a large enough tree, two runs sampling the same posterior distribution on treespace might never sample the same exact phylogeny. But they damn well should be sampling the same distribution of split frequencies.
[^3]: I'm not sure if a picture is really worth a thousand words, but a plot of many splits is certainly worth more than one summary statistic.
[^4]: The maximum is definitely more in keeping with how we usually assess inter-run convergence in Bayesian inference. You don't average all the variables' PSRFs, you look at the worst.
[^5]: Admittedly, I have trouble contemplating more than two chains at a time, but it's still nice to have statistics that work in higher dimensions.
[^6]: Or chains with really bad ESS. Either way, we should run them longer.
[^7]: Where it's the same as doing a likelihood ratio test.
[^8]: Yeah, sure, maybe we shouldn't be computing convergence diagnostics on those. But I would still prefer statistics that say this is fine, instead of undefined.
[^9]: Isn't this (non-ironically) fun? I basically never think in terms of critical values. It is 2024, after all, and we don't need p-value tables anymore.
[^10]: If they've converged at all, that is.