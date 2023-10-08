For lambertian surfaces, one way to sample them in path tracing is to simply use a uniform distribution of unit directions oriented around the surface normal at the shading point. However, we can do better than this.
# Why
The uniform distribution doesn't account for the fact that the projected area of a beam of light grows as the angle of incidence becomes shallower. 
![[Pasted image 20230917182004.png]]
![[Pasted image 20230917182012.png]]
This effect is described by the cosine term in [[The rendering equation]]. Looking at this, we can intuit that shallow angles are in some sense "less important" than narrow ones, because they contribute more to the final value of the integral. Thus, this is a perfect place to apply importance sampling.

Said in more a more boring way: Path tracing is an application of monte carlo integration (see [[Monte carlo methods]]) to estimate the rendering equation. Since the integrand of the rendering equation contains a cosine term, we should importance sample based on that term - matching the "shape" of the integrand with the "shape" of the sampling distribution is a wise choice in general.
# How
To do this, we first need to generate samples from our sampling distribution, the cosine weighted distribution:
```rust
pub fn cosine_sample_hemisphere(r1: f32, r2: f32) -> Vec3 {
    let theta = r1.sqrt().acos();
    let phi = 2.0 * core::f32::consts::PI * r2;
    Vec3::new(
        theta.sin() * phi.cos(),
        theta.cos(),
        theta.sin() * phi.sin(),
    )
}
```
Remember to then transform the sample into the coordinate space oriented about the shading normal!

The distribution is shown below. Clearly more samples towards the center of the hemisphere.
![[Pasted image 20230917182832.png]]

Once we have our cosine distributed samples, we need to account for the change in sampling distribution by adjusting our PDF. We know that the PDF should be proportional to the cosine term of the rendering equation - that was the whole point. If the PDF is correct, it should integrate to 1 over the hemisphere. Let's check (see [[Spherical integrals]] for how to calculate this):
$$
\int_{\Omega}cos(\theta)d\omega = \int_0^{2\pi} \int_0^\frac{\pi}{2} cos(\theta) sin(\theta) d\theta d\phi = \pi
$$
That didn't quite work - we need to add a correction term of $\pi$:
$$
\int_{\Omega}\frac{cos(\theta)}{\pi}d\omega = \int_0^{2\pi} \int_0^\frac{\pi}{2} \frac{cos(\theta)}{\pi} sin(\theta) d\theta d\phi = 1
$$
So the PDF to use is $cos(\theta)/\pi$. Using this distribution causes a noticeable reduction in noise for lambertian surfaces:
![[Pasted image 20230917184315.png]]
# Collapsing the rendering equation
Something interesting happens to the estimator for the rendering equation when we use cosine weighted sampling. Let's write out the rendering equation for our setup. Recall that the lambertian BRDF is $\frac{albedo}{\pi}$
$$
L_o = \int_\Omega f_r \ L_i \ cos (\theta) \ d\omega = \int_\Omega \frac{albedo}{\pi} \ L_i \ cos (\theta) \ d\omega
$$
> Note: I'm omitting the emission term because it isn't relevant

Next, we write the estimator for monte carlo integration with cosine weighted sampling:
$$
L_o \approx \frac{1}{N} \sum_{i=1}^N \frac{\frac{albedo}{\pi}L_i \ cos(\theta_i)}{\frac{cos(\theta_i)}{\pi}}
$$
Now lets simplify, this a bit:
$$
L_o \approx
\frac{1}{N} \sum_{i=1}^N \frac{\frac{albedo}{\pi}L_i \ cos(\theta_i)}{\frac{cos(\theta_i)}{\pi}} =
\frac{albedo}{\pi N} \sum_{i=1}^N \frac{L_i \ cos(\theta_i)}{\frac{cos(\theta_i)}{\pi}}
$$
$$
=
\frac{albedo}{\pi N} \sum_{i=1}^N \frac{L_i \cos(\theta_i) \pi}{cos(\theta_i)}
= \frac{albedo}{\pi N} \sum_{i=1}^N L_i \pi = \frac{albedo}{N} \sum_{i=1}^N L_i
$$
So the whole rendering equation estimator boils down to:
$$
\frac{albedo}{N} \sum_{i=1}^N L_i
$$
Which means we can simply sum over incoming radiance without having to take PDFs into account at all - it happens implicitly, so we don't actually need the PDF for cosine weighted sampling in practice.
# A secret trick
A secret trick graphics developers don't want you to know: If you want to generate cosine distributed directions around a given normal, you can do something much cheaper than the code shown earlier:
```
sample_direction = normalize(surface_normal + random_unit_direction);
```
Don't ask me why this works - I got no clue.

https://ciechanow.ski/lights-and-shadows/

https://www.rorydriscoll.com/2009/01/07/better-sampling/

https://ameye.dev/notes/sampling-the-hemisphere/

#math #light-transport #path-tracing #monte-carlo #probability #global-illumination #pbr 