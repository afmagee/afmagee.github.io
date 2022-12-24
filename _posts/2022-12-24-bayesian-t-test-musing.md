---
title: "What is a Bayesian t-test?"
author_profile: true
layout: single
---
<!-- # What is a Bayesian t-test? -->

<!-- Note to future self: be careful with subscripts (_) in LaTeX, it can really break things on, but \_ is safe, for some reason-->
<!-- Of course, the opposite is true for the preview renderer in VSCode, because of course it is -->

For some time[^1] I've wondered whether the things we talk about as Bayesian analogs of null hypothesis significance tests are what we say they are.
This has been on my mind even more after reading one of [Dan Simpson's blog posts](https://dansblog.netlify.app/posts/2022-11-12-robins-ritov/robins-ritov.html), which had as a moral, "If you’re going to make Bayes work for you, think in terms of observables rather than parameters."
(This statement will prove, I think, to crystallize the difference I've been trying to pin down, but it might take me awhile to get there.)
In particular, what has drawn my attention is the notion of a Bayesian t-test, so I'm going to focus here on t-tests, and answer the slightly more focused question in the title than the broader one I began the paragraph with.
I'll probably[^8] write some more general thoughts on this broader question as a follow-up.

## My kingdom for a t-test

Two-sample t-tests are ways of asking, "is the mean of this thing, $Y_1$, different from the mean of that thing, $Y_2$?"
First, we ~~build our straw man~~ posit our null hypothesis: the mean of this thing is not different from the mean of that thing.
As ~~noted eugenecist Ronald Fisher demands~~ frequentist statistics runs on sampling distributions, we need to construct a test statistic and work out its sampling distribution under the null hypothesis, such that we can determine how implausible our observations would be if our null were true[^4].
Since we live in the 21st century and we're going to make a computer do all the actual math, we'll do away with the utterly unnecessary assumption that the variances of the quantities of interest are equal.

We'll have an easier time if we flesh out our model a little more.
Namely, we assume that both $Y_1$ and $Y_2$ are Normally-distributed, with the same mean but different variances.
Therefore (because Normal distributions are friendly like that) the sample means for both, $\bar{Y_1}$ and $\bar{Y_2}$, are Normally-distributed, as is the difference between them, $\Delta = \bar{Y_1} - \bar{Y_2}$.
The mean of $\Delta$ is 0, and the standard deviation is $\sigma_{\Delta}$ which is estimated using the standard errors of the sample means, $\text{SE}\_{\bar{Y_1}}$ and $\text{SE}\_{\bar{Y_1}}$, as $\sqrt{\text{SE}^2\_{\bar{Y_1}} + \text{SE}^2\_{\bar{Y_2}}}$[^5]. 
The [test statistic](https://en.wikipedia.org/wiki/Welch%27s_t-test#Calculations), for Welch's t-test, is the observed difference in means divided by its standard error, $\Delta / \text{SE}\_{\Delta} = (\bar{Y_1} - \bar{Y_2})/\sqrt{\text{SE}^2\_{\bar{Y_1}} + \text{SE}^2\_{\bar{Y_2}}}$[^9].
This is approximately[^6] t-distributed, hence the name of the test.

So what we're doing, then, is imagining what sorts of (appropriately scaled) differences in means we would observe if the null model/hypothesis were true.
This is the sampling distribution of the test statistic under the null model.
Then we can pick a particular implausibility factor (an $\alpha$ value) and use our null model to define values of the (appropriately scaled) observed difference in means which would be implausibly big (and/or small, this is the rejection region).
Then if our observed difference falls into this region, we reject the null model as implausible.

## Bayesian t-tests, take one
Okay, so if that's a frequentist's take on a t-test, what's a Bayesian take?
I will admit that I have never _run_ a Bayesian t-test, or studied the topic intensely. 
But when one asks google, or google scholar, what one is, one tends to quickly find Bayes Factors turning up.
And since I do know a bit about Bayesian hypothesis testing, let's define some models and get on with it.

Our first model, $\mathcal{M}\_1$, will be basically the same as the t-test null model.
We have our data $\boldsymbol{y}$ which is composed of two subsets, $\boldsymbol{y_1}$ and $\boldsymbol{y_2}$.
The likelihood will be,  
$\boldsymbol{y_1} \overset{iid}{\sim} Normal(\mu,\sigma_1^2)$.  
$\boldsymbol{y_2} \overset{iid}{\sim} Normal(\mu,\sigma_2^2)$.  
That is, both groups of observations are assumed to be drawn from Normal distributions which share a mean, but not a variance.
What about the priors, you ask?
Well, what about them? 
They're going to be pretty tangential here.  
$\mu \sim \text{Pr}(\mu)$  
$\sigma_1,\sigma_2 \overset{iid}{\sim} \text{Pr}(\sigma)$  
Fill these in with whatever sparks joy.

Now, as it will turn out, we also need a second model, $\mathcal{M}\_2$.
This one will let each group have its own mean as well as variance.
The likelihood will be,  
$\boldsymbol{y_1} \overset{iid}{\sim} Normal(\mu_1,\sigma_1^2)$.  
$\boldsymbol{y_2} \overset{iid}{\sim} Normal(\mu_2,\sigma_2^2)$.  
Again, the actual priors are rather tangential to the point.
But we should probably use the same priors for $\mu_1$ and $\mu_2$ as for $\mu$ above for fairness, though.  
$\mu_1,\mu_2 \overset{iid}{\sim} \text{Pr}(\mu)$

We could also parameterize the model in terms of the difference in means, which is the actual quantity of interest.
But, the way I see it, there isn't a way to do that with both proper priors and the same distributions for $\boldsymbol{y}\_1$ and $\boldsymbol{y}\_2$, which I don't like.
And more importantly, the overall point isn't going to be changed, so we're going to do it this way.[^7]

For model $\mathcal{M}\_1$, the posterior distribution is given by $\text{Pr}(\mu, \sigma_1, \sigma_2 \mid \boldsymbol{y}) = \text{Pr}(\boldsymbol{y} \mid \mu, \sigma_1, \sigma_2) \text{Pr}(\mu) \text{Pr}(\sigma_1)\, \text{Pr}(\sigma_2) / \text{Pr}(\boldsymbol{y})$.
The denominator, $\text{Pr}(\boldsymbol{y})$, is a constant, which is why we usually ignore it in Bayesian inference.
We've sort of hidden an important conditioning, though, because we are conditioning on the model, $\mathcal{M}\_1$, so really this is $\text{Pr}(\boldsymbol{y} \mid \mathcal{M}\_1)$.
Adding this back in shows us why this denominator, known better as the marginal likelihood, is important: it represents the evidence for the model.
It is the probability of generating the data given the model, integrating out all the parameters (according to their uncertainty).
We get it by integrating the likelihood (hence it is the _marginal_ likelihood), $\text{Pr}(\boldsymbol{y} \mid \mathcal{M}\_1) = \int_{\sigma_1}\,\int_{\sigma_2}\, \int_{\mu} \text{Pr}(\boldsymbol{y} \mid \mu, \sigma_1, \sigma_2) \text{Pr}(\mu) \text{Pr}(\sigma_1)\, \text{Pr}(\sigma_2)\, \text{d}\mu\, \text{d}\sigma_2\, \text{d}\sigma_1$.

We can compare $\text{Pr}(\boldsymbol{y} \mid \mathcal{M}\_1)$ to $\text{Pr}(\boldsymbol{y} \mid \mathcal{M}\_2)$ to compare the relative strength of evidence in favor of $\mathcal{M}\_1$ and $\mathcal{M}\_2$.
If $\text{Pr}(\boldsymbol{y} \mid \mathcal{M}\_2) > \text{Pr}(\boldsymbol{y} \mid \mathcal{M}\_1)$, we have evidence that the means are different.
A Bayes factor represents this as the ratio of the marginal likelihoods, $BF_{2,1} = \text{Pr}(\boldsymbol{y} \mid \mathcal{M}\_2) / \text{Pr}(\boldsymbol{y} \mid \mathcal{M}\_1)$, the larger the ratio the stronger the evidence.
As with p-values, there are cutoffs that are commonly used for different strengths of evidence.
That is we might declare there to be a difference in means if $BF_{2,1} > 10$, meaning model $\mathcal{M}\_2$ is 10x more probable than model $\mathcal{M}\_1$.
Bayes Factors can be nice because they cut both ways (you can support the null model), and because they directly involve probabilities of models (the conditioning goes the way many people intuitively want it to go, where p-values are in a sense backwards).

## Bayesian t-tests another way
Now, that's not the only way we can use a Bayesian analysis to determine if there's evidence for a difference in means.
We can also just fit $\mathcal{M}\_2$ and look at the results.
In particular, while it wasn't a model parameter, we can look at the posterior distribution on the compound parameter $\delta = \mu_2 - \mu_1$, or for a two-tailed scenario, $|\delta| = |\mu_2 - \mu_1|$.
We can then use $\text{Pr}(|\delta| \mid \boldsymbol{y})$ to determine if there is a difference in means by adopting a cutoff, $\alpha$, and declaring a difference if $\text{Pr}(|\delta| \mid \boldsymbol{y}) < \alpha$.

## Will the real Bayesian t-test please stand up
However, the thing that bugs me about calling either of the above a Bayesian t-test is that they're built on different frameworks from the frequentist t-test.
We're either comparing the relative fit of a two-mean model to a one-mean model, or we're looking at the posterior distribution of a (compound) _parameter_ from a two-mean model.
A two-sample t-test only ever assesses the fit of the one-mean model, and it does not (directly) use parameters of distributions to do so.
It seems to me that what would really deserve the name "Bayesian t-test" would involve the use of posterior-predictive distributions.

In a posterior predictive framework, we don't just have a model, we also have some summary or statistic of interest, $\mathcal{T}$.
We are interested in comparing the observed value, $\mathcal{T}^{\,\text{obs}}$, to predicted values, $\mathcal{T}^{\,\text{rep}}$.
The idea being that if our observed summary value is a bad match to the predicted values, it indicates our model is not capturing important features of the real world.

Let's take our statistic to be $\mathcal{T} = \Delta / \text{SE}\_{\Delta}$, the test statistic from Welch's t-test.
The posterior predictive distribution is the distribution on new values obtained by first drawing parameter values from the posterior, then new data from the likelihood, and computing the summary.
For model $\mathcal{M}\_1$, that's $\text{Pr}(\mathcal{T}^{\,\text{rep}} \mid \boldsymbol{y}) = \int_{\sigma_1}\, \int_{\sigma_2}\, \int_{\mu} \text{Pr}(\mathcal{T}^{\,\text{rep}} \mid \mu, \sigma_1, \sigma_2)\, \text{Pr}(\mu, \sigma_1, \sigma_2 \mid \boldsymbol{y})\, \text{d}\mu\, \text{d}\sigma_2\, \text{d}\sigma_1$.

_This_ has the right form to match up with the logic of a frequentist t-test.
We're looking not at the parameters, but observables, namely the standardized difference in means.
We're also looking at the distribution of this observable for new hypothetical datasets.
And we're only considering the model we want to _reject_.
There's no model to support, we're just positing a null model and asking "could this plausibly describe our data."

[^1]: That is, long enough that I've forgotten when I first wondered it. So at least a month.[^2]
[^2]: Yes, I'm doing footnotes now. I read Douglas Adams at a formative age, I've been reading a lot of Terry Pratchett, and Dan's blog only served to encourage me. My asides will probably be less informative and entertaining than all of theirs.
[^3]: You've been warned.
[^4]: It isn't.
[^5]: Isn't frequentist statistics _fun_? We get so many sampling distributions we start running out of letters and/or we start stacking subscripts on subscripts like metaphorical turtles.
[^6]: Why is it _approximately_ t-distributed and not actually? Surely, since the sampling distribution of the sample variance is $\chi^2$ and the sum of $\chi^2$-distributed variables is $\chi^2$, the denominator is the square-root of a $\chi^2$-distributed variable and the numerator is Normally-distributed and the whole thing has the [form of a t-distribution](https://en.wikipedia.org/wiki/Student%27s_t-distribution#As_the_distribution_of_a_test_statistic), right? Not so much, as it turns out. The sampling distributions of the sample variances, $\hat{\sigma}\_1^2$ and $\hat{\sigma}\_2^2$, are in fact $\chi^2$. But the denominator is in fact (the square root of) a linear combination of these, $\text{SE}^2\_{\bar{Y_1}} + \text{SE}^2\_{\bar{Y_2}} = \hat{\sigma}\_1^2/n_1 + \hat{\sigma}\_2^2/n_2$. And this [is not $\chi^2$](https://en.wikipedia.org/wiki/Welch%E2%80%93Satterthwaite_equation), though it can be _approximated_ by one. This puzzled me more than maybe it ought to have, but that's what I get for taking mental leaps and not just working through things.
[^7]: It's my party and I'll parameterize things how I like, thank you.
[^8]: Eventually, maybe.
[^9]: Man I hope the math is rendering right for anyone reading this. Apparently there's a difference of opinion between different engines between whether one should use \_ or \\_ in equations, to chaotic results.