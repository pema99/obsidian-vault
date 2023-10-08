In a typical path tracer with a PBR shading model, you'll want to distinguish between diffuse and specular bounces. This not only allows combining different models for diffuse and specular reflection, but also lets you sample one kind of bounce differently from the other. Typically we want to importance sample specular reflection, otherwise it will take forever to converge.

A typical path tracer will not branch paths, however. Every ray will follow exactly 1 path from start to finish. This means you need to make the choice of whether to sample diffuse or specular reflection at each bounce. Since we are doing monte carlo integration, the choice must be random.

When I was writing my last path tracer, I first used the simple ratio `0.5 + 0.5 * metallic` for choosing specular and diffuse. This works ok-ish (any ratio is fine so long as you account for it in the PDF for the sample), but isn't great and takes long time to converge.

The ratio between reflected and refracted light is governed by the fresnel equations. In the context of a dielectric PBR material, refracted light is roughly equivalent to diffuse reflection. Explained in pseudocode:
```pseudocode
fr = fresnel(cos_theta, 1.0, 1.5); // assuming 1.5 IOR
spec_contrib = lerp(fr, 1.0, metallic);
diffuse_contrib = lerp(1.0-fr, 0.0, metallic); // neglect absorbption
```
The reason we just use `1.0` instead of fresnel for conductive (fully metallic) materials, is because metals do not diffuse light at all.

When I implemented this, it caused the image to converge faster, but introduces nasty fireflies:
![[Pasted image 20230917175503.png]]

The reason for this is that essentially all dielectrics (anything with IOR greater than 1.0) will have _some_ specular contribution. For example, with an air-dielectric interface, with the dielectric having metallic=0, roughness=1, and an IOR=1.5, around 4% of the photons will be reflected specularly. Since this number is so low, our rays won't sample specular paths very often, which leads to high frequency noise - in other words, fireflies.

Jakub Boksansky and Adam Marrs propose a fix for this in RT gems 2 chapter 14, which is stupid simple but works well in practice - simple clamp the specular ratio to `[0.1; 0.9]` when it isn't 1 or 0.
![[Pasted image 20230917180234.png]]

Code to implement this:
```rust
let approx_fresnel = util::fresnel_schlick_scalar(
	1.0, DIELECTRIC_IOR, normal.dot(view_direction).max(0.0));
	
let mut specular_weight = util::lerp(approx_fresnel, 1.0, self.metallic);

if specular_weight != 0.0 && specular_weight != 1.0 {
	specular_weight = specular_weight.clamp(
		self.specular_weight_clamp.x, self.specular_weight_clamp.y);
}
```
Remember to divide the contribution by `specular_weight` or `1-specular_weight` to remain unbiased!

https://seblagarde.wordpress.com/2013/04/29/memo-on-fresnel-equations/

#math #pbr #light-transport  #path-tracing #global-illumination 