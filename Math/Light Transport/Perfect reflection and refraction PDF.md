For the case of a perfectly reflective and refractive glass material, the BSDF is a weighted sum of 2 delta distributions, where the weight is $Fresnel(w_i)$ for reflection and $1-Fresnel(w_i)$ for refraction.

In a path tracer you typically want to only continue one path when you sample a BSDF consisting of multiple parts. You can sample such a BSDF by randomly choosing between reflection and refraction, effectively splitting the integral into 2 addends, and dividing the samples contribution by the probability of picking it to eliminate the added bias when doing monte carlo.

I was confused why I my implementation which did not counteract the bias was giving seemingly unbiased results, but it turns out I was sort of accidentally importance sampling, and some terms cancel out. 

For both reflection and refraction your estimator essentially looks like $\frac{F(\omega_i) * L(\omega_i)}{p(\omega_i)}$ where $L$ is incoming radiance, $F$ is fresnel or (1-fresnel) depending on reflection or refraction, and $p(w_i)$ is the probability of picking the sampled direction. 

The way I was picking between reflection and refraction was by generating a random number `[0;1]` and picking reflection if it was lower than the fresnel value. This makes $p(w_i)=F(w_i)$, so the estimator simplifies to $\frac{F(\omega_i) * L(\omega_i)}{p(\omega_i)} = \frac{F(\omega_i) * L(\omega_i)}{F(\omega_i)} = L(\omega_i)$ which explains why not doing any pdf divison was unbiased.

See also [[Ratio for diffuse and specular bounce]]

#math #path-tracing #global-illumination #light-transport #probability 