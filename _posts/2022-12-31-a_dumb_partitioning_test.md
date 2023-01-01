---
title: "Partitioning done wrong"
author_profile: true
layout: single
---

Sometimes when my brain is a bit stuck, and working is hard, I find it useful to think about off-the-wall[^0] sorts of phylogenetics.
Between the largest higher-ed labor strike in US history and the holidays, I've been pretty decidedly off my A-game.[^1]
So I've been casting around for something, and I found my attention drawn back to one of the perpetually hard bits of phylogenetics: partitioning.

## What's all this, then?

Phylogenetic partitioning is, broadly, the question of what, if any, parameters should be shared between the different loci in our dataset.
Though in practice it's mostly thought of as specifically the question of what parameters of the _substitution model_ should be shared among loci.
Primarily, like most things to do with the substitution model, I think this over-emphasis is because it's the easiest part of the model to deal with, rather than it being the most important question.

Let's take a look at why it's easiest to just think about partitioning in terms of the substitution model.
To be a bit more concrete in notation, let's call the tree topology (and just the topology) $\tau$, the branch lengths $\boldsymbol{\nu}$, and the miscellaneous other substitution model parameters $\boldsymbol{\theta}$.
Let's say we have $L$ loci, and since I'm feeling like an applied statistician today, let's call the alignment for any locus $\boldsymbol{Y}\_l$.[^2]
A simple unpartitioned model has the same model for all loci, so the likelihood is $\text{Pr}(\boldsymbol{Y} \mid \tau, \boldsymbol{\nu}, \boldsymbol{\theta}) = \prod^L\_{l=1} \text{Pr}(\boldsymbol{Y}\_l \mid \tau, \boldsymbol{\nu}, \boldsymbol{\theta})$.
The maximally partitioned model has different _everything_ for each different locus, $\text{Pr}(\boldsymbol{Y} \mid \boldsymbol{\tau}, \boldsymbol{\nu}, \boldsymbol{\theta}) = \prod^L\_{l=1} \text{Pr}(\boldsymbol{Y}\_l \mid \tau\_l, \boldsymbol{\nu}\_l, \boldsymbol{\theta}\_l)$.
Generally in unrooted land we put our fingers in our ears, hum loudly, and assume a single tree for all loci,[^3] making the maximally partitioned model you'll ever find in practice, $\text{Pr}(\boldsymbol{Y} \mid \tau, \boldsymbol{\nu}, \boldsymbol{\theta}) = \prod^L\_{l=1} \text{Pr}(\boldsymbol{Y}\_l \mid \tau, \boldsymbol{\nu}\_l, \boldsymbol{\theta}\_l)$.
Most commonly, because it requires way less computation, we assume the same branch lengths for each locus, at least up to an overall rate multiplier, which I'll hide in $\boldsymbol{\theta}$, making the maximally partitioned model you'll probably ever ask a program to consider, $\text{Pr}(\boldsymbol{Y} \mid \tau, \boldsymbol{\nu}, \boldsymbol{\theta}) = \prod^L\_{l=1} {Pr}(\boldsymbol{Y}\_l \mid \tau, \boldsymbol{\nu}, \boldsymbol{\theta}\_l)$.
For $s$ sequences in a tree, there are (give or take rooting) $2s - 3$ branches, while there are rarely more than 10 free parameters in $\boldsymbol{\theta}\_l$.
So this last model has way fewer parameters than one where the branch lengths also differ.

I say the _maximally_ partitioned model.
What's in between the minimum and the maximum?
Some loci will share parameters with other loci.
Let's say that we've partitioned the dataset into $K$ subsets.
And because this will be much easier this way, let's bastardize and mix our notation, and use $l \in \mathcal{K}_k$ to denote the set of locus indices which are in the $k$th subset.
Then, a more-than-minimally, less-than-maximally, partitioned dataset has a likelihood, $\text{Pr}(\boldsymbol{Y} \mid \tau, \boldsymbol{\nu}, \boldsymbol{\theta}) = \prod^K\_{k=1} \prod\_{l \in \mathcal{K}_k} {Pr}(\boldsymbol{Y}\_l \mid \tau, \boldsymbol{\nu}, \boldsymbol{\theta}\_k)$.[^4]
This is just how we express mathematically the fact that some loci share some bits of the model.[^5]

## No small parts

There are two reasons that partitioning is a pain in phylogenetic analyses.
They are deeply intertwined.

Partitioning is hard because there are so many possible partitioned models.
Even in the somewhat simplified framework we're using here (we aren't allowing for two subsets to share some parameters but not others), we're dealing with [Bell numbers](https://en.wikipedia.org/wiki/Bell_number).
These get big.
Fast.
Not quite as big as treespace, but big.
So either finding a best partitioning scheme for maximum likelihood purposes, or sampling from partitioning schemes for Bayesian purposes, is going to be hard and slow.

