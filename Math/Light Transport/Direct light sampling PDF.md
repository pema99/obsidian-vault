---

---
When we estimate the rendering equation by monte carlo integration, we typically integrate over the solid angle domain, but with direct light sampling, we use the (surface) area domain of the light sources, and need to convert between the 2.

An integral for direct lighting can be written as so:
$$\int_{\Omega_d} f_r(x, w_i, w_o) L_i(x, w_i) w_i \cdot w_n dw$$
Where $\Omega_d$ crucially denotes only the part of the hemisphere - the part which the visible lights project onto. This is similar to the rendering equation, except we know that the incoming ray is coming from a light source.

We can instead integrate over area domain like this:
$$\int_\triangle f_r(x, w_i, w_o) L_i(x, w_i) w_i \cdot w_n \frac{-w_i \cdot n}{r^2} dA$$
Where $r$ is the distance to the light source, $n$ is the surface normal of the light source, and $\triangle$ denotes the domain of light source surface area. $dA$ is thus projected differential area, centered around the point on the light source which we hit.
This uses the fact that:
$$dA = r^2 \frac{1}{-w \cdot n} dw = \frac{r^2}{-w \cdot n} dw$$
Which can be rewritten to:
$$dw = \frac{-w \cdot n}{r^2} dA$$
The intuition behind this is fairly straight forward. As we move the light source further from the shading point, by a distance of $r$, the projected area grows by a factor of $r^2$. This is the inverse square law, and the reason for the $r^2$ term.
As we tilt the light source away from the shading point, the projected area also grows. The factor with which it grows is determined by the cosine of the angle between the surface normal on the light source and the negated incoming light direction. As the angle grows, the cosine shrinks. Since the area should grow and not shrink when this happens, we have the $\frac{1}{-w_i \cdot n}$ term. Since both 
vectors are normalized, that dot product is exactly the cosine of the angle. See https://arxiv.org/abs/1205.4447 for more details.

We can write out an estimator for our integral over area domain, assuming uniform sampling over the surface of the light source:
$$\frac{1}{n}\sum^n \frac{f_r(x, w_i, w_o) L_i(x, w_i) w_i \cdot w_n \frac{-w_i \cdot n}{r^2}}{\frac{1}{|\triangle|}}$$
Where $|\triangle|$ is the surface area of the light source.
We can further simplify to:
$$\frac{1}{n}\sum^n f_r(x, w_i, w_o) L_i(x, w_i) w_i \cdot w_n \frac{(-w_i \cdot n) |\triangle|}{r^2}$$
Note now that this last part:
$$\frac{(-w_i \cdot n) |\triangle|}{r^2}$$
Is precisely the reciprocal of the formula you see written out in code below. The reason I use the reciprocal is because I am dividing rather than multiplying this weighting term. It's an implementation detail.

Note what this tell us about direct light sampling - to evaluate direct lighting, for a given bounce, we choose a direction towards a light source, and multiply the incoming radiance (light source emission) by the BRDF, by the regular cosine term (often folded into the BRDF), and by the weighting factor described just above - there are 2 cosine at terms at play!

This explanation doesn't include visibility checks or weight for multiple different sources, though, so let me briefly describe. When we don't pass a visibility check (ie. the chosen light point is occluded), we simply don't add the contribution, since the probability of hitting that point is 0. When we have multiple light sources, we simply pick one at random and divide the contribution by the probability of picking the given light source. This is just splitting the estimator into multiple addends.

Code:
```rust
let cos_theta = light_normal.dot(-light_direction);
if cos_theta <= 0.0 {
        return 0.0;
}
return light_distance.powi(2) / (light_area * cos_theta);
```

#path-tracing #monte-carlo #math #light-transport #global-illumination #probability 