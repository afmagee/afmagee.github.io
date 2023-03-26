---
title: "A two-sample test for tree distributions"
author_profile: true
layout: single
---

A relatively common question that crops up in phylogenetics is, "are these two tree distributions the same?"
Perhaps most commonly,[^1] we ask this when we have multiple independent MCMC runs and we want to know if they're sampling from the same distribution.[^2]
But other questions can be rephrased into this framework, including whether two loci have the same underlying phylogeny.

I was reading through Susan Holmes' "Statistical approach to tests involving phylogenies"[^3] and came across an approach I found intriguing. 
It uses a tree of trees to attack the problem, and as a permutation test it's pretty interpretably straightforward.
I'll walk through the test, and then take it for a spin to see how it does.

If you're one of those people who doesn't want to read a long post about a method that you probably don't want to use, turn back now.

## Holmes' two-sample tree test
As presented in "Statistical approach to tests involving phylogenies," the test is a two sample test for equality of distribution.
It is immediately generalizable to more samples, so we'll just assume we have $K \geq 2$ samples, and we want to know if we can reject the null hypothesis that there is a single underlying tree distribution (one group).

We start by choosing a tree-to-tree distance metric and getting the Minimum Spanning Tree (MST) for the trees (according to that metric).
This is a tree, though not in the usual phylogenetic sense.
As a spanning tree, the MST connects _all_ the sampled trees in a tree (it has no cycles or loops, so it is a tree), and as the _minimum_ spanning tree, the MST takes the shortest path to do so.

Each edge of the MST connects two trees, which may either be from the _same_ sample of trees or a different sample.
The test statistic, $S_0$, is the number of edges which connect trees in the same sample.
The larger $S_0$ is, the more distinct the sets of trees are, at the extreme all but $K - 1$ edges will connect trees in the same sample.
But, how big is surprisingly big?
Big enough that we can reject the null hypothesis of a single group?

We need a null distribution.
What would things look like if there really was only one group?
Well, every tree in each of our sets of tree samples is independent and identically distributed (IID).
And, more importantly, since every tree is IID, the labels are arbitrary. 
That is, which tree goes in which set of samples is arbitrary, since they're all the same thing.
So, we can generate our null distribution by permuting the labels (the assignment of each tree to one of the $k \in 1, \dots, K$ groups).
The p-value for this test is the proportion of permutations of the labels of the trees which produces a larger test statistic.

There are many possible permutations of the labels.
If there are $n$ total trees and $n_k$ in any given sample set, there are $n! / (n_1! \times n_2! \times \dots n_K!)$ permutations.[^4]
In the most extreme case, with $S_0 = n - 2$, we can compute the p-value by hand.
There are no ways to produce a larger value, and only $K!$ ways to produce a value as large.[^5]
But, outside this, the number of permutations to consider quickly gets large, so we resort to sampling.
We take $B$ (permutation) bootstrap replicates, and estimate the p-value as the proportion of those which have a larger test statistic, $p = 1/B\, \sum_b \mathbb{I}(S^{*}\_b > S_0)$.

There are a lot of things to like about this setup, including that it's relatively intuitive and straightforward.
It's also pretty computationally efficient to boot.
To see the efficiency, consider what would happen if we did something else, like trying to bootstrap the tree samples, and not the labels.
This would allow us to put a confidence interval on $S_0$ (not that we necessarily care) and still test the one-group null hypothesis.
But if we did this, we'd have to find $B$ MSTs, which is a cost that adds up fast, especially as $n$ gets big.


### A note on the distance metric
When Holmes introduces this test, it is with a tree distance metric that includes branch lengths, and everything works smoothly.
When dealing with a purely topological distance, the MST may not be unique.
Non-unique MSTs occur when there are identical distances between trees, which is essentially impossible with a branch-length-inclusive distance metric (which is continuous) but entirely possible with a purely topological distance metric (which is discrete).
In such a situation we can get different answers depending on which of the equally-minimal spanning trees we end up with.
Depending on how that MST is chosen, this could be disastrous.
If, given several equidistant neighbors, we always chose the one with the next index, for example, we would bias $S_0$ by artificially making more edges within sets of samples than between.[^6]

## But does it work?

Yes.
And no.

Or maybe no and yes.

Or more accurately, it works if you know what it's testing, and does not if you think it's testing something different.

## What is the two-sample tree test actually testing?
I was drawn to Holmes' two-sample tree test less out of interest in the Bayesian tree convergence problem and more out of a desire to address questions of topological congruence.
This is a gnarly problem and the obvious solutions require a substantial amount of work.[^7]
So I hoped that this would provide a faster approach.
Get a sample of trees for both loci, use a topological distance measure, and run the test, right?
Wrong!

In hindsight, it is kind of obvious why this was a fool's errand.
But perhaps if the reason was not immediately obvious to me, it will not be immediately obvious to someone else out there, and looking at why I was wrong will help.

This is a test that the trees come from the same distribution.
The _exact_ same distribution.
Where I went wrong was what that means.

Two independent MCMC runs under the same model should produce samples from the exact same distribution.
Two independent sets of bootstrap replicates under the same model should produce samples from the exact same distribution.
Same data.
Same inference procedure.
Same distribution.

Two analyses under the same model of two different loci, even when the true tree is the same for both, are not from _the exact same distribution_.
In the land of finite data that is the real world, different loci will have different mutational histories, even if only slightly.
This will produce different distributions on branch lengths and different amounts of certainty about splits in the tree.
I don't know where this line is drawn, but certainly loci of very different sizes will contain different amounts of information and the distributions will reflect that.
(Consider this yet another reason that asymptotic properties of phylogenetic estimators are often unhelpful.)

