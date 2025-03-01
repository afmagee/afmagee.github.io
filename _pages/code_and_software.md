---
title: "Code and software"
filename: _pages/code_and_software.md
permalink: /code_and_software/
---

## R
- I am the lead developer of [treess](https://github.com/afmagee/treess), a package for diagnosing the convergence of MCMC samples of phylogenies and other objects beyond the reach of standard approaches. Provides [Fréchet](https://en.wikipedia.org/wiki/Fr%C3%A9chet_mean) generalizations of the effective sample size and visual diagnostics.
- I am the lead developer of [nbbp](https://github.com/cdcgov/nbbp), a package for inference of the effective reproduction number from final size outbreak data under a negative binomial branching process model.
- R code for two computational labs I helped to develop is available [here](https://github.com/afmagee/BIOL_481_W2018).
One lab explores the [Luria-Delbruck experiment](https://en.wikipedia.org/wiki/Luria%E2%80%93Delbr%C3%BCck_experiment) and modifications thereof.
This lab is designed to work in conjunction with a wet lab recapitulating the Luria-Delbruck experiment, but can be done with any data from such an experiment (including the original data from Luria and Delbruck).
The other lab explores models of population cycling, and includes all necessary datasets.


## python
- I am the lead developer of [cladecombiner](https://github.com/cdcgov/cladecombiner), a package leveraging phylogenetic taxonomy to streamline the process of aggreagating viral lineages for nowcasting and short-term forecasting of variant dynamics.
- I have contributed to the development of tooling to test the [efficacy of variant nowcasting models](https://github.com/CDCgov/cfa-viral-lineage-model) via retrospective analyses.
- I helped create a simulation based [python streamlit app](https://github.com/cdcgov/cfa-ring-vax-widget) for exploring the behavior of branching processes in the presence of multiple observation processes.


## Other

### [RevBayes](https://github.com/revbayes/revbayes)
RevBayes is a probabilistic programming language written in C++. It offers a friendly scripting-based interface for model specification, a large number of phylogenetic models, and highly customizable MCMC. In order to streamline the user experience, the original repository and its history was [archived](https://github.com/revbayes/revbayes.archive/) and the rebooted version is now in use. Among other things, I have contributed
  - A general time-varying model of a partially-observed branching process (now [here](https://github.com/revbayes/revbayes/blame/master/src/core/distributions/phylogenetics/tree/birthdeath/BirthDeathSamplingTreatmentProcess.cpp), originally [here](https://github.com/revbayes/revbayes.archive/blob/master/src/core/distributions/phylogenetics/tree/birthdeath/EpisodicBirthDeathSamplingTreatmentProcess.cpp)). I also wrote a [tutorial on using this model](https://revbayes.github.io/tutorials/divrate/efbdp_me.html).
  - An implementation of the [elliptical slice sampler](https://proceedings.mlr.press/v9/murray10a/murray10a.pdf) (now [here](https://github.com/revbayes/revbayes/blob/master/src/core/moves/EllipticalSliceSamplingSimpleMove.cpp), originally [here](https://github.com/revbayes/revbayes.archive/blob/master/src/core/moves/EllipticalSliceSamplingSimpleMove.cpp))

### [BEAST X](https://github.com/beast-dev/beast-mcmc/)
BEAST X is an extraordinarily long-lived scientific software project for phylogenetic inference, in service since the [early aughts](https://code.google.com/archive/p/beast-mcmc/source/default/commits?page=68).
Users can specify a wide variety of phylogenetic models through an XML interface, while the source code itself is written in java and many of the more computationally intensive tasks are offloaded to C++ via the [BEAGLE library](https://github.com/beagle-dev/beagle-lib).
Among other things, I have
  - Assisted in the implementation of [efficient](https://github.com/beast-dev/beast-mcmc/blob/master/src/dr/evomodel/speciation/EfficientSpeciationLikelihood.java) and [differentiable](https://github.com/beast-dev/beast-mcmc/blob/master/src/dr/evomodel/speciation/SpeciationLikelihoodGradient.java) likelihoods for [partially-observed branching processes](https://github.com/beast-dev/beast-mcmc/blob/master/src/dr/evomodel/speciation/NewBirthDeathSerialSamplingModel.java).
  - Written code for [tracking](https://github.com/beast-dev/beast-mcmc/blob/master/src/dr/evomodel/speciation/NewBDSSHistorySimulator.java) the implied total number of infected individuals from a partially-observed branching process.
  - Implemented the [ExpGamma distribution](https://github.com/beast-dev/beast-mcmc/blob/master/src/dr/inference/distribution/ExpGammaDistributionModel.java).
