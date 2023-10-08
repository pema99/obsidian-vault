For basic info on how to calculate integrals, see [[Integration primer]]. Just as we can integrate over a number line, we should be able to integrate over the surface of a unit sphere. To do so, we can use spherical coordinates.

![[SphericalCoordinates.png]]

A naive approach would be a double integral like so:
$$
\int_0^{2\pi} \int_0^\pi f(x) \ d\theta \ d\phi
$$
We know that the surface area of a sphere is $4\pi r^2$, so for a unit sphere, $4\pi1^2 = 4\pi$. # 

Let's try integrating the constant $1$ over the unit sphere using this naive approach. This should give us the surface area of the sphere.

$$
\int_0^{2\pi} \int_0^\pi 1 \ d\theta \ d\phi = \int_0^{2\pi} \pi \ d\phi = 2 \pi^2
$$

That is definitely not $4\pi$! It turns out we are missing a correction factor for the curvature of a sphere. When we integrate over a sphere, our differential is an infinitely small quadrilateral surface patch on the sphere surface.

![[SphericalIntegration.png]]

Notice how the top and bottom edge of the differential surface patch are different lengths! We need to account for the shape of the differential, otherwise we are just integrating over a perfectly rectangular plane! The height of the quadrilateral is $d\theta$, but the width is $sin(\theta)d\phi$. Notice how the sine term makes the width small when theta is close to one of the poles.

We can revisit our naive method, and add the correction factor of $sin(\theta)$:

$$
\int_0^{2\pi} \int_0^\pi 1 \sin(\theta) \ d\theta \ d\phi = \int_0^{2\pi} 2 \ d\phi = 4 \pi
$$

And now we get the correct surface area of the unit sphere, $4\pi$, so this method of spherical integration works.

We can turn the spherical integral into a hemispherical one by changing range of integration for $\theta$ to $[0; \frac{\pi}{2}]$:
$$
\int_0^{2\pi} \int_0^\frac{\pi}{2} 1 \sin(\theta) \ d\theta \ d\phi = \int_0^{2\pi} 1 \ d\phi = 2 \pi
$$
[[The rendering equation]] contains a hemispherical integral: 
$$ L_o = L_e + \int_\Omega f_r \ L_i \ cos (\theta) \ d\omega$$
Where $\Omega$ denotes a unit hemisphere of directions. Why does this look different from what we've seen, and what exactly is $d\omega$?

The formulation of the rendering equation shown earlier looks different because it is integrating over _[[Solid Angle]] domain_. The relationship between differential solid angle and differential spherical coordinates is:
$$
d\omega = \sin(\theta) d\theta d\phi
$$
Thus:
$$
\int_\Omega f(\omega ) d\omega = \int_0^{2\pi} \int_0^{\frac{\pi}{2}} f(\theta, \phi) sin(\theta) d\theta d\phi 
$$
A final note on spherical integrals: Why use this alternate integration domain and not just spherical coordinates? 

Essentially, this is just because it makes the math much nicer. There is nothing stopping us from writing the rendering equation out with a parameterization using spherical coordinates. You can think of $\omega$ as roughly representing a ray direction. See [[Solid angle]] for more info on solid angles.

#light-transport #math 
