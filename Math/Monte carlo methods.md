Let's say we want to calculate a simple integral of the form:
$$
\int_a^b f(x) dx
$$
But we are unable do anything else with $f(x)$ than evaluate it, so we can't get the antiderivative. If we can't solve it analytically, maybe we can do it numerically?

We could use a riemann sum, but this won't work well for high dimensional integrals, such as the rendering equation, due to the _curse of dimensionality_ - there is just too much space to cover in the integration domain.

Instead, we can use _monte carlo methods_, which are stochastic methods relying on random sampling of the domain.
# Monte carlo integration
The basic idea of monte carlo methods is to sample the domain randomly many times, and average the results.

For our purposes, we are interested in a specific kind of monte carlo method typically called monte carlo integration, which is used for estimating definite integrals like ours.
$$
\int_a^b f(x) dx
$$
In probability theory, we have a theorem called the Law Of The Unconcious Statistician (LOTUS):
$$
E[F(X)] = \int F(X)P_X(X)dX
$$
Note that we only need the probability density on $X$ to calculate the integral, not the probability density on $F(X)$. From this law, we can [derive the estimator for monte carlo integration](https://scratchapixel.com/lessons/mathematics-physics-for-computer-graphics/monte-carlo-methods-mathematical-foundations/expected-value-of-the-function-of-a-random-variable.html).
# Estimators
The cornerstone of monte carlo integration is the monte carlo _estimator_. For our simple placeholder integral, it can be written as so:
$$
\frac{1}{N}\sum_{i=0}^N \frac{f(X_i)}{p(X_i)}
$$
- $N$ denotes the amount of random samples we have taken
- $X$ is a random variable
- $p(x)$ is the _probability density function_ of $X$
- $f(x)$ is our integrand from earlier.
# TODO
- [ ] Intuition for division of pdf
- [ ] Inverse transform method
- [ ] Importance sampling
- [ ] Derive estimator for monte carlo integration

#math #monte-carlo #probability