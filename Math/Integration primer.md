For simple univariate functions, integration is equivalent to "finding the area under the function".

A discrete method is a Riemann sum: Split into $n$ rectangles with height given by the integrand, sum their area:
$$ \sum_{i=0}^n f(x_i) \Delta x_i $$
The limit of a Riemann sum is a definite integral:
$$ \int_a^b f(x) dx = \lim_{n\rightarrow\infty} \sum_{i=0}^n f(x_i) \Delta x_i $$
Where $x$ ranges from $a$ to $b$. We can imagine a definite integral as a Riemann sum of _infinitely_ thin rectangles.

This infinitely small quantity, $dx$, is called a differential.

Definite integrals typically look like this:
$$ \int_a^b f(x) dx $$
Fundamental theorem of calculus tells us that (assuming continuity):
$$ \int_a^b f(x) dx = F(b) - F(a) $$
Where $\frac{d}{dx} F(x) = F'(x) = f(x)$

Sometimes the domain of integration is written at the bottom, as we saw in the rendering equation:
$$
\int_D f(x) dx
$$
This notation lets us use more complex domains, such as "a hemisphere of directions"

#math #light-transport 