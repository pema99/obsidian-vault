[[The rendering equation]] for lambertian BRDF is (see [[Spherical integrals]] for info on how to solve):
$$ \int_\Omega f_r \ L_i \ cos(\theta) d\omega = \int_\Omega \frac{albedo}{\pi} \ L_i \ cos(\theta) d\omega = \int_0^{2\pi} \int_0^{\frac{\pi}{2}} \frac{albedo}{\pi} \ L_i \ cos(\theta) sin(\theta) d\theta d\phi$$
Assume single directional light with direction (a, b) in spherical coordinates:
$$ = \int_0^{2\pi} \int_0^{\frac{\pi}{2}} \delta_{light}(\theta, \phi) \frac{albedo}{\pi} \ L_i \ cos(\theta) sin(\theta) d\theta d\phi $$
---
Where $\delta_{light}$ is a delta distribution with value $\infty$ in the direction of the light and $0$ everywhere else. This is written as (see [[Dirac delta for unit sphere]]):
$$ \delta_{light}(\theta, \phi) = \frac{\delta(\theta-a) \delta(\theta-b)}{sin(a)} $$
Now we can solve the integral: 
$$\int_0^{2\pi} \int_0^{\frac{\pi}{2}} \delta_{light}(\theta, \phi) \frac{albedo}{\pi} \ L_i \ cos(\theta) sin(\theta) d\theta d\phi = \frac{albedo}{\pi}cos(a)$$
Note now that $cos(a)$ is precisely "N dot L" as you typically know it, so this matches typical realtime shading.

Note, I used Mathematica to solve the last integral. Wolfram isn't good at delta distributions.
```mathematica
FullSimplify[ Integrate[ Integrate[(DiracDelta[t - a, p - b])/Sin[a] Sin [t], {t, 0, pi/2}], {p, 0, 2 pi}], Assumptions -> {0 < a < pi/2, 0 < b < 2 pi}] = 1
```

#math #light-transport #pbr 