---
title: "Selected publications and presentations"
filename: _pages/publications.md
permalink: /publications/
---

## Scientific publications

This paper is an attempt to ask "is there such a thing as a phylogenetic effective sample size (ESS)?" by asking both "how would we compute it?" and "does it work?"
This split was the key to being able to compare a number of different putative phylogenetic ESS measures we devised (and others had suggested previously) despite wildly different derivations.
The classical ESS (as well as multivariate extensions and revisions to make it more sensitive to between-chain problems) that we are used to seeing in MCMC output is defined in terms of the standard error of the sample mean.
As such, it is relatively straightforward to see how to estimate this quantity from MCMC samples.
But a phylogenetic tree doesn't inhabit Euclidean space, and one can take a number of roads in trying to generalize the ESS to the space of trees.

By saying, effectively, that an ESS measure works if it describes Monte Carlo error, and 

 we were able to computationally test performance of these different approaches.
We concluded that there is such a thing as a phylogenetic ESS, as several different estimators captured Monte C


---

I want to give a bit more explanation of what we did here.

First, I want to note that by epistasis we mean epistasis as modeled by Nasrallah and @HuelsenbeckJohn (2013). Paper here https:/10/10doi.org/1010.1093/10molbev/10mst108

1/10






Also, I note that all phylogenetic inference of simulated data here was Bayesian inference in RevBayes. Don't worry, we ran 2 replicate chains per analysis and checked for convergence.

2/10







We simulated 3375 alignments under the NH2013 model, varying the strength of epistasis as well as the alignment size. Under the NH2013 model, some sites evolve independently while others are paired epistatically. We let both numbers vary.

3/10




Why vary the alignment size? Because it allows us to examine how additional sites influence the accuracy and precision of phylogenetic inference.

We defined a conversion factor r, the relative worth of a paired site in terms of an independently evolving site.

4/10






We chose a few summaries of the posterior distribution to reflect accuracy and precision, and estimated r using our simulations.

If 0 < r < 1 that's good; epistatic sites are useful for inference.

If r < 0 that's bad; epistatic sites make inference worse.

5/10




We estimated r > 0 in all cases, good news for phylogenetics! We found r varies a bit depending on how you define accuracy and precision. We found that paired sites contribute more to precision than to accuracy.

6/10




This means that when there are epistatic interactions in an alignment, phylogenetic inference is a bit more precise than it "should be." In essence, you have three alignment lengths. One real and two effective, and real length > precision length > accuracy length.

7/10




The other thing we did was ask if we can detect epistasis through posterior predictive checks. Posterior predictive checks entail simulating a lot of datasets using posterior model parameters and comparing summaries of these to summaries of the real alignment.

8/10





We developed a new test statistic which seems to work well. For realistic strengths of epistasis, power can easily be >80% (admittedly, for weak epistasis, power can be more like 20%). False positive rate of about 6% at alpha = 0.05.

9/10





The test statistic itself is also pretty simple. You compute a vector containing mutual information values for all n choose 2 possible pairs of sites in an alignment. Then you take the maximum. Easy peasy.

10/10

---


https://www.annualreviews.org/content/journals/10.1146/annurev-statistics-033021-112532

## Scientific presentations
- [How trustworthy is your tree? Bayesian phylogenetic effective sample size through the lens of Monte Carlo error.](https://youtu.be/DCwiQDJ_AG8) Presented at the BayesComp-ISBA Workshop ``Measuring the quality of MCMC output.'' October 2021; online.
- [Birth-death congruence classes can be collapsed using Bayesian shrinkage priors.](https://www.youtube.com/watch?v=_b2evwyo9Q4) Co-presented with Bjørn Kopperud as part of the online [phyloseminar](https://phyloseminar.org/) series.
