---
layout: post
title: On mipmap selection
---
In this post, I want to shed some light over something I've been wondering about for quite a while: How exactly are mipmap/LOD levels selected when sampling textures on the GPU? If you already know what mipmapping is, why we use it, and what pixel derivatives (`ddx()` / `ddy()`) are, you can skip to the section [[#Derivatives to mipmap levels]]. The post does however assume some knowledge of graphics programming.
# Mipmapping primer
A very common operation when writing [shaders](https://en.wikipedia.org/wiki/Shader) is texture sampling. In [HLSL](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl), we typically use the `Texture2D.Sample()` function for this. For a regular 4-channel floating point texture, the full function signature looks like this:
```glsl
float4 Texture2D.Sample(SamplerState sampler, float2 location);
```
It takes as input a location to sample the texture at, and a `SamplerState`, which is an object containing metadata used to influence the sampling, like which [texture filtering](https://en.wikipedia.org/wiki/Texture_filtering) mode should be used.

Here is an example of a simple fragment/pixel shader which shades each pixel using the result of sampling a texture `_Texture` using the UV coordinates in the 0th channel:
```glsl
float4 frag (float2 uv : TEXCOORD0) : SV_Target  
{  
    return _Texture.Sample(sampler_Texture, uv);  
}
```

When applied to a quad and fed a brick texture as input, we get this:
![[Unity_HTfdhuIUKO.png]]

Notice that the image looks rather 'grainy', especially near the far edge. This is texture [aliasing](https://en.wikipedia.org/wiki/Aliasing). The more shallow the viewing angle, the further apart the [texels](https://en.wikipedia.org/wiki/Texel_(graphics)) sampled in the texture for neighboring screen pixels can be.
<!--![[Pasted image 20250508002744.png]]-->

The typical solution to this is [mipmapping](https://en.wikipedia.org/wiki/Mipmap) - we first generate a bunch of smaller versions of our texture, called mipmaps, by averaging texels from the higher resolution texture. As the mipmaps get smaller, their contents get more blurry. We are effectively applying a series of [low-pass filters](https://en.wikipedia.org/wiki/Low-pass_filter). We typically name the mipmaps using numbers, where 0 is the full resolution texture, and each subsequent number corresponds to a lower resolution texture. I'll call these numbers 'mipmap levels'.
![[ezgif-31017683bfd76e.gif]]
In cases where texture sampling would likely result in aliasing, such as at when the texture is viewed at shallow angles, we sampling a lower resolution mipmap to mitigate it. Here's the same setup before, but now with mipmapping enabled:
![[AVNlMTV1Uq.png]]
We didn't actually have to change the shader at all to achieve this - when `Texture2D.Sample` is used, and the input texture contains mipmaps, the GPU will automatically select an appropriate mipmap level for each sample!

If you are like me, you won't find this explanation very satisfying. The GPU does _some magic_ to select a mipmap level? Ok - so how does it actually work? That's what the rest of this post is about.

# Pixel derivatives
A common mental model for a fragment shader is "the piece of code inside of a big parallel for-loop iterating over every pixel on the screen". The fragment shader is invoked for every pixel in parallel, each invocation gets a set of per-pixel inputs, and uses them to calculate the final color at that pixel. This is a fine mental model, but it doesn't tell the full story.

Fragment shaders aren't actually invoked every for pixel in isolation - rather, they are invoked for 2x2 blocks of pixels, typically called "pixel quads", or just "quads". This setup allows us to approximate derivatives in screen space for any value using the [finite difference](https://en.wikipedia.org/wiki/Finite_difference) method! In HLSL, the functions to do this are called `ddx()` and `ddy()`, and return partial derivatives along the X and Y axis in screen space respectively. You couldn't actually implement these functions yourself - they are magic intrinsic which are implemented in hardware by subtracting the values associated with difference pixels in the quad.

Now, observe what happens if we apply a small fragment shader that visualizes the partial derivatives along the Y-axis of the UV coordinates we were using to sample the texture from earlier:
```glsl
float4 frag (float2 uv : TEXCOORD0) : SV_Target  
{  
    // Multiply by 15 so we can actually see the derivatives,  
    // they are typically very small.    return float4(ddy(uv), 0, 1) * 15;
}
```

The more shallow the viewing angle, the larger the derivatives get. Additionally, the derivatives are larger further away from the camera. ![[nZeABptci6.gif]]

These conditions line up well with the cases where texture sampling would result in ugly aliasing, so it shouldn't come us a big surprise that selection of mipmap levels has something to do with the derivatives of the UV coordinates. In fact, `Texture2D.Sample` can be seen as syntax sugar for the more general `Texture2D.SampleGrad` function:
```glsl
// These 2 lines of code are equivalent:
Texture2D.Sample(sampler, location);
Texture2D.SampleGrad(sampler, location, ddx(location), ddy(location));
```
`Texture2D.SampleGrad()` takes the derivatives of the sampling location explicitly, whereas `Texture2D.Sample()` implicitly calculates them using `ddx()` and `ddy()`. Passing these explicitly can be useful in cases where the sampling locations are warped in a way that prevents `ddx()` and `ddy()` from returning useful derivatives (for example, during [texture raymarching](https://iquilezles.org/articles/filteringrm/) or parallax mapping). It also lets us use analytical derivatives in cases where we have a way to compute them.

This description still leaves an open question: How exactly does `Texture2D.SampleGrad()` use the input derivatives to determine which mipmap level to sample?

# Derivatives to mipmap levels
`Texture2D.SampleGrad()` clearly has some internal logic that maps from the derivatives of the sampling location, to a specific mipmap level. This function directly corresponds to a hardware instruction on the GPU, though, so we unfortunately can't just 'look at the code'. To get a better idea of how it works, we can ask 2 questions: What does the mapping look like conceptually, and what does the hardware do in practice? Both questions are surprisingly difficult to find answers to online, but I'll tackle them both one by one.
### What does the mapping look like conceptually?
One place you might think to look for information on how the mapping should work is in the spec for your graphics library of choice. I find the [GLES3.0](https://registry.khronos.org/OpenGL/specs/es/3.0/es_spec_3.0.pdf) spec particularly readable. According to section 3.8.10.1 "Scale Factor and Level of Detail", the mipmap level is calculate like so:
$$
MipLevel(x, y) = log_2(\rho(x, y))
$$
Where $x, y$ are the coordinates of the pixel, and $\rho$ is a "scale factor" defined by:
$$
\rho = max\Bigg\{\sqrt{\bigg(\frac{\partial u}{\partial x}\bigg)^2 + \bigg(\frac{\partial v}{\partial x}\bigg)^2},\sqrt{\bigg(\frac{\partial u}{\partial y}\bigg)^2 + \bigg(\frac{\partial v}{\partial y}\bigg)^2}\Bigg\}
$$
Here, $u$ and $v$ denote the coordinates of the sampling location, so $\frac{\partial u}{\partial x}$ for example is the partial derivative of the horizontal coordinate of the sampling location with respect to the $x$, i.e. `ddx(u)`.

That's a bit of a mouthful, so lets translate it to code:
```glsl
float MipLevel(float u, float v)
{
	float2 du_dx = ddx(u);
	float2 dv_dx = ddx(v);
	float2 du_dy = ddy(u);
	float2 dv_dy = ddy(v);

	float lengthX = sqrt(du_dx*du_dx + dv_dx*dv_dx);
	float lengthY = sqrt(du_dy*du_dy + dv_dy*dv_dy);
	float rho = max(lengthX, lengthY);

	return log2(rho);
}
```
This should be _somewhat_ intuitive - as the length of the derivatives increase, so should the mipmap level. We only really care about the worst case - the axis with largest magnitude - hence the `max()`. The `log2()` accounts for the fact that each increment in mipmap level corresponds to sampling a texture that is half the size.

If you search around the web for how to calculate mipmap level manually, you would probably find a solution like this. However, as we will soon see, this is not the full story.
## What does the hardware actually do?
As I mentioned earlier, since `Texture2D.SampleGrad` is implemented in hardware, we can't look at the implementation directly. However, we _can_ use observations of its behavior to deduct what it is doing. To facilitate this, I will first need a texture with mipmaps that are clearly distinguishable from each other. Luckily, most graphics frameworks let you manually fill in the texture data for each mipmap - they don't _have_ to be blurry versions of the full-resolution texture. Since I'm using Unity for visualization, I wrote [a little script](https://gist.github.com/pema99/c706b38eb94b13a1680c8635c180b228) that produces a texture where each mipmap is a single, bright, clearly distinguishable color. The resulting texture looks like this:
![[J9MzgxLxZf.gif]]

Next, I wrote a shader that samples the texture using `Texture2D.SampleGrad()`, but where the sampling location is some fixed point, and the derivatives are based on UV coordinates. `Texture2D.SampleGrad()` takes a partial derivative for both the X and Y axis, each of which are 2D vectors, so this is technically a 6-dimensional function. Since we are keeping the sampling location fixed, it is a 4-dimensional function in practice. Functions of over 3 dimensions are a bit tricky to visualize, so I'll focus on just the derivatives for the X axis for now, and leave the Y axis derivatives as 0:
```glsl
float4 frag (float2 uv : TEXCOORD0) : SV_Target  
{  
    // Scale UVs from [0; 1] to [-1; 1]
    float2 scaledUV = (uv - 0.5) * 2.0;

    // Sample texture at location (0.5, 0.5),
    // using X derivative = UV coords,
    // and Y derivative = 0.
	float2 samplingLocation = 0.5;
	float2 yDerivative = 0;
    float4 hardwareSample = _MainTex.SampleGrad(
        sampler_MainTex,
        samplingLocation,
        scaledUV.xy,
        yDerivative);  
  
    return hardwareSample;
}
```
Applying this shader to a quad yields on my machine (GPU is a RTX 4070 super):
![[Pasted image 20250508020347.png]]
The image is scaled such that bottom left corner corresponds to X axis derivatives of (-1, -1), and the top left corner corresponds to (1, 1). The color at each pixel indicates which mip level is being sampled.

When I first saw this result, I was a bit surprised. Why are there jagged edges!? The formulas described in the previous section should result in a bunch of perfect concentric circles! As it turns out, current graphics libraries leave the specifics of the implementation up to the GPU vendor, and only prescribe some loose criteria on the implementation. Nvidia cards use a particularly crude approximation.

> Sidenote: This is just one of many, many pieces of functionality that differ in implementation across different vendors. It's a pet peeve of mine when people assume that there is a "one true correct" implementation of pretty much anything in GPU-land. To name a few other areas where vendors differ: The precision of transcendentals like `cos(x)` and `sin(x)`, the implementation of AlphaToCoverage (AMD uses dithering), subtle differences in rasterizer output, especially with conservative rasterization, etc. This is unfortunately a big source of pain when doing image-based/golden-image testing for anything involving the graphics pipeline, and is one of the big reasons such test setups often have per-platform reference images, and use fuzzy comparisons.

My next instinct was to determine which vendors *actually* differ in their implementation, and to what extent. I asked a bunch of friends to try it on their hardware, and collected the results:

NVidia 30xx series, 40xx series, 50xx series:
![[Pasted image 20250508021935.png]]

AMD 7900XTX, AMD RX580, AMD 9000 series APU:
![[Pasted image 20250508021838.png]]

Unknown MacBook:
![[Pasted image 20250508022033.png]]

Intel UHD Graphics 620 iGPU:
![[Pasted image 20250508021705.png]]

Adreno 740 (Oculus Quest 3):
![[Pasted image 20250508021750.png]]

A few notes on these results:
- All tested vendors seem to *roughly* agree on the mapping function. Every image resembles a set of concentric circles with the same radius. The extent to which the circle is approximated differs quite a bit across every vendor, though.
- No 2 vendors agree _entirely_ on the implementation.
- All tested vendors seem to be consistent across their hardware lineup. Interestingly AMD's APUs are consistent with their dedicated GPUs.

And with this, I present perhaps the first-ever GPU tier list ranked in order of fidelity of derivative-to mipmap-level-mapping-function (the only important metric, clearly): Adreno/Qualcomm > AMD > Intel > MacBook/Apple > Nvidia. That's right Nvidia - you suck! (/s)
# Approximating the vendor implementations
## Exploring the full 4D mapping function
## Missing elliptical transformation
## Bilinear and trilinear filtering
## Anisotropic filtering
