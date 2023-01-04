---
title: "A well-supported tree isn't (necessarily) a good tree"
author_profile: true
layout: single
---

I want to talk about something that bugs me.
I might call it a common, if pernicious, misunderstanding.
It's about support for phylogenies.
But really it's about uncertainty.
Or maybe it's about precision[^1].

This is a mistake you've likely heard others make, even if neither you nor they knew it was a mistake.
It goes something like this, "We tried a handful of different approaches for inferring the tree[^2] and this one produced the best-supported tree, so we're going with that."
The gist of this, is, "the best estimate is the one with the least uncertainty."
This is wrong.
Let's take a look at a few reasons why.

## A brief aside on uncertainty
Because it's helpful for my thought process, and for seeing links between concepts, I'm going to look at non-phylogenetic examples as well as phylogenetic case studies.
In more classical statistical contexts, uncertainty about estimates can be simple to express.
In Bayesian statistics we get posterior distributions, which we can describe in terms of variance, entropy, or the width of credible intervals.
In frequentist statistics, we get standard errors which impact the widths of confidence intervals.

Phylogenies are a bit more difficult.
We've been doing statistical phylogenetics for decades, but in some ways the "statistical" part of that is still the wild west.
In Bayesian contexts, we could talk about the [Fréchet variance](https://academic.oup.com/sysbio/article/69/1/139/5511414) of the posterior, the number of trees in a credible set, or perhaps how well-resolved a consensus tree is.
In maximum likelihood (frequentist!) contexts could consider the Fréchet standard error, or the size of [confidence sets](https://www.tandfonline.com/doi/full/10.1080/01621459.2017.1395342), or perhaps how well-resolved the bootstrap consensus tree is.
I'm going to go with the somewhat more nebulous "strength of support" for a summary tree, since that's generally how I hear the fallacy at hand phrased.

## Stronger support does not mean a better model

First up: is the model that produces the better-supported tree necessarily the better model?

### A hypothetical non-phylogenetic example
Imagine you're interested in the relationship between the size at adulthood and size at birth.[^3]
You get a bunch of health data and start modeling this, only to realize after you're through that your data comes from a bunch of families.
So, you go back to the drawing board, and concoct a new model.
This one has family clusters, within which there is covariance due to relatedness, with the strength of this effect estimated for good measure.
Then you compare these models and find that the independent-person model has a much narrower confidence interval for the birth-to-adult weight parameter than the family-clustered model.
Does this mean you've built a worse model by adding in family covariance?
No!
You've built a demonstrably more realistic model, and that model has more accurately reflected the _true_ uncertainty at hand.

### A phylogenetic case study
Now for a phylogenetic example, starring substitution models on a dataset I had lying around.
It's a little old dataset from [Hedges _et al._ (1990)](https://pubmed.ncbi.nlm.nih.gov/2283953/), an amniote-focused phylogeny.
Sometimes it gets referred to as DS1, as it was the first of a number of datasets used in [Lakner _et al._ (2008)](https://academic.oup.com/sysbio/article/57/1/86/1704335).
The tree itself isn't the greatest, but it's a common benchmark dataset.[^4]

I used two distinct models on this dataset, I'm going to call them $\mathcal{M}\_1$ and $\mathcal{M}\_2$ for now.
The only difference between them is the substitution model, otherwise both were bog-standard phylogenetic models fit in IQTREE.[^5]
Both models produced the same maximum likelihood tree, but they give different sense about the support for the tree.
$\mathcal{M}\_1$ and $\mathcal{M}\_2$ agreed on the support for 4 of the 24 splits in the tree, while $\mathcal{M}\_1$ had higher support for 14 and lower support for 6.
$\mathcal{M}\_1$ had, on average, about 3% higher support for splits than $\mathcal{M}\_2$, and a minimum support 6% higher.
All in all, I'd say $\mathcal{M}\_1$ suggests that the tree is better-supported than $\mathcal{M}\_2$.

So, which model is better, $\mathcal{M}\_1$ or $\mathcal{M}\_2$?
Trick question! 
We can't say from the information I've given so far.
In fact, $\mathcal{M}\_2$ has a log-likelihood 402.4 log-likelihood units higher than $\mathcal{M}\_1$, and the $\Delta$-AIC in favor of $\mathcal{M}\_2$ of 786.9.
It turns out that $\mathcal{M}\_2$ is actually a _much_ better model.
This should not be surprising, because $\mathcal{M}\_1$ was Jukes-Cantor, while $\mathcal{M}\_2$ was GTR with gamma-distributed among-site rate variation.

A model is not better than another just because it has higher bootstrap support values, larger posterior probabilities, or a smaller 95% credible set.

_Uncertainty about an estimate is not a measure of model fit._

## Strong support does not mean a halfway good model
Next up: does a strongly-supported tree say anything about how good our model is in the absolute sense?

### A hypothetical non-phylogenetic example
Say you're looking at the relationship between the time a person spends napping and their age.
I don't know what shape this should be, exactly, but I know that young children and the elderly nap more than young adults.
Something u-shaped might be appropriate.
Say you choose to fit a simple linear model instead.
The more samples you add, the smaller the confidence intervals get on all the slopes until they basically vanish.
But your model is still _wrong_.

### A phylogenetic case study
Let's consider a phylogenetic example.
This one comes from a paper by [Paul Lewis and colleagues](https://academic.oup.com/sysbio/article/65/6/1009/2281635).
It's a very small dataset indeed, 5 tips, 456 sites, all plants (angiosperms).
The paper has already done all the analyses we need, so I'm not going to analyze anything here myself.[^6]

When Lewis and co. ran a Bayesian analysis of the dataset, they found a single tree in the posterior.
That's a lot of support!
A posterior probability of 1.0.
You may be wondering what the catch is, and it is horizontal gene transfer.
Half the locus supports a sane tree, with a posterior probability of 0.77 for the MAP tree.
The other half, due to horizontal gene transfer from a monocot, moves bloodroot, a eudicot, into the monocots[^7] with a posterior probability of 0.64 for the MAP tree.
The tree that gets the entire posterior to itself in the concatenated analysis isn't _good_, it's just the only one found in the posteriors of both halves.
In fact, there is no single tree.
There are two!
So the concatenated model is _completely inadequate_.

A model is not adequate just because it has very high bootstrap support values, large posterior probabilities, or a very small 95% credible set.

_Uncertainty about an estimate is not a measure of model adequacy._

## The end?
I hope we can see that our certainty about an estimate has very little to do with anything other than _our certainty about an estimate_.

What follows is some more philosophical musing about the issues at hand.
They are not necessary for understanding the point being made, and if anyone reads this, you would be perfectly justified in stopping here.

## Bias and variance, accuracy and precision
Why do we care about model fit and model adequacy?
Model adequacy is the easier one to understand: if our model misses key features of reality, we probably should not trust it very far at all.
Model _fit_ is a bit fifferent.
I think the logic here goes something like:
1. We cannot know the truth.
2. We hope that the best-fitting model will be the one closest to the true data-generating process.[^8]
3. Therefore we hope that the best-fitting model will give us the best possible answer.

The fallacy we're concerned with here lies, I think, in conflating model fit with the amount of uncertainty a model's estimate produces.
Or maybe it's wrong because it directly assumes that an estimate which has less uncertainty must be closer to the truth.
I'm not quite sure, but either way there's a fallacy at hand.
Because the amount of uncertainty we get around an estimate has nothing to do with either how good the estimate is or how well the model fits.
Uncertainty is uncertainty.
In phylogenetics, we have to deal with the fact that our data are outcomes of a stochastic mutational process and that means we will always have uncertainty about our estimates.

I think the issue here is in a sense related to a somewhat larger one, which is mistaking precision for accuracy.
To be clear, accuracy and precision are properties of _estimators_, statistical procedures for trying to answer a question.
That is, these are not things we can know about from a single dataset.[^10]
In general, the matter of how close an estimate will tend to be to the true value (accuracy/bias) is decoupled from how close repeated estimates are from each other (precision/variance).

Now, we were not talking about estimators, but particular estimates and their uncertainty.
For simpler models, there are well-documented direct links between all the things we're talking about.
In multiple regression, we can directly see _how_ adding parameters increases uncertainty about our estimates.[^11]
In phylogenetics, things are more complicated.
But I think that, in general, we can expect the same idea to apply.
At the very least, we should not be _surprised_ when a more complex model produces more uncertainty about the tree.
This is not to say that all certainty is false certainty,[^12] just to hammer home the point that we should never expect certainty to be an indicator that we've done a good job of inference.

## Coda
I think this is all aided and abetted by peoples' inherent aversion to uncertainty.
People like certainty and clear-cut answers.
But nothing short of magic, certainly not science nor statistics, can conjure certainty from thin air.
The best we can do in an uncertain universe is to _quantify_ our uncertainty and do our best to get on with things.

[^1]: Or variance, if you'd rather.
[^2]: This could be maximum likelihood and Bayesian inference under the same model, different partitioning schemes, smashing a bunch of SNPS together versus modeling them separately, the list goes on.
[^3]: This could be a very simple model, like $E[w_{\text{adult}}] = \beta \times w_{\text{birth}}$.
[^4]: What's that saying? Every benchmark dataset is wrong in it's own way? At least with likelihood it doesn't put mammals as sister to birds like neighbor-joining and parsimony.
[^5]: Which is very fast and let me get a thousand bootstrap replicates in less time than it took me to tab over to my email. But the support values changed a bit uncomfortably at 1000 when I changed the seed, so I upped it to 10,000 which was still done in under half a minute. This makes me wonder if we're all collectively doing enough bootstrap replicates, though.
[^6]: The paper has also written the analyses up better than this very quick overview. The overall writing is excellent, and the methods it devises are very cool. I highly suggest reading it in full.
[^7]: That this is what is happening is perhaps easier to see in [the study from which the dataset is subsampled.](https://www.nature.com/articles/nature01743)
[^8]: Which is itself, as it turns out, a whole other kettle of fish. Smarter people than me have written about issues with model selection in the real world where all of our models are wrong. I really like [Danielle Navarro's piece on Bayes Factors](https://psyarxiv.com/nujy6).
[^10]: Nor even any number of _real_ phylogenetic datasets. We never know the truth, so accuracy/bias is right out. And we cannot draw independent and identically-distributed alignments because evolution happened once, so we can't even get at the easier question of precision/variance. I suppose in cases where we're willing to assume a single tree we could perhaps use subsampling to approximate this and try to get at it.
[^11]: Somewhat more precisely, [adding parameters decreases the degrees of freedom in the sampling distribution,](https://book.stat420.org/multiple-linear-regression.html#sampling-distribution) which increases the width of the confidence intervals. This is about _sampling distributions_, which is also what accuracy/bias and precision/variance are about.
[^12]: Nonreversible substitution models have many more parameters than reversible ones. But, if they better-capture the substitution process, they can actually increase certainty about certain features in the tree. For example, say we infer that the G->A rate is much higher than the A->G rate, and compare to a reversible model. The reversible model would not be able to decide between topologies requiring A->G and G->A substitutions. But the nonreversible model will strongly favor topologies with G->A substitutions. 