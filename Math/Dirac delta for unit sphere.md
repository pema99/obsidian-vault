Relevant [stackoverflow question](https://math.stackexchange.com/questions/1231609/delta-function-in-spherical-coordinates-does-my-professor-have-a-mistake)

Dirac delta is a function given by:
$$
\delta(x) =
\Bigg\{
\begin{array}{lr}
\infty & \text{if } i = 0\\
0, & \text{if } i \neq 0
\end{array}
$$
We typically use it to pick specific values out of an integral:
$$
\int_{-\infty}^\infty f(x)\delta(x-a) dx = f(a)
$$
We sometimes use shorthand and write something like this:
$$
\int_{-\infty}^\infty f(x)\delta_a(x) dx = f(a)
$$
Where $\delta_a$ is "the delta function which is 0 everywhere but $a$".

Dirac delta function unit sphere:
$$  \delta_{us}(\omega) = \delta_{us}(\theta, \phi) = \frac{\delta(\theta-a) \delta(\theta-b)}{sin(a)} $$
Where $(a, b)$ are the desired $(\theta, \phi)$ where the function should be $\infty$. The reason for the extra $sin(a)$ term is because of conversion to spherical coordinates.

The Dirac delta must integrate to 1 over the domain. Let's try the naïve approach without the sine (see [[Spherical integrals]] for why the $sin(\theta)$ term is there):
$$
\int_0^{2\pi} \int_0^\pi \delta(\theta-a)\delta(\phi-b)sin(\theta) d\theta  d\phi = sin(a)
$$
Now we add the sine term, which cancels out the other one:
$$
\int_0^{2\pi} \int_0^\pi \frac{\delta(\theta-a)\delta(\phi-b)}{sin(a)}sin(\theta) d\theta  d\phi = 1
$$

#math 