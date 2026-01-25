---
title: "Multifurcating birth death models"
author_profile: true
layout: single
---

Branching process models for phylogenies that allow for multifurcating trees, and not just binary ones, have been in the literature for some time.
You have your $\Lambda$-coalescents and your [Beta-coalescents](https://arxiv.org/pdf/2407.14976)[^1] and such, where you hear tale of heavy-tailed offspring distributions and how those can lead to such distributions.[^2]
I'm going to stay out of population genetic theory and in the realm of phylodynamics, where I go back and forth on this topic.
Are there ever really _simultaneous_ infections?
Perhaps not, but maybe close enough on the time scales our trees take up.
Though would we ever see them?
I'm certainly not used to datasets where we sample infections that densely, but it could still leave signatures we care about in the tree.

I like a good forward-time generative model, and the other day I stumbled across some pieces that make me think it could actually be pretty easy to get tractable[^3] birth-death phylodynamic models with multi-births.
But to get there, we need some background.

## The short-ish version

Deriving the likelihood function for a phylogenetic birth-death model more or less amounts to figuring out how two quantities change through time.
I've called them [$D(t)$ and $E(t)$ before](https://www.biorxiv.org/content/10.1101/2021.01.14.426715v1.full.pdf), but others call them [$q(t)$ and $p\_0(t)$](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003919).
I'll stick with D and E, and note that the lack of subscripts implies that we're restricting ourselves to models where all lineages alive at some time $t$ are exchangeable.
So, time-dependent birth-death models yes, clade-dependent models no.

I like to think of $D(t)$ as the probability of straight lines in the observed phylogeny.
Slightly more rigorously, it's the probability of a single lineage surviving from time $t$ (measured in time before the present) to 0 (the present).[^4]
For a lineage surviving from some time $t_o$ to some more recent $t_y$, the probability is $D(t_o) / D(t_y)$.
Thus, the birth-death likelihood of a tree can be expressed as a product of $D(t_o) / D(t_y)$ terms, plus terms for the observed births and samples taken.
In turn, $D(t)$ depends on $E(t)$, the probability that a lineage alive at time $t$ leaves no sampled descendants in our tree.

To actually get anywhere with a birth-death model derivation, we start by thinking about $\mathrm{d}E/\mathrm{d}t$ and how to integrate it.
Then we think about $\mathrm{d}D/\mathrm{d}t$ and how to integrate it.
Then we sprinkle in the probabilities of births and other events we can see, namely births and sampling events.
We can ignore sampling, which can get complicated, and we'll get to the births later.

## What are we talking about changing?

Before we look at how each of these changes if we allow for multiple births, we need to be clear what we mean.
In a typical birth-death model, there's a birth rate $\lambda(t)$ at which one lineage gives birth to another.
I'm suggesting our model instead says that there's a rate $\lambda(t)$ at which one lineage gives birth to _some number_ of additional lineages.
That is, to simulate forward, when your process says that the next event is a birth in lineage $\ell$, you draw the number of children from some distribution, $\nu \sim \mathrm{Pr(\nu)}$.

## $E(t)$

I prefer to think about $E(t + \Delta t)$ to $\mathrm{d}E/\mathrm{d}t$.
That is, in a sufficiently small window of time, how does the extinction probability change?
Then you can rearrange and divide through and get the differential needed.

Now I'm going to add novel notation in the possibly misguided hopes that it will help abstract away from model details.
Let's call $\mathcal{E}$ be the event that the lineage goes extinct, $\mathcal{N}$ the event that nothing happens to it, $\mathcal{B}$[^5] the event that it experiences a birth event.
Then let $\mathrm{Pr}(\mathcal{E}| t, \Delta t)$ the probability that it goes extinct between $t$ and $t + \Delta t$.

Extinction probabilities are propagated according to equations of the form
```math
E(t + \Delta t) = \mathrm{Pr}(\mathcal{N}| t, \Delta t) E(t) + \mathrm{Pr}(\mathcal{E}| t, \Delta t) + \mathrm{Pr}(\mathcal{B}| t, \Delta t) E(t)^2
```
These correspond to the three things that can happen in the interval: absolutely nothing, it goes extinct, or there's a birth.
If nothing happens, then it still has to go extinct eventually, hence the $E(t)$ in that term.
If it goes extinct, end of story, extinction achieved.
And if there's a birth, both lineages have to then go extinct, hence the $E(t)^2$.

If we want to allow multi-births, only this last term changes, because now there are many possibilities: a birth event with 1 child, 2 children, and so on.
All of these will eventually have to go extinct, so the equation becomes
```math
E(t + \Delta t) = \mathrm{Pr}(\mathcal{N}| t, \Delta t) E(t) + \mathrm{Pr}(\mathcal{E}| t, \Delta t) + \mathrm{Pr}(\mathcal{B}| t, \Delta t) E(t) \left[ \sum_{\nu} E(t)^\nu \mathrm{Pr}(\nu) \right]
```

So, if we want this to work and be tractable, we need a $\mathrm{Pr}(\nu)$ that plays nicely with this functional form and produces an analytical solution.
Because if it doesn't (even approximately), we can't get at one of the most important quantities in our likelihood.

Also, let's be careful, because I have now done the math assuming that at a birth event one lineage survives, and some number $\nu$ of _additional_ lineages arise.
We could assume that the original one dies and then $1 + \nu$ lineages are born, but this formulation should play better with PMFs of most distributions, so if we want to think that way, we should do the math this way, and then reinterpret later.

### A distribution is just a sequence with partial sums we know[^6]

What would play nicely with a sum of this form?
The [geometric distribution](https://en.wikipedia.org/wiki/Geometric_distribution) seems a likely candidate, with parameter $\theta \in [0,1]$ and random variable $\nu$ the PMF is
```math
\mathrm{Pr}(\nu) = \theta(1 - \theta)^{\nu - 1}
```
Where the Geometric goes, typically goes the [Negative Binomial](https://en.wikipedia.org/wiki/Negative_binomial_distribution), of which it is a special case.
Adding parameter $r$, and shopping around for the [right parameterization](https://en.wikipedia.org/wiki/Negative_binomial_distribution#Alternative_formulations) we get,
```math
\mathrm{Pr}(\nu) = {\nu + r - 1 \choose \nu} \theta^\nu(1 - \theta)^r
```
There's also the [Poisson distribution](https://en.wikipedia.org/wiki/Poisson_distribution) (which allows any $\theta > 0$),
```math
\mathrm{Pr}(\nu) = e^{-\theta} \frac{\theta^\nu}{\nu!}
```
A final candidate is the [Logarithmic distribution](https://en.wikipedia.org/wiki/Logarithmic_distribution), for $\theta \in (0,1)$
```math
\mathrm{Pr}(\nu) = \frac{-1}{\ln(1 - \theta)} \frac{\theta^\nu}{\nu}
```

It was the Logarithmic I was staring at when this whole idea really took shape.
It shows up in compound Poisson contexts, which is a reason to be hopeful it might work here.
That it naturally lives on $\nu \geq 1$ instead of $\nu \geq 0$ may or may not have much utility, though it would be nice to not have to work with conditional PMFs.
The Geometric can, depending on how you define write it, live on either space; I've written it above for $\nu \geq 1$.
The Negative Binomial rears its head in a lot of places, though often as a _result_ of mucking with probability distributions.
In this case, the Geometric looks more plausible, since it's already off by a $1 - \theta$ from the form we want, and the Negative Binomial just keeps going that way.

## $D(t)$

Propagating $D(t)$ looks similar.
In the typical case,
```math
D(t + \Delta t) = \mathrm{Pr}(\mathcal{N}| t, \Delta t) D(t) + 2 \mathrm{Pr}(\mathcal{B}| t, \Delta t) D(t) E(t)
```
This corresponds to two possibilities.
Either nothing happens in the interval, so the straight line we see in the tree was what really happened in that time, or there's a birth where all the other lineages go extinct, and the straight line we see is simply all that's left.
I think we can interpret the 2 as ${2 \choose 1}$, that is, pick one of the two lineages to survive, as we can't distinguish them.

The generalization would then be,
```math
D(t + \Delta t) = \mathrm{Pr}(\mathcal{N}| t, \Delta t) D(t) + \mathrm{Pr}(\mathcal{B}| t, \Delta t) D(t) \left[ \sum_{\nu} {\nu \choose 1} E(t)^\nu \mathrm{Pr}(\nu) \right]
```

Does this tell us anything more about distributions that might work for $\mathrm{Pr}(\nu)$?
Maybe.
The ${\nu \choose 1}$ term looks a bit awkward being added to the Geometric, and turns the $1 / \nu!$ in the Poisson into a $1 / (\nu - 1)!$.
Both of those tilt the sequences in ways that seem less than promising to make the math work out.
On the other hand, it simply cancels the Logarithmic's $1 / \nu$ in the denominator, leaving a cascading sum of $(E(t) \theta)^\nu$ terms, which ought to sum up nicely.[^7]

## $\lambda(t)$

I was about to start doing the math,[^7] when I realized I had missed something.
In a typical birth-death model, the contribution of the births observed at times $\mathbf{t}$ to the likelihood is very straightforwardly the product of the birth rate at all times of observed births
```math
\prod_i \lambda(t_i)
```
But we have undone the stipulations that make it this straightforward.
Now it's not that every birth leads to two lineages, both of which we see.
Now every birth even leads to some number of children, some (usually) lesser number of which we see.
That last bit smuggles in the extinction probability.
Say we see a trifurcation in the tree: were there two offspring lineages, and we're seeing all, or might there have been eight?[^8]

Let there be $\nu$ lineages descending from the birth event in the tree, and $\nu\_\mathcal{E}$ be the number of unobserved extinct lineages, then the likelihood ought to be
```math
\prod_i \lambda(t_i) \left[ \sum_{\nu_\mathcal{E} \geq 0} {\nu + \nu_\mathcal{E} \choose \nu} E(t_i)^{\nu_\mathcal{E}} \mathrm{Pr}(\nu + \nu_\mathcal{E}) \right]
```

## Attempting mathematics

With all that said and done, and having written far more than I expected to by this point, I am now going to try actually making this work.
I don't think I'm going to actually finish the math, but I'm going to squint at things and see if I think there's hope for finishing it up later.
I think the thing to do is start with the sum that looks the hardest, and work our way towards easier ones.
Without showing my work, I'll say I tried the Logarithmic and gave up.
The $1 / \nu$ in the PMF reacted poorly with the combinatorial coefficient in the probabilities of birth events, leaving a stray random variable lying about.

### $\lambda(t)$ revisited

Replacing $E(t)$ with $E$ to reduce clutter, we want the following to work out to something nice
```math
\sum_{\nu_\mathcal{E} \geq 0} {\nu + \nu_\mathcal{E} \choose \nu} E^{\nu_\mathcal{E}} \theta (1 - \theta)^{(\nu + \nu_\mathcal{E}) - 1}
```

```math
\sum_{\nu_\mathcal{E} \geq 0} {\nu + \nu_\mathcal{E} \choose \nu} E^{\nu_\mathcal{E}} \theta (1 - \theta)^{\nu_\mathcal{E}} (1 - \theta)^{\nu - 1}
```

```math
\theta (1 - \theta)^{\nu - 1} \sum_{\nu_\mathcal{E} \geq 0} {\nu + \nu_\mathcal{E} \choose \nu} E^{\nu_\mathcal{E}} (1 - \theta)^{\nu_\mathcal{E}} 
```

Define $p := E (1 - \theta)$
```math
\frac{\theta (1 - \theta)^{\nu - 1}}{(1 - p)^{\nu + 1}} \sum_{\nu_\mathcal{E} \geq 0} {\nu + \nu_\mathcal{E} \choose \nu} p^{\nu_\mathcal{E}} (1 - p)^{\nu + 1}
```

Note that
```math
{\nu + \nu_\mathcal{E} \choose \nu} = \frac{(\nu + \nu_\mathcal{E})!}{\nu! \nu_\mathcal{E}!} = {\nu + \nu_\mathcal{E} \choose \nu_\mathcal{E}}
```

Ergo
```math
\frac{\theta (1 - \theta)^{\nu - 1}}{(1 - p)^{\nu + 1}} \sum_{\nu_\mathcal{E} \geq 0} {\nu + \nu_\mathcal{E} \choose \nu_\mathcal{E}} p^{\nu_\mathcal{E}} (1 - p)^{\nu + 1}
```

We can now recognize the summand as the PMF of a $\mathrm{NegativeBinomial}(\nu_\mathcal{E}; \nu + 1, p)$ random variable, which means it evaluates to 1.
Before we even think about cleaning it up, we need to see that the other terms work out.

### $D(t)$ revisited

We need this sum to work out too
```math
\sum_{\nu \geq 1} \nu E^\nu \theta (1 - \theta)^{\nu - 1}
```

```math
\frac{\theta}{1 - \theta} \sum_{\nu \geq 1} \nu E^\nu (1 - \theta)^{\nu}
```

Define $(1 - p) := E (1 - \theta)$
```math
\frac{\theta}{p(1 - \theta)} \sum_{\nu \geq 1} \nu (1 - p)^\nu p
```

We can expand the summand to include $\nu = 0$ where it is 0,
```math
\frac{\theta}{p(1 - \theta)} \sum_{\nu \geq 0} \nu (1 - p)^\nu p
```
And now we can recognize the sum as the expectation of a $\mathrm{Geometric}(\nu; p)$-distributed random variable,[^10] but the 0-including kind of Geometric, so we can simplify to
```math
\frac{\theta}{p(1 - \theta)} \frac{1 - p}{p} = \frac{\theta (1 - p)}{p^2(1 - \theta)}
```

### $E(t)$ revisited

The last, though perhaps most important, thing that we need to work out nicely is
```math
\sum_{\nu \geq 1} E(t)^\nu \theta (1 - \theta)^{\nu - 1}
```

```math
\frac{\theta}{(1 - \theta)} \sum_{\nu \geq 1} E(t)^\nu (1 - \theta)^{\nu}
```

Using $p := E (1 - \theta)$ again
```math
\frac{\theta}{(1 - \theta)} \sum_{\nu \geq 1} p^\nu
```

This is everyone's favorite geometric sequence, with a sum of $p / (1 - p)$, yielding
```math
\frac{\theta p}{(1 - \theta) (1 - p)}
```

## So where are we?

I haven't checked my math sufficiently to be sure I'm right, but I think even with some inevitable slips, there will be an analytical form for all three of the terms we've looked at.
Which means that there are analytical solutions to $D(t + \Delta t)$ and $E(t + \Delta t)$, and a derivation of an analytically tractable multi-birth-death model is possible.
The details of actually making the math work out will depend on the model itself, and I'd be shocked if something didn't get ugly.[^11]
But it looks possible to me.

This model could be useful in phylodynamics when sampling is sufficiently good to actually see superspreading in a tree, or perhaps when sampling isn't quite as good, but there's still more heterogeneity in offspring distribution than a usual time-varying birth-death model.[^12]
Or it could be useless.
But it might be fun to find out.

[^1]: Which, just for fun, _are_ $\Lambda$ coalescents. And for extra fun, write out Beta.
[^2]: To their credit, [Zhang and Palacios (2024)](https://arxiv.org/pdf/2407.14976), who I linked to, also discuss in a somewhat more concrete way, places we might want models of this ilk.
[^3]: Speculative, simulation-only models have their place. But I like models I can fit to data, and I like likelihoods. Yes, I'm a hopelessly basic Bayesian at times.
[^4]: There are alternative ways to define this. You'll often find the p/q notation with a time endpoint other than the present.
[^5]: I'll include in here the "and only a birth" stipulation of the usual "there's only one event in this window" assumptions. E.g., you can have a multi-birth but not a birth and a death.
[^6]: You can get far in probability by being able to do math. But you can get farther faster by being able to skip the math and say "well, that's just the kernel of a squingleforth distribution, and thus we know..."
[^7]: Yes I could just start doing the math, but I won't. Math is hard and I want some notion I'm barking up the right tree before I get started.
[^8]: Actually, for that matter, are we seeing the original lineage and two children, or three descendants? Does it matter? I think that, given our assumptions about lineages being exchangeable, it's all the same thing. But I think one would very much have to be careful when taking this model and trying to BiSSE-fy it, allowing lineage-dependency. Because I'm about to make good on the assumption that I can toss around a bunch of equivalent $E(t)$ terms for a bunch of lineages coming out of the birth event, and that might not be true. As always, caveat generalizator.[^9]
[^9]: I'm pretty sure that that's pig latin. It's my blog and I'll get things wrong if I want to.
[^10]: Sure, we could have just tackled the sum directly. And I was thinking about it, but then I saw the resemblance and saved myself all that effort.
[^11]: Well, uglier, anyways. Some time-varying phylogenetic birth-death model likelihoods are real doozies.
[^12]: We could also just adopt a multi-type birth-death model where birth rates for individuals are drawn randomly from some distribution. But that's likely to end up going down the road of a BiSSE-style likelihood, and those require numerical integration, which it looks like we might be able to avoid here.