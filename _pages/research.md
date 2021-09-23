---
title: "Research"
filename: _pages/research.md
permalink: /research/
---

## Birth-death modeling
Birth-death models link time-calibrated phylogenies to birth, death, and sampling rates through time.
In macroevolutionary contexts, this means estimating rates of speciation, extinction, and fossilization, while in infectious disease phylodynamic contexts we use some algebra on these parameters to estimate the effective reproductive number.
I have worked to develop a time-varying birth-death prior, the [Horseshoe Markov random field](https://doi.org/10.1371/journal.pcbi.1007999) (HSMRF) birth-death model, that is applicable to both macroevolutionary and phylodynamic cases.
By providing temporal smoothing, the HSMRF allows for fine-scaled estimation of birth and death rates through time.
I gave a talk (available [here](https://www.youtube.com/watch?v=_b2evwyo9Q4)) with my colleague Bjørn Kopperud on how these HSMRF models interact with the problem of [birth-death model nonidentifiability](https://www.nature.com/articles/s41586-020-2176-1).
With my colleague Sebastian Höhna, I have also developed the [generalized episodic fossilized birth-death model](https://www.biorxiv.org/content/10.1101/2021.01.14.426715v1).
This is a time-varying birth-death model which integrates fossils and extant taxa and thus allows for better estimation of diversification rates through time and mass extinctions than trees of extant taxa alone.
It is also capable of modeling simultaneous speciation across lineages, mass sampling events, and (for use in phylodynamic modeling) treatment.


## Validating Bayesian phylogenetic inference
Phylogenetic models are complex, and it can be difficult to understand how far we can trust our inferences.
There are two related questions that must be addressed: are we confident that our inference procedure worked, and is our model good enough to use?
In Bayesian inference, addressing this first question requires the use of MCMC convergence diagnostics.
I have worked to extend notions of the [effective sample size to trees](https://arxiv.org/abs/2109.07629), so that we can see whether our MCMC is giving us trustworthy output for the key parameter.
The second question is one of model adequacy.
In a Bayesian framework, we can address the matter of model adequacy through posterior predictive simulations, where we simulate new datasets that our (fit-to-data) model predicts and compare attributes of these datasets to our real data.
In my research, I have found that alignment-based posterior predictive checks may not be able to [distinguish between tree priors](https://doi.org/10.1111/biom.13273), but they are capable of [detecting epistasis](https://www.biorxiv.org/content/10.1101/2020.11.17.387365v1).
I have also developed a handful of posterior predictive test statistics applicable for assessing the adequacy of [birth-death models](https://www.biorxiv.org/content/10.1101/2021.01.14.426715v1)

## Speeding and scaling phylogenetic inference up
Inferring phylogenies, and the parameters of phylogenetic models, can be a painful, and painfully slow, process.
Implementing phylogenetic models such as the HSMRF birth-death model requires carefully thinking about MCMC moves and operators, especially because we cannot often rely on tricks used in other fields.
I am interested in how we can build smarter proposals for phylogenetic models, and especially for phylogenetic trees, to speed up Baysian phylogenetic inference.
I am also interested in alternatives to standard Bayesian phylogenetic inference, which I helped investigate in a pair of papers, one on [quickly computing the marginal likelihood of tree topologies](https://doi.org/10.1093/sysbio/syz046), and one on [exploring treespace without MCMC](https://doi.org/10.1093/sysbio/syz047).