## When should we use Holmes' two-sample tree test?

This test appears to be pretty powerful.
One might also describe it as tetchy.

I have not applied it extensively to samples from real Bayesian analyses.
In fact, I've only mucked around with a single example.
But autocorrelation seems to be a serious issue.
I took two runs which I was pretty sure sampled from the same distribution and ran the test, $p < 1/1000$.
I tried thinning to account for the [effective sample size](https://projecteuclid.org/journals/bayesian-analysis/advance-publication/How-Trustworthy-Is-Your-Tree-Bayesian-Phylogenetic-Effective-Sample-Size/10.1214/22-BA1339.full),[^8] $p < 1/1000$.
I tried thinning to dozens trees, instead of hundreds, and found that the test seemed to reject pretty consistently for much more than 30 trees out of 900 (already thinned and post-burnin) when the effective sample size is on the order of 200.

Now, I could be wrong.
The tree distributions really could be different, but I don't think so, and I don't think we should expect the test to start passing at extreme thinning values if that were true.
I think that the thinning regime where the test passes corresponds not so much to the effective sample size as it does to the distance at which autocorrelation between the samples becomes undetectable.[^9]
Which is a pretty stringent standard.
"Run the chains long enough such that you can have a few hundred trees pass Holmes' two-sample tree test" is certainly a convergence standard one could advocate for.
I get the sense, though, that for many practical problems where we want to do phylogenetic inference it is a pretty strict standard that sits somewhere between "unachievable on a large fraction of said problems" and "far stricter than is useful."
Then again, maybe having a stringent standard out there as a counterweight to the usual laissez-faire approaches would be good.

What about other circumstances?
We certainly can't use it to test for congruence among loci.
We don't usually need to test if two sets of bootstrap replicates are the same distribution,[^10] unless perhaps we're testing a new fast approach and want to check that it works.
So I admit, I'm coming up blank.

Perhaps some day we'll have MCMC efficient enough to pass the test easily.
Perhaps I will find a use I have overlooked thus far for the test.
Or perhaps the test is most useful to get us thinking about how to compare tree distributions.

## Salvaging the test in the face of non-unique MSTs
I'll end with the note that we can "rescue" Holmes' test from the non-unique MST problem, at the cost of significantly more computation.
Not that it's really much of a problem, because there really aren't many (any?) circumstances when we should be using it with a purely topological distance metric.

The idea is to simply average Holmes' test statistic across the non-unique MSTs.
Since there are potentially rather a lot of these, we'll probably have to settle for taking, at random, a set of $R$ of them.
This does seem to work when I play around with it in R.
However, to get the p-value to stabilize across replication, it appears that $R$ needs to be quite large, say 1000, possibly more.
This means computing $R$ times as many test statistics, but as getting the MST is slower still, the big slowdown is getting all $R$ of those.

[^1]: At least among those of us who use Bayesian approaches and are foolish enough to peel back the veil and stare into the abyss of multi-chain convergence.
[^2]: Depending on your knowledge of statistics and phylogenetic inference, this might sound like an absurdly simple problem. A phylogenetic tree topology is a discrete object, so a set of samples of tree topologies is a multinomial distribution, and testing multinomials is easy! And that last part is true (multinomials are great), but phylogenetic posterior distributions are eldritch horrors. Two independent MCMC runs can be sampling the same target and _never once sample the same tree in a finite run_. A phylogenetic posterior distribution can be remarkably "flat" at the level of trees while still being (somewhat) informative about features of those trees. I think this is one of the core folk theorems of phylogenetics, and it's just one reason that when people want to compare tree distributions they turn to things like the ASDSF (average standard deviation of split frequencies) which focus on features which are informed.
[^3]: Chapter 4 in Gascuel's "Mathematics of Evolution and Phylogeny."
[^4]: I may be missing rotational symmetry, but since the spanning tree is fixed, I think this frees us from the issue.
[^5]: Though this does raise the question of how to handle ties, should it be the proportion of permutations which produces as large or larger of a test statistic?
[^6]: Something like this appears to happen if one directly uses igraph's MST implementation, though not for ape's.
[^7]: What are the "obvious" approaches? I'd say ML or Bayesian model selection. We could compare two models, one where the loci share a tree topology, and one where they get their own topologies. But this requires us to infer both the topology-linked and topology-unlinked models (to borrow MrBayes' terminology). And, since I don't think anybody knows what the degrees of freedom difference would be here for a likelihood ratio test, we'd need to simulate the null distribution with a parametric bootstrap. Which means hundreds of analyses of simulated datasets (simulate the unpartitioned model, analyze with both, track the likelihood difference). Surely Bayes rides to our rescue, right? Well, since I don't think anyone has implemented this model comparison in reversible-jump, we'd need marginal likelihoods, so, no. We could try [Paul Lewis' information-based approach](https://academic.oup.com/sysbio/article/65/6/1009/2281635), which absolves us of the need for marginal likelihoods. That only requires fitting both models in a Bayesian framework. But the approach has yet to be widely adopted, and so we don't have quite the years of collective experience which tell us how to interpret the strength of evidence like we do with likelihood ratios or Bayes factors.
[^8]: I don't feel quite so bad about shameless self-citation on a blog post.
[^9]: This idea is implemented in [treess](https://github.com/afmagee/treess) in the `jumpDistanceBootstrapUnsmoothedESS` and `jumpDistanceBootstrapESS` methods. Which we show in the paper do not correspond to the ESS and are far more stringent. All those non-independent samples in the intervening time lags still contribute useful information.
[^10]: Taking bootstrap trees from one analysis and repeatedly dividing them up into two random subsets produces an apparently uniform distribution of p-values. So it does seem to work as expected here.