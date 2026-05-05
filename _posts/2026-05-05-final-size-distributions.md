---
title: "A different kind of branching process"
author_profile: true
layout: single
---

I remember, 3-4 years ago, starting to hear the term "stuttering chains" tossed around in phylodynamics circles.[^1]
Loosely, this is about tracking disease spread in the regime where transmission dies out, where phylodynamics is usually about tracking bigger, self-sustaining outbreaks.
More recently, I've also dabbled in this regime, in particular in the kinds of models you can use when your data are solely the [final sizes of transmission clusters](https://github.com/cdcgov/nbbp).

The de-facto model of choice here is a negative binomial branching process, so named because every infection causes a number of secondary infections according to a negative binomial distribution.[^2]
In many ways this is nice: it's a highly flexible model that includes Geometric branching processes as a special case and Poisson branching processes as a limiting case.
And the math works out nicely to yield useful quantities in closed form.
The probability mass function on final chain sizes exists in closed form, as do the mean and variance of that distribution.

On the other hand, the negative binomial isn't always exactly the most scrutable distribution.
Sometimes we can get around this by thinking about the negative binomial not in its marginal form, but in its representation as a [mixture of Poissons](https://en.wikipedia.org/wiki/Negative_binomial_distribution#Gamma%E2%80%93Poisson_mixture).
Other times, we still end up stuck with vague notions about relatively higher variance for some parameter values than others.
This is, I think, a common enough feature of continuous mixture distributions: (when) the math works out (it works out) nicely, but discrete mixtures are easier to comprehend.
So, with that in mind, what about a _discrete_ mixture of Poisson distributions for transmission?

## Our model

Let there be a vector of reproduction numbers $\mathbf{R}$, and associated probabilities $\mathbf{p}$, such that an individual has reproduction number $R\_i$ with $p\_i$.
Conditional on $R$, then, an individual infects a $\mathrm{Poisson}(R)$ number of people.


## Where we need to go

Following along with [Lloyd-Smith _et al_. (2005)](https://www.nature.com/articles/nature04153) and [Blumberg and Lloyd-Smith (2013)](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1002993),[^3]

For chain size $j$, we have

$$
\mathrm{Pr}(j) = \frac{1}{(j - 1)!}T_j^{(j - 1)} \, \Big|_{s = 0}
$$

which involves evaluating at $s = 0$ the $(j - 1)$th derivative of

$$
T_j(s) = \frac{Q(s)^j}{j}
$$

The term $Q(s)$ is the probability generating function of the branching process, which can be represented as

$$
Q(s) = \int_0^\infty e^{-u(1 - s)} f(u) \mathrm{d}u
$$

where $f(u)$ is the probability distribution function for individual-level $R$.


## Some (hopefully mostly correct) math

Now we need to backtrack through these steps and hope I don't go and bodge either the Dirac deltas or derivatives too badly.
To start, I think we get

$$
\begin{aligned}
Q(s) &= \int_0^\infty \sum_i e^{-u(1 - s)} p_i \delta(u - R_i) \mathrm{d}u \\
&= \sum_i e^{-R_i(1 - s)} p_i R_i
\end{aligned}
$$

This does reassuringly align with the homogenous Poisson case, where there is one $R = R_1$ and one $p_1 = 1$.

Then we can tackle $T_j(s)$

$$
T_j(s) = T_j^{(0)}(s) = \frac{\left( \sum_i e^{-R_i(1 - s)} p_i R_i \right)^j}{j}
$$

Then, for sanity's sake letting $x := \sum_i e^{-R_i(1 - s)} p_i R_i$, we can apply the product rule to get,

$$
\begin{aligned}
T_j^{(1)}(s) &= \frac{\mathrm{d}}{\mathrm{d}j} \frac{x^j}{j} \\
&= \frac{\mathrm{d}}{\mathrm{d}j} \left( x^j \times j^{-1} \right)\\
&= \frac{x^j \log(x)}{j} - \frac{x^j}{j^2} \\
&= \frac{x^j}{j^2} \left( j \log(x) - 1 \right)
\end{aligned}
$$

This is ugly and going to get uglier.
WolframAlpha provides the following,

$$
T_j^{(2)}(s) = \frac{x^j}{j^3} \left(j^2 \log^2(x) - 2 j \log(x) + 2 \right)
$$

$$
T_j^{(3)}(s) = \frac{x^j}{j^4} \left(j^3 \log^3(x) - 3 j^2 \log^2(x) + 6 j \log(x) - 6 \right)
$$

$$
T_j^{(4)}(s) = \frac{x^j}{j^5} \left(j^4 \log^4(x) - 4 j^3 \log^3(x) + 12 j^2 \log^2(x) - 24 j \log(x) + 24 \right)
$$

Clearly there's a pattern here.
The denominator in the $k$-th order derivative is $j^{k + 1}$.
The term $j^k \log^k(x)$ gets introduced in the $k$-th order, and for order $\ell > k$ becomes scaled by each successive $k$.
The sign in front of this term alternate, starting positive, and thus matching the order.
Generalizing, we get,[^4]

$$
T_j^{(k)}(s) = \frac{x^j}{j^{k + 1}} \left( \sum_{\ell = 0}^{k} (-1)^{k - \ell} \frac{k!}{\ell!} j^\ell \log^\ell(x) \right)
$$

Despite this being ugly, we're on the home stretch now.
We have

$$
\begin{aligned}
\mathrm{Pr}(j) &= \frac{1}{(j - 1)!}T_j^{(j - 1)} \, \Big|_{s = 0}\\
&= \frac{1}{(j - 1)!} \frac{x^j}{j^j} \left( \sum_{\ell = 0}^{j - 1} (-1)^{j - \ell - 1} \frac{(j - 1)!}{\ell!} j^\ell \log^\ell(x) \right)   \Bigg|_{s = 0}\\
&= \frac{x^j}{j^j} \left( \sum_{\ell = 0}^{j - 1} \frac{(-1)^{j - \ell - 1}}{\ell!} j^\ell \log^\ell(x) \right)   \Bigg|_{s = 0}\\
\end{aligned}
$$

Finally, removing $x$ and putting back our mixture,

$$
\begin{aligned}
\mathrm{Pr}(j) &= \frac{\left[ \sum_i e^{-R_i(1 - s)} p_i R_i \right)^j}{j^j} \left( \sum_{\ell = 0}^{j - 1} \frac{(-1)^{j - \ell - 1}}{\ell!} j^\ell \log^\ell\left( \sum_i e^{-R_i(1 - s)} p_i R_i \right) \right] \,  \Bigg|_{s = 0}\\
&= \frac{\left( \sum_i e^{-R_i} p_i R_i \right)^j}{j^j} \left[ \sum_{\ell = 0}^{j - 1} \frac{(-1)^{j - \ell - 1}}{\ell!} j^\ell \log^\ell\left( \sum_i e^{-R_i} p_i R_i \right) \right]
\end{aligned}
$$

This is a horrific PMF.
In good news, for a fixed mixture parameter set $\mathbf{R}, \mathbf{p}$, we can pre-compute the mixture term.
Other than that, we hope we never have to evaluate it for particularly large $j$.


[^1]: Judging by some of the literature that's come out since, probably due to [Blumberg and Lloyd-Smith (2013)](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1002993) crossing the field barrier.
[^2]: A homogenous continuous-time birth-death process, like we use in phylogenetics, is also a negative binomial branching process. Infections last an Exponential duration, given which they infect a Poisson number of additional people. But you get negative binomials marginalizing over _Gamma_ distributions, so we're in a pretty small corner of parameter space being stuck with exponentials.
[^3]: See [the supplement](https://static-content.springer.com/esm/art%3A10.1038%2Fnature04153/MediaObjects/41586_2005_BFnature04153_MOESM1_ESM.pdf), S2.4.1, of Lloyd-Smith for the integral representation of the PGF, and the methods subsection "Size distribution of stuttering chains" for the rest.
[^4]: Claude gets this as well. Perhaps reassuringly, it has chosen a slightly sillier representation with a binomial coefficient it has to cancel, rather than the two factorials.