Given that, we'd like to choose a partitioning scheme before we run the analysis.
But, the matter of choosing one requires us to go through phylogenetic modeling.
Which means that it's as hard as doing the analysis in the first place.

With some strong assumptions to enable streamlining the process, programs like PartitionFinder and IQTREE's ModelFinder can make a pretty decent stab at partitioning in pretty impressively short times.
But still, it would be nice if we could circumvent the phylogenetic part, wouldn't it?

## What if we took the trees out of trees?
How can we circumvent phylogenies for a phylogenetic question?
This is the sort of thing I spend way too much time thinking about.[^6]
In this case, the thought occurs, what if we ignored the tree and went straight to genetic distances?
Distances are simpler, and don't require the whole tree thing.
On the other hand, there are a _lot_ of them, as the number of pairs of sequences grows as the number of sequences squared.
And so I beat on, a boat very much _against_ the current of good ideas.

### This is a blog post so I can talk about bad ideas that didn't work

Let's try to get as far away from the usual phylogenetic modeling assumptions as possible.
In general, phylogenetic substitution models work by parameterizing an instantaneous rate matrix $Q$ and defining the transition probability matrix over a branch of length $\nu$ as $P = \exp(Q\nu)$.
Now, to hell with $Q$, let's get rid of it.
For any pair of sequences we can just count, and estimate $\hat{P} = n_{ij}/n_{i\cdot}$.
We've now avoided committing to any particular substitution model (we're even leaving reversibility behind), and even to the notion of a homogeneous substitution model across the tree.

Now what?
We can estimate a directional substitution probability matrix for every pair of sequences.
But we've abstracted away the actual distances.
How are we going to know anything about partitioning?

Let's assume $\boldsymbol{Y}\_1$ and $\boldsymbol{Y}\_2$ come from the same model and consider the implications.
The fact that they have the same tree, $\tau$, with the same branch lengths, $\boldsymbol{\nu}$, means that the paths between any pair of sequences have the same branches between them.
The fact that the substitution model parameters are the same means that, regardless of it's functional form, or whether it is branch-heterogeneous, for a pair of sequences $u$ and $v$, we should expect to see the same substitution patterns for both $\boldsymbol{Y}\_1$ and $\boldsymbol{Y}\_2$.

So, as long as we're careful to keep the direction the same (that is, keep $u$ as the starting sequence and $v$ as the ending sequence), we should be able to compare likelihoods under two models.
In the unpartitioned model we have $P_{uv}$, while in the partitioned model we have $P_{uv}^1$ and $P_{uv}^2$.
We can estimate these pretty easily with maximum likelihood[^7].

Now, there's still the matter of every other sequence pair.
And of course, the distances between those are all dependent through the underlying phylogeny.
But, let's boldly ignore that and just smash them all together.[^8]
Then we'll follow up with a likelihood ratio test and hope for the best.[^9]

Now, you may or may not be surprised to find out that this does not work very well.
I tested it in a handful of simulations, and it does a great job rejecting the unpartitioned model when the truth is there are two partitions.
But it also rejects the unpartitioned model most of the time when it's the true model.
As in, the false positive rate is something like 50%.
I'm probably paying the price for treating the pairwise comparisons as independent.

### But wait, there's more
So now what?
We know we have a dependence problem, and our first horrible idea didn't work.
We could give up, but that's boring.
We could try to find some correction to the degrees of freedom.
But that sounds hard.
So let's try changing tack.

For each of our two subsets, $\boldsymbol{Y}\_1$ and $\boldsymbol{Y}\_2$, we can relatively easily estimate distances between all sequence pairs, $\hat{\boldsymbol{d}}^1$ and $\hat{\boldsymbol{d}}^2$.
Since they depend on the tree and branch lengths, we might be interested in some measure of how different these two sets of branch lengths are, as a surrogate for the tree.
Let's take the absolute sum $D_{\text{obs}} = \sum_{i>j} \\! \mid \\! \hat{d}^1\_{ij} - \hat{d}^2\_{ij} \mid$.
If we can figure out what the distribution of this quantity under the null model of only one subset, then we can _test_ for whether there is a single subset or not.

There are a few difficulties with this null distribution.
For one, won't we have to condition on a phylogenetic model, which is exactly what we were trying to avoid?
And, more importantly, won't the math be hard?
Thankfully, computational approaches come riding to our rescue like knights in slightly buggy armor.

Let's do this as a _permutation test_.
In general, permutation tests can allow us to test whether things come from the same distribution without making many assumptions about that distribution.
If there is no distinction in the data-generating model producing $\boldsymbol{Y}\_1$ and $\boldsymbol{Y}\_2$, then it doesn't matter how sites are assigned to subsets.
So if we examine all possible assignments of sites to subsets (that match the original sizes of the real data) and look at the distribution this gives us on $D$, that tells use whether we can reject the unpartitioned model or not.[^10]

This approach shockingly well.
I tried testing it on four little simulated scenarios: same everything, different branch lengths, different trees (and branch lengths), and different models (same tree and branch lengths).
At $\alpha = 0.05$, we get a 0% false positive rate when the null is true and the tree, branch lengths, and substitution model are the same.
When the truth is that the trees and/or branch lengths are different, but the substitution model is the same, we have a power of over 90%!
Interestingly, when I simulated different substitution models on the same tree (with the same branch lengths), it always failed to reject.
This could be due to my choice of the logDet distance[^11], which is somewhat agnostic to the substitution process.
In a sense this is nice, because it suggests the approach isn't sensitive to the substitution model.
If this pattern bears out, it means we could perhaps apply techniques like this where we care about the tree but not the substitution model, like recombination detection.

[^0]: Read: somewhere between potentially silly and definitely stupid.
[^1]: Not that that means much, mind you. Also, this is not an endorsement of working over the holidays! It's more like the mental equivalent of trying to lose that extra few holiday pounds.
[^2]: I'd also accept $D$ for data, lower case $y$, or bold versions of either. I'm not really good enough at notation to be that much of a stickler.
[^3]: Though really, would we know what to do with more? We know how to handle one tree, and we sort of know how to handle a bunch of completely unlinked loci, but the great middle is pretty under-modeled. We have the notion ancestral recombination graphs, which are great, except for the whole bit where they're incredibly hard to work with. Then again, I think most things in the middle will be.
[^4]: Which we can see covers both extremes. In the unpartitioned model $K = 1$ and we drop the outer product. In the fully partitioned model, $\mid \mathcal{K}\_k \\!\\! \mid \, = 1$ (all subsets have one entry) and we drop the inner product.
[^5]: As it turns out, like basically everything in ~~phylogenetics~~ ~~statistics~~ life, it's all about arranging sums and products. Marc has been trying to teach me this for a long time now. Sorry it took me this long, Marc!
[^6]: Strangely enough, though, it has actually led to me learning a fair bit of statistics.
[^7]: Since MLE for observed Markov models is pretty straightforward, we estimate the rate matrix by counting. So for the first partition in the partitioned model, $\hat{p}\_{uv,ij}^1 = n_{uv,ij}^1/n_{uv,i\cdot}^1$. I am truly sorry for the ballooning sub- and super-scripts. But with this many things being indexed, there may not be much of a cleaner way.
<!-- Under the unpartitioned model, for this pair of sequences, the log likelihood[^777] at the maximum is $(\sum_{ij} (n_{uv,ij}^1 \log(\hat{p}\_{uv,ij}))) + (\sum_{ij} (n_{uv,ij}^2 \log(\hat{p}\_{uv,ij})))$.
For the partitioned model, the log likelihood at the maximum is $(\sum_{ij} (n_{uv,ij}^1 \log(\hat{p}^1\_{uv,ij}))) + (\sum_{ij} (n_{uv,ij}^2 \log(\hat{p}^2\_{uv,ij})))$. -->
[^8]: I think this might, very (very) generously, be called a composite likelihood.
[^9]: That is, we'll define the unpartitioned likelihood to be, $\mathcal{L}\_{\text{unpart}} = \sum_{ij} ((\sum_{ij} (n_{uv,ij}^1 \log(\hat{p}\_{uv,ij}))) + (\sum_{ij} (n_{uv,ij}^2 \log(\hat{p}\_{uv,ij}))))$. The partitioned likelihood will be, $\mathcal{L}\_{\text{part}} = \sum_ij ((\sum_{ij} (n_{uv,ij}^1 \log(\hat{p}^1\_{uv,ij}))) + (\sum_{ij} (n_{uv,ij}^2 \log(\hat{p}^2\_{uv,ij}))))$. Then we'll take $-2(\mathcal{L}\_{\text{unpart}} - \mathcal{L}\_{\text{part}})$ to be $\chi^2$-distributed with $12 \times {s \choose 2}$ degrees of freedom (because there are $s$ sequences and ${s \choose 2}$ pairs of sequences, each with a difference 12 free parameters).
[^10]: I've already written way more math than anyone ever will want in this post, so let's bring it home. Call $\mathcal{B} = \\{\mathcal{B}^1,\mathcal{B}^2\\}$ the set of all bipartitions of the concatenated dataset into two datasets with the same numbers of columns as the original. We're interested in the proportion of these which have a larger $D$ than the observed value, $1/\|\mathcal{B}\| \sum_{b^1,b^2 \in \\{\mathcal{B}^1,\mathcal{B}^2\\}} \mathbb{I}(D < D_{\text{obs}})$. There are way too many possible rearrangements, so we instead estimate this by sampling from them at random.
[^11]: Which I suppose might technically not be a distance.