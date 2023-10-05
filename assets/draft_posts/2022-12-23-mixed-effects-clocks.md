---
title: "Mixed-effect clock models for fun and profit"
author_profile: true
layout: single
---

Mixed-effect clock models are very, very useful tools for the modern phylogeneticist.
This post comes in part due to discussions I've had with colleagues about how these models work.
These conversations have helped me see that some things that some things I and other statistician-types take for granted obscure the deeper (and, I think, intuitive) truths about these models.
There are a lot of things I want to do here, but the two biggest are to build intuition for these models and to help folks understand how to set them up in BEAST.

## Who is this for?
The primary audience of this is people who are relatively comfortable with phylogenetic modeling and know how to set up a few types of clock models.
I will endeavor to provide sufficient background that the less-experienced might be able to follow, and so that terminology will be clear enough.
But if you've never heard of a phylogeny, this probably isn't a great place to start.
And in general, I will assume more biological/evolutionary knowledge than statistical.

## The short, short version
A mixed-effect clock model is what you get when you smash a local clock model and an uncorrelated clock model together.
Or, as a statistician might say, it generalizes both local and uncorrelated clock models.

## The slightly-less short version

A _strict clock_ model is one which specifies the same rate of evolution for every branch on the tree.
Now, that rate is not necessarily known, it can be (and usually is) estimated.

A _local clock_ model is one where rates are locally strict.
That is, instead of the entire tree sharing the exact same rates, there are a few rates which are shared

A _random local clock_ model is a special kind of local clock model where we don't specify where on the tree the local clocks go.
The number of change points is something we infer, and, importantly, 
This is **not** the sort of local clock model that mixed-effect clock models are built on top of.

## Mixed-effect clock models as log-linear models

### Differences, not absolute rates