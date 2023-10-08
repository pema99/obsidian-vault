Solid angles are mathematical quantity that shows up fairly commonly in light transport, for example in the typical formulation of [[The rendering equation]]. They are a measure of how much field of view is covered by an object from the point of view of a static observer.

Solid angles are typically measured in a unit called _steradians_. One steradian is equal to one unit of area on a unit sphere (sphere with radius=1). Thus, an object that fully covers the observers view from all direction would be represented with a solid angle of $4\pi$ steradians, as the surface area of the unit sphere is $4\pi$.

The formula for calculating solid angles is:
$$
\Omega = \frac{A}{r^2}
$$
$A$ is portion of surface area on a sphere centered at the observer, which is covered by a considered object. $r$ is the radius of aforementioned sphere. Note how the definition of solid angle naturally accounts for the [inverse square law](https://en.wikipedia.org/wiki/Inverse-square_law). As the distance to the observer ($r$) grows linearly, the perceived portion of the observers field of view being covered decreases by a factor of the distance squared ($r^2$). 

> Note: This isn't very important, but interesting nonetheless: Since both the numerator and denominator in the formula for solid angle have the same unit, length squared, solid angles are what you call a "dimensionless" unit, and steradians are a dimensionless quantity. Concretely, the definition steradian, $\frac{m^2}{m^2}$ cancels out simplifies to $1$.

Solid angles are typically illustrated as a cone section of a sphere:
![[Pasted image 20231008194242.png]]
However, it is important to note that solid angles have no inherent "shape", as they are only a measure of a _portion_ of a sphere, which is inherently an amorphous quantity.

As mentioned in the notes on [[Spherical integrals]], the relationship between differential solid angle and differential spherical coordinates is:
$$
d\omega = \sin(\theta) d\theta d\phi
$$
Which is useful for converting between spherical integrals of solid angle and spherical coordinate domain. Differential solid angles can be visualized as the section of a unit sphere subtended by a surface patch of the unit sphere with infinitesimal area:
![[SphericalIntegration.png]]
This quantity is typically associated with rays of lights. For example, if we integrate over a sphere, written $S^2$, in solid angle domain:
$$
\int_{S^2} f(\omega) d\omega
$$
We can imagine dividing the sphere into infinitely many infinitesimal surface patches, each associated with a ray. Therefore, when reading integrals in solid angle domain, such as the one above, it can be helpful to think of the variable being integrated over ($\omega$) as meaning "a ray direction", and the differential solid angle ($d\omega$) as meaning "the infinitely small portion of the the total sphere which any given ray accounts for".

It should come as no surprise that numerical computation of spherical integrals like the one above are often done by shooting a bunch of rays in random directions from the an observers point of view, and "averaging" their contribution using [[Monte carlo methods]]. 

See [[Spherical integrals]] for more info on integrating over the sphere and hemisphere.

#math #light-transport #physics 