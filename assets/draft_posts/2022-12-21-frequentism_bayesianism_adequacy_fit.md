---
title: "What is a Bayesian t-test?"
author_profile: true
layout: single
---

## What is a Bayesian t-test?

For some time now[^1] I've wondered whether the things we talk about as Bayesian analogs of null hypothesis significance tests (hereafter NHST because I'm lazy) are what we say they are.
This has been on my mind even more after reading one of [Dan Simpson's blog posts](https://dansblog.netlify.app/posts/2022-11-12-robins-ritov/robins-ritov.html), which had as a moral, "If you’re going to make Bayes work for you, think in terms of observables rather than parameters."
I think this statement will serve to crystallize the difference I've been trying to pin down, but it might take me awhile to get there.
Since examples are easier for me to grasp, I'm going to think about this first in terms of t-tests, then a phylogenetic example, and finally I'll attempt mathematics[^3] for a more general answer.

The ultimate question I think I'm asking here is, "what is really the Bayesian analog of NHST?"
The answer, which should probably not be surprising, is posterior predictive model checks.
The reason this should perhaps not be surprising is that posterior predictive checks produce things called posterior predictive p-values, NHST is based on p-values, and nothing with p-value in the name ever enters into Bayesian hypothesis testing.

### 

[^1]: That is, long enough that I've forgotten when I first wondered it. So at least a month.[^2]
[^2]: Yes, I'm doing footnotes now. I read Douglas Adams at a formative age, I've been reading a lot of Terry Pratchett, and Dan's blog only served to encourage me. My asides will probably be less informative and entertaining than all of theirs.
[^3]: You've been warned.