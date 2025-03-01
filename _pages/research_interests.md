---
title: "Statistical and scientific interests"
filename: _pages/research_interests.md
permalink: /research_interests/
---

## Bayesian inference
Bayesian inference is a powerful statistical tool for hierarchical models, a useful framework for incorporating prior understanding into models, and a convenient framework for jointly accounting for and propagating uncertainty.
Beyond the development of models to answer scientific questions, I am interested in efficient and scalable inference, model adequacy, and diagnosing MCMC convergence.
I have investigated the usefulness of a variety of approaches for scalable inference, including gradient-based MCMC techniques (both [with](https://academic.oup.com/sysbio/article/73/3/562/7665881) and [without](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1011640) further approximations) and more [ad-hoc approaches](https://doi.org/10.1093/sysbio/syz046).
I have devised model adequacy diagnostics for genomic data to examine [correlated evolution](https://academic.oup.com/mbe/article/38/10/4603/6287069) and for trees to investigate [evolutionary rates through time](https://www.biorxiv.org/content/10.1101/2021.01.14.426715v1).
For model parameters which aren't simple numeric quantities, I have worked to extend notions of the [effective sample size](https://doi.org/10.1214/22-ba1339), so that we can still see whether our MCMC is giving us trustworthy output.


## Branching processes
Branching process models are useful tools in both evolution and epidemiology.
Evolutionary trees can be modeled as partially-observed branching processes, most commonly birth-death models.
In studies of the tree of life, speciation is modeled by births and extinction by deaths.
In epidemiological contexts where genomic sequence data is available, we can also apply these evolutionary methods, with infection and recovery replacing speciation and extinction.
Branching processes can further be useful approximations early in an outbreak or for subcritical regimes.
I have worked on statistical models for [phylogenetic trees](https://www.biorxiv.org/content/10.1101/2021.01.14.426715v1) and [final size data](https://github.com/cdcgov/nbbp), as well as tools for understanding models [via simulation](https://github.com/cdcgov/cfa-ring-vax-widget).
I have devised [Bayesian nonparametric models](https://doi.org/10.1371/journal.pcbi.1007999) for time-varying birth and death rates, and examined the [limits of what can be learned from data](https://www.pnas.org/doi/full/10.1073/pnas.2208851120).

## Evolution
Evolution is both the science of the history of life and a lens through which we can better study life as we know it.
One particularly useful toolkit for studying evolution centers around the phylogeny, an evolutionary tree that describes relationships between, say, species or viral lineages.
Phylogenies can be used as the backbone of [systems of taxonomy](https://github.com/cdcgov/cladecombiner), to investigate [mass extinctions](https://www.biorxiv.org/content/10.1101/2021.01.14.426715v1), to study rates of [viral spread](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1011640), and to examine [the geospatial dispersal patterns](https://academic.oup.com/sysbio/article/73/3/562/7665881) of viruses.
(For a statistical perspective on questions you can ask with a phylogeny, check out [this review paper](https://www.annualreviews.org/content/journals/10.1146/annurev-statistics-033021-112532).)
I am also interested in the process of phylogenetic inference itself.
Trees are a challenging object to infer, which can make it difficult to usefully ask questions like ["what happens when our model's assumptions are bad?"](https://academic.oup.com/mbe/article/38/10/4603/6287069) or ["how can we tell if our model output is trustworthy?"](https://doi.org/10.1214/22-ba1339)
I study these questions with computational experiments and, where possible, mathematical extensions of existing theory.