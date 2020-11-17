---
title: "Research"
filename: _pages/research.md
permalink: /research.html
---

## Birth-death modeling
Birth-death models link time-calibrated phylogenies to birth, death, and sampling rates through time.
In macroevolutionary contexts, this means estimating rates of speciation, extinction, and fossilization, while in infectious disease phylodynamic contexts we use some algebra these parameters to estimate the effective reproductive number.
I have worked to develop a time-varying birth-death prior, the [Horseshoe Markov random field](https://doi.org/10.1371/journal.pcbi.1007999) (HSMRF) birth-death model, that is applicable to both macroevolutionary and phylodynamic cases.
By providing temporal smoothing, the HSMRF allows for fine-scaled estimation of birth and death rates through time.

## Phylogenetic model adequacy
For a long time, the phylogenetics modeling community was primarily concerned with choosing the *best* model out of a set of candidates.
For example, taking a phylogeny and asking if birth rates were constant, piecewise constant, or increased exponentially.
This is a question of model selection.
In recent years, the phylogenetics community has become increasingly interested in whether any of these models actually fit the data at hand.
This is a question of model adequacy.
In a Bayesian framework, we can address the matter of model adequacy through posterior predictive simulations, where we simulate new datasets that our (fit-to-data) model predicts and compare attributes of these datasets to our real data.
In my research, I have found that alignment-based posterior predictive checks may not be able to [distinguish between tree priors](https://doi.org/10.1111/biom.13273), but they are capable of [detecting epistasis]().

## Improving phylogenetic inference
Inferring phylogenies, and the parameters of phylogenetic models, can be a painful, and painfully slow, process.
Implmenting phylogenetic models such as the HSMRF birth-death model requires carefully thinking about MCMC moves and operators, especially because we cannot often rely on tricks used in other fields.
I am interested in how we can build smarter proposals for phylogenetic models, and especially for phylogenetic trees, to speed up Baysian phylogenetic inference.
I am also interested in alternatives to standard Bayesian phylogenetic inference, which I helped investigate in a pair of papers, one on [quickly computing the marginal likelihood of tree topologies](https://doi.org/10.1093/sysbio/syz046), and one on [exploring treespace without MCMC](https://doi.org/10.1093/sysbio/syz047).
