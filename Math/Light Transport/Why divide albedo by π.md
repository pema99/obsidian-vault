Let's try calculating total reflected outgoing energy without it:
$$ \int_\Omega f_r \ L_i \ cos(\theta) d\omega = \int_\Omega albedo \ L_i \ cos(\theta) d\omega $$
We can pull out $albedo$ and $L_i$ as they are both constant:
$$ \int_\Omega albedo \ L_i \ cos(\theta) d\omega = albedo \ L_i \int_\Omega \ cos(\theta) d\omega $$
Now, note that (See [[Spherical integrals]]):
$$ \int_\Omega  cos(\theta) d\omega = \int_0^{2\pi} \int_0^{\frac{\pi}{2}} cos(\theta) sin(\theta) d\theta d\phi = \int_0^{2\pi} \frac{1}{2} d\phi = \pi $$
So we get:
$$
albedo \ L_i \int_\Omega \ cos(\theta) d\omega = albedo \ L_i \ \pi
$$
Oh no! This does not satisfy energy conservation:
$$
\int_\Omega f_r \ L_i \ cos(\theta) d\omega \leq L_i
$$
If we divide albedo by $\pi$:
$$ \int_\Omega \frac{albedo}{\pi} \ L_i \ cos(\theta) d\omega = albedo \ L_i $$
We reflect exactly incoming incoming energy attenuated by albedo, and energy conservation is satisfied.

See also https://seblagarde.wordpress.com/2012/01/08/pi-or-not-to-pi-in-game-lighting-equation/

#math #light-transport #pbr