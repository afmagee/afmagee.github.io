---
title: "Code and Software"
filename: _pages/code_software.md
permalink: /code_software/
---

I am the principle devloper for the R package [treess](https://bitbucket.org/afmagee/treess/), which is useful for diagnosing the MCMC output of Bayesian phylogenetic analyses.
The primary use of the package is to compute the effective sample size for phylogenetic trees, via one of a number of implemented approaches.
There is also functionality for comparing multiple chains while accounting for within-chain variability, and for assessing the stability of a single run via block-bootstrapping.

I contribute to the open source phylogenetic inference software [RevBayes](https://revbayes.github.io/), including code for birth-death and coalescent likelihoods, univariate distributions, and MCMC moves.

I have also contributed significant code to the following paper-associated repositories:
- Magee *et al.* (2021) [Preprint](https://arxiv.org/abs/2109.07629). [Code](https://bitbucket.org/afmagee/tree_convergence_code/).  Repository contains code designed to test whether phylogenetic effective sample size measures are useful or not via simulation.
- Magee and Hoehna (2021) [Preprint](https://www.biorxiv.org/content/10.1101/2021.01.14.426715v1). [Code](https://github.com/afmagee/gefbd). Repository contains code for a fossilized birth-death with mass extinctions analysis of Crocodylomorpha, as well as simulations based on that analysis, and a number of sensitiity analyses.
- Magee *et al.* (2020) [Paper](https://academic.oup.com/mbe/advance-article/doi/10.1093/molbev/msab163/6287069). [Code](https://github.com/WSDeWitt/phyload/).  Repository contains code for a simulation study to characterize how well phylogenetic inference performs in the face of epistasis. There is also RevBayes code implementing the epistatic doublet model of [Nasrallah and Huelsenbeck (2013)](https://doi.org/10.1093/molbev/mst108).
- Magee *et al.* (2020). [Paper](https://doi.org/10.1371/journal.pcbi.1007999). [Code](https://github.com/afmagee/hsmrfbdp). Repository contains code for simulating phylogenies under time-varying birth-death models and analyzing them with the Horseshoe Markov random field birth-death model I developed in this paper. Also includes code for join inference of time-calibrated phylogenies and time-varying birth-rates (with the Horseshoe model) from sequence data using RevBayes.
- Faulkner *et al.* (2020) [Paper](https://doi.org/10.1111/biom.13276). [Code](https://github.com/jrfaulkner/phylocode/). Repositoy contains code for analyses of population genetic and phylodynamic datasets under time-varying coalescent models. Jim Faulkner and Vladimir Minin developed the models, I implemented them in RevBayes.

For my work on birth-death processes, I have also contributed to the following RevBayes tutorials, which walk you through how to set the model up on your own data.
- Estimating mass extinctions from phylogenies of extinct and extant taxa. [Link](https://revbayes.github.io/tutorials/divrate/efbdp_me.html)
- Estimating diversification rates through time with a Horseshoe Markov random field model. [Link](https://revbayes.github.io/tutorials/divrate/ebd.html)

R code for two computational labs I helped to develop is available [here](https://github.com/afmagee/BIOL_481_W2018).
One lab explores the [Luria-Delbruck experiment](https://en.wikipedia.org/wiki/Luria%E2%80%93Delbr%C3%BCck_experiment) and modifications thereof.
This lab is designed to work in conjunction with a wet lab recapitulating the Luria-Delbruck experiment, but can be done with any data from such an experiment (including the original data from Luria and Delbruck).
The other lab explores models of population cycling, and includes all necessary datasets.
