# On a generalized Bus Factor

## Introduction
Many have previously proposed to measure resiliency of projects or organizations via the "Bus Factor." 
Bus Factors take values in the positive integers $\{\mathbb{Z}^{+}\}$.

The Standard Bus Factor (SBF) is, loosely, the number of people who must be removed from their roles befall before a crucial system cannot be maintained (or similar, such as a project needing to be suspended).
The model of resiliency underlying the SBF is of non-exclusive or.
That is, the system will work if individual 1 is present _or_ individual 2 is present _or_... and so forth.
Thus, resiliency increases with increasing SBF.

A less typical Alternative Bus Factor (ABF) definition runs in the opposite direction, indexing the number of individuals of which the removal of any one is fatal.
The model of resiliency underlying the ABF is "and."
In this model, a system only works if individual 1 is present _and_ individual 2 is present _and_... and so forth.
In this case, increasingly large values indicate _decreasing_ resiliency, with more crucial links which can be taken out.

## New approaches

It is not necessary, either mathematically or conceptually, to retain both Bus Factors.
Under the ABF model, failure occurs if one out of any number of other individuals is "bussed." 
This suggests that we might instead measure and-based Bus Factors in terms of fractions.
Thus, $\frac{1}{3}$ would indicate that if one out of any three key people were bussed, failure would occur, and as the fraction decreases, so does stability.

Thus we propose replacing the SBF and ABF with the Unified Bus Factor (UBF), which takes values in $\{\frac{1}{\mathbb{Z}^{+}}, \mathbb{Z}^{+}\}$.
The UBF increases with increasing stability, and encompasses both the SBF and ABF.
A UBF of 2 indicates that _either_ of two people are sufficient for resiliency, while a value of 1/2 indicates that _both_ are required.
As the SBF is undefined for values less than 1, we further suggest that the UBF can in general be safely used implicitly, requiring only (ill-advised) usage of the ABF to be qualified.

## Discussion and Conclusions
The SBF and ABF both presuppose at most one type of individual, the "or" type and the "and" type.
This is not necessarily the case for all systems of interest.
For example, a baseball team requires individuals capable of filling a number of different roles: catcher, pitcher, first base, and so forth.
Without one for every role, the team cannot play, but a team has more than one pitcher available, and thus can play even without one.
This "and"-based unification of multiple "or" types, we leave for future work.

Despite its limitations, the SBF has proven a useful tool for communicating about system resiliency.
We therefore hope that the UBF will provide a helpful extension for enumerating least-stable systems in clear and compact forms.