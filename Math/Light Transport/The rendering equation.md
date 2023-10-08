# Full formulation
$$ L_o(x, \omega_o) = L_e(x, \omega_o) + \int_\Omega f_r(x, \omega_i, \omega_o) L_i(x, \omega_i) (\omega_i \cdot n) d\omega_i$$
- $x$ - position in space
- $\omega_o$ - outgoing direction of light (towards the eye)
- $\omega_i$ - incoming direction of light (towards the light)
- $n$ - surface normal vector at $x$
- $\Omega$ - hemisphere of unit directions oriented around $n$
- $L_o(x, \omega_o)$ - light leaving $x$ in direction $\omega_o$
- $L_e(x, \omega_o)$ - light _emitted_ at $x$ in direction $\omega_o$
- $f_r(x, \omega_i, \omega_o)$ - proportion of light reflected from $\omega_i$ to $\omega_o$ (BRDF)
- $L_i(x, \omega_i)$ - light incoming at $x$ from direction $\omega_i$
- $(\omega_i \cdot n)$ - Cosine term (N dot L)

# Simplified formulation
$$ L_o = L_e + \int_\Omega f_r \ L_i \ cos (\theta) \ d\omega$$
- $L_o$ - Outgoing light from the given point
- $L_e$ - _Emitted_ light from the given point
- $\Omega$ - The hemisphere of directions oriented around the normal
- $f_r$ - Ratio of incoming and outgoing light (BRDF)
- $L_i$ - Light incoming to the given point
- $cos(\theta)$ - Cosine term (N dot L) - more on this later

# Why do we need monte carlo?
Can we calculate the rendering equation analytically? No, because the incoming light from a given direction $L_i$ depends on the scene we are rendering! There exists no general solution to the rendering equation, so what can we do - we use [[Monte carlo methods]]!

#light-transport #global-illumination 