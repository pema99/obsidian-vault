Spherical Harmonics pop up in certain fields of graphics, as they are useful for encoding spherical functions. In this document I describe various notes and tidbits about them.
# Vector spaces and basis vectors
> Note: Skip to [[#Fourier basis]] section if you already know what a basis function is.

A _vector space_ is (roughly) a set whose elements can be added together or multiplied by scalars (ie. scaled). These 2 operations must satisfy some sensible axioms, such as a associativity and commutativity of addition. Here is a full list of axioms:
![[Pasted image 20231005215702.png]]
Some common vector spaces you may be familiar with are $\mathbb{R}^2$ and $\mathbb{R}^3$ ie. the space of _real-valued_ 2D and 3D vectors, or what you may just know as colloquially as "2D and 3D vectors". Another example of a vector space is $\mathbb{C}^2$, the space of complex-valued 2D vectors.

A _basis_ for a given vector space, is a set vectors with the special property that any element of the vector space can be written in a _unique_ way, as a linear combination of elements of the basis. Here are some examples of bases for $\mathbb{R}^2$:
- $\{(1, 0), (0, 1)\}$ aka. the "standard basis"
- $\{(1,0), (0,0.5)\}$
-  $\{(1,0), (0.6,0.6)\}$

Note that something like $\{(1,0), (0.5, 0)\}$ is _not_ a valid basis, because the 2 vectors are parallel, meaning that not every element of the vector space can be written as a linear combination of the 2.

We call a basis "orthogonal" if all of the vectors in the basis are mutually orthogonal. We additionally call the basis "orthonormal" if all of the vectors are normalized. Looking at the 3 examples listed above, the first is orthonormal, the second is only orthogonal, and the third is neither orthogonal nor orthonormal.
## Basis functions
We typically think of elements of vector spaces, vectors, as lists of numbers. However, this is limiting a definition. In fact, if we loosen this definition, we can use _functions_ as our vectors, forming a _function space_. The addition and scaling operators are defined as such:
$$
\begin{gathered}
(f+g)(x) = f(x) + g(x)\\
(k \cdot f)(x) = k \cdot f(x)
\end{gathered}
$$
Where $k$ is a scalar, and $f$ and $g$ are functions.

The elements of a basis for a function space is then called a _basis function_. A simple example of a function space is a the space of single-indeterminate polynomials. One possible basis for this vector space is the _monomial basis_:
$$
\{x^n \ | \ n \in \mathbb{N}\} = \{x^0, x^1, x^2, ..., x^n\}
$$
This is clearly a valid basis as every polynomial can be written as a linear combination of the elements:
$$
c_0 + c_1 x^1 + c_2x^2 + ... +a_nx^n
$$
Where $c_n$ are constant coefficients for each basis function.
# # Fourier basis
Spherical Harmonics are an analogue of the Fourier basis on the surface of a sphere. To understand this, we need to go over a few more concepts.

The _Fourier series_ is a neat kind of infinite series which can be used to approximate _periodic_ functions (functions that repeat their values at regular intervals, called the period). Assume we have a periodic function $f(x)$ with a period of $2\pi$, defined on the interval $[0, 2\pi]$. We can such a function to a sum of a constant and an infinite series of cosine and sine functions, known as a Fourier series, like so:
$$
f(x) = c_0 + \sum_{n=1}^\infty a_n \cdot cos(n\cdot x) + b_n \cdot sin(n\cdot x)
$$
Where $c_0$ is a constant scalar, and $a_n, b_n$ are constant coefficients. The functions $1, cos(n \cdot x), sin(n \cdot x)$. Form a special kind of basis known as a _Fourier basis_. Written more explicitly, the Fourier basis is:
$$
\{1, cos(x), sin(x), cos(2x),sin(2x),cos(3x),sin(3x),...\}
$$
Note that this is just one of infinitely many possible Fourier bases, since scaling a basis yields another basis. Common among all Fourier bases is the constant term and the alternating cosine and sine terms.

> Note: You may have seen different definitions of Fourier series than what I have shown. In fact, Fourier series have several equivalent definitions, including the sine-cosine definition, the exponential definition, and amplitude-phase definition. The other definitions are not relevant to the topic at hand.
# Circular Harmonics
One particular kind of Fourier basis is formed by the so-called "Circular Harmonics" (CH). This basis is a scaled version of what we have seen before, given by:
$$
\{\frac{1}{\sqrt{2\pi}}, \frac{cos(x)}{\sqrt{\pi}}, \frac{sin(x)}{\sqrt{\pi}}, \frac{cos(2x)}{\sqrt{\pi}}, \frac{sin(2x)}{\sqrt{\pi}}, \frac{cos(3x)}{\sqrt{\pi}}, \frac{sin(3x)}{\sqrt{\pi}},...\}
$$
These functions form a basis for periodic functions with period $2\pi$, or the circumference of a unit circle (hence the name Circular Harmonics - these functions can be "wrapped" around a unit circle). The denominators in the basis functions aren't of particular importance - they are just normalization terms, there to ensure that we can easily _project into the basis_ (ie. calculate the required coefficients for a linear combination of the basis functions, given a function we want to approximate) using the exact same basis functions as we would use in the final _reconstruction_ (linear combination of basis functions). Importantly, this basis is orthonormal.
## Projection and reconstruction
As mentioned just above, 2 operations of particular importance are _projection_ and _reconstruction_. They are both fairly straight forward to define. If we want to project a function $f(x)$ into the CH basis, we do it like so:
$$
c_i = \int_0^{2\pi} f(x) \cdot b_i(x) \ dx
$$
Where $b_i$ is the i'th basis function, and $c_i$ is the coefficient to associate with the i'th basis function. Naturally, to reconstruct our approximate version of $f(x)$, we take a simple inner product:
$$
f(x) \approx \sum_{i=0}^n c_i \cdot b_i(x)
$$
A pragmatic way to think about Circular Harmonics are as a kind of compression - a way to store a $2\pi$-periodic function numerically using very little space. The more coefficients we store, the better the approximation will be, but we'll quickly face diminishing returns. In other words, the approximation will already be quite good with only a few coefficients for many kinds of functions, and in particular those of low-frequency. 
## Projection and reconstruction in practice
Next I'll show a concrete example of the aforementioned projection and reconstruction operations using some Python code. 

First, let's define a function which we will project into CH basis and then reconstruct. I've chosen a saw wave:
```python
def f(x):
    x = np.mod(x, 2*np.pi)
    return x/(2*np.pi)
```
![[Pasted image 20231005232357.png]]

Next, let's define a function that gives us the n'th basis function:
```python
import numpy as np

# n is the index of the basis function, x is the input to the function
def fourier_basis(n, x):
    multiplier = np.ceil(n / 2)
    if n == 0:
        return 1 / np.sqrt(2 * np.pi)
    elif n % 2 == 1:
        return np.cos(multiplier * x) / np.sqrt(np.pi)
    else:
        return np.sin(multiplier * x) / np.sqrt(np.pi)
```

Next, we need a way to calculate integrals. I've implemented a simple [Riemann sum](https://en.wikipedia.org/wiki/Riemann_sum):
```python
# function is the function to integrate
# a and b are the bounds of the integral
# n is the number of discrete steps to take
def riemann(function,a,b,n):
    sumval = 0
    h = (b-a)/n
    for i in range(0,n-1):
        current_x = a+i*h
        sumval = sumval + function(current_x) * h
    return sumval
```

With those 2 building blocks, we can define a function that calculates coefficient for the n'th basis function:
```python
# f is the function to project into the CH basis
# n is the index of the basis function
# calculates the integral of the n'th basis function multiplied by f
def fourier_project(n, f):
    return riemann(lambda x: f(x) * fourier_basis(n, x), 0, 2*np.pi, 1000)
```

We can then define a function that performs reconstruction. We'll use 30 basis functions for this example:
```python
# f is the function to project into CH basis, then reconstruct
# x is the is the input to the function
def fourier_reconstruct(f, x):
    sum = 0
    for n in range(30):
        sum += fourier_project(n, f) * fourier_basis(n, x)
    return sum
```

Finally, we can compare the actual and reconstructed functions:
```python
from matplotlib import pyplot as plt

x = np.linspace(-np.pi*2, np.pi*2, 1000)
fig, ax = plt.subplots()
ax.plot(x, f(x), label='f(x)')
ax.plot(x, fourier_reconstruct(f, x), label='f_reconstructed(x)')
ax.legend()
plt.show()
```
![[Pasted image 20231005232635.png]]
Looks pretty good to me!
# Fourier basis on the surface of a sphere
I mentioned earlier that Spherical Harmonics (SH) are an analogue of the Fourier basis on the surface of a sphere. Phrased differently, they are analogous to the Circular Harmonics, but with an extra dimension of space. Just like the CH basis, the SH basis is orthonormal.

Unlike with Circular Harmonics, the basis functions for the SH basis come in _levels_. Each level contains $(n+1)^2$ basis functions. You may have seen this represented as a "pyramid" of basis functions before:
![[Pasted image 20231008034016.png]]
By convention, we number each basis function starting at a negative index, and going to a positive index of equal absolute value. We typically write each basis function as $Y_l^m(\theta, \phi)$, where $l$ is the level, $m$ is the index, and $(\theta, \phi)$ are spherical coordinates representing a direction (see [[Spherical integrals]] for more info).

While the SH basis functions are typically written in literature using Spherical Coordinates, we are more often working with 3 dimensional unit vectors in graphics. These 2 representations are equivalent with the following transformation:
$$
\begin{gathered}
x = sin(\theta)cos(\phi)\\
y = sin(\theta)sin(\phi)\\
z = cos(\theta)
\end{gathered}
$$
Using this more convenient representation, the first 3 levels of SH basis functions are given by:

| Order | Basis functions |
| ----- | --------------- |
| $l=0$ |   $$Y_0^0(x,y,z) = \frac{1}{2}\sqrt{\frac{1}{\pi}}$$             |
| $l=1$ |      $$ Y_1^{-1}(x,y,z)=\frac{1}{2}\sqrt{\frac{3}{\pi}}y $$ $$ Y_1^{0}(x,y,z)=\frac{1}{2}\sqrt{\frac{3}{\pi}}z $$ $$ Y_1^{1}(x,y,z)=\frac{1}{2}\sqrt{\frac{3}{\pi}}x $$          |
| $l=2$ |     $$ Y_2^{-2}(x,y,z) = \frac{1}{2}\sqrt{\frac{15}{\pi}}xy $$  $$ Y_2^{-1}(x,y,z) = \frac{1}{2}\sqrt{\frac{15}{\pi}}yz $$ $$ Y_2^0(x,y,z) = \frac{1}{4}\sqrt{\frac{5}{\pi}}(3z^2-1) $$ $$ Y_2^{1}(x,y,z) = \frac{1}{2}\sqrt{\frac{15}{\pi}}xz $$ $$ Y_2^{2}(x,y,z) = \frac{1}{4}\sqrt{\frac{15}{\pi}}(x^2-y^2) $$

A few notable observations to be made here:
- The $l=0$ basis function is just a constant!
- The $l=1$ basis functions are just a constant multiplied by the $y$, $z$, and $x$ components of the input vector respectively.
- 3 of the $l=2$ basis functions are just a constant multiplied by the components of the input vector.
All of these observations are pertinent to efficiently evaluating Spherical Harmonics in certain use cases within the fields of graphics.

> Note: What I've shown thus far are known as the "real" (as in real numbers) Spherical Harmonics. Spherical Harmonics have a different expansion which uses complex instead of real numbers, and is a bit more complex. Since we are only usually working with real-valued functions in graphics programming, this expansion is less useful to us, so I've omitted it.
## Projection and reconstruction
Projection and reconstruction with the SH basis is completely analogous to case of circular harmonics. For projection:
$$
c_l^m = \int_0^{2\pi}\int_0^\pi f(\theta, \phi)Y_l^m(\theta,\phi)sin(\theta) \ d\theta \ d\phi
$$
Where $c_l^m$ is the coefficient to associate with basis function $Y_l^m(\theta, \phi)$. If you are confused about this integral, please see [[Spherical integrals]].

Reconstruction is again just an inner product:
$$
f(\theta, \phi) \approx \sum_{l=0}^n \sum_{m=-l}^l c_l^m \cdot Y_l^m(\theta, \phi)
$$
Where $n$ is the order (amount of levels) of the basis we are using.

> Note: The reason why I use Spherical Coordinates in the above description, despite having just explained that we usually prefer to use unit vectors, is primarily to make the integral more tenable. The space of unit vectors doesn't map intuitively to a pleasant domain of integration. Alternatively, we could write the double integral as a single integral in solid angle domain, but integrals in solid angle domain are not easy to calculate directly, so we usually translate to spherical coordinates before calculating anyways.
# Convolution
Spherical Harmonics have a nice property that lets us efficiently calculate [convolutions](https://en.wikipedia.org/wiki/Convolution). Consider this case of the convolution operation:
$$
\int_{S^2} F(\omega)G(\omega) \ d\omega
$$
Where $S^2$ denotes the 2-sphere aka. regular 3-dimensional sphere, and $d\omega$ is differential solid angle (see [[Spherical integrals]]).

The integral looks a bit daunting at first, but if we happen to have both $F(\omega)$ and $G(\omega)$ represented as Spherical Harmonics, we can calculate the result with ease using the following identity:
$$
\int_{S^2} F(\omega)G(\omega) \ d\omega = \sum_{l=0}^\infty\sum_{m=-l}^l f_l^m g_l^m
$$
Where $f_l^m$ are SH coefficients for $F(\omega)$ and $g_l^m$ are SH coefficients for $G(\omega)$. This is just another inner product.
# Use in global illumination
## A brief primer on radiometry
Two quantities of particular importance when rendering scenes with global illumination (GI) are radiance and irradiance. I'll briefly summarize what these quantities mean below:

| Quantity     | Unit                                                     | Meaning                                                                           |
| ------------ | -------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Radiant flux | Watts (Joule/Second), denoted $W$                       | Energy received or emitted per unit time.                                         |
| Irradiance   | Watts per square meter, denoted $W/m^2$                 | Radiant flux _received_ by a surface per unit area.                                |
| Radiance     | Watts per steradian per square meter, denoted $W/(m^2 \cdot sr)$ | Radiant flux received or emitted per unit solid angle, per unit _projected_ area. |
- [ ] TODO: Explain solid angle
Radiance is the quantity typically associated with a single ray of light, while irradiance is the quantity associated with a point on a surface. You can think of radiance as roughly being being the energy carried by a ray of light, and irradiance as the total energy incident on a point on a surface from all directions.

We can calculate the irradiance $E(x, n)$ at a given point on a surface by integrating radiance over the hemisphere of directions centered around the normal vector at the point, attenuated by the cosine of the angle between incoming light direction and normal vector:
$$
E(x, n) = \int_{\Omega(n)} L(x, \omega) max(\omega\cdot n, 0) \ d\omega
$$
Here $\Omega(n)$ denotes the hemisphere centered around normal vector $n$, $x$ is the point on the surface. $\omega$ denotes incoming light direction. The $\omega \cdot n$ term is the dot of the normal and incoming light direction, ie. the cosine of the angle between the 2. The entire $max(\omega \cdot n, 0)$ is sometimes aptly called the "clamped cosine term" and is sometimes written as $\lfloor\omega \cdot n\rfloor$.

>  The cosine term is necessary due to how projected area varies with the angle between the surface normal and the normal of the area being projected. Roughly speaking, we want to weigh light coming from shallow angles less, as that light is spread over a larger area. For some context, see the images in [[Cosine weighted sampling#Why]].
## Light probes
One technique for lighting scenes with GI is _light probes_. These are positions in space with which we associate a quantity of light. When it is time to shade a surface point, we can grab the few nearest probes to said point, interpolate between them, and use them to get an estimate of the indirect light contribution. That begs the question - what exactly do we store in each light probe? Spherical Harmonics turn out to be a great a fit, for a few main reasons:
- They are trivial to interpolate between.
- They can approximate low-frequency signals (such as is the case for irradiance) using very little data. We'll typically L2 Spherical Harmonics, which amounts to 27 coefficients per probe assuming we are using RGB (as opposed to a different method of sampling the visible spectrum).
- They are very cheap to evaluate in realtime. Evaluation boils down to basically a few dot products.

Recall that the radiometric quantity associated with light incident on a surface point is irradiance. It should then be no surprise that the quantity we should project into the SH basis and store in each light probe is also _irradiance_. To compute the coefficients for each probe, we can use path tracing.

Path tracing involves shooting a bunch of light rays from a point, bouncing them around the scene, and attenuating their contribution whenever they hit a surface. Note however that the radiometric quantity associated with a ray of light is _radiance_, not irradiance. I'll ignore this mismatch in desired quantities for now, and first explain how to project radiance into the L2 SH basis. Below is some pseudo-C#-code to do this. Note that I'm not going to bother with color - we'll just be using monochromatic lighting for simplicity.
```cs
// Given a probe position, returns the SH coefficients for radiance
// incoming from all directions to that position.
float[] ProjectRadianceIntoSH(Vector3 probePosition)
{
	float[] radianceSH = new float[9];
    for (int i = 0; i < TotalSamplesPerProbe; i++)
    {
	    // Trace a light path going in a random direction to get radiance
	    // associated with the direction
	    Vector3 rayDirection = GetRandomDirection();
	    float radiance = TracePath(probePosition, rayDirection);

		for (int n = 0; n < 9; n++)
		{
			// SHBasis(n, d) evaluates the n'th SH basis function
			// given a direction vector d.
			radianceSH[coefficient] = radiance * SHBasis(n, rayDirection);
		}
    }
    return radianceSH;
}
```
For convenience, I've used a flat index for indexing SH basis functions, rather than an explicit level $l$ and index-in-level $m$. Index 0 thus means basis function $Y_0^0$, index 1 is $Y_1^{-1}$, index 2 is $Y_1^0$ etc. Using L2 SH means we have to store 9 coefficients for the 9 basis functions. This pseudocode is essentially just implementing the SH projection operator described earlier, with radiance in a given direction, $L(x, w)$ as the function to project.
## Radiance to irradiance
Now we are faced the issue I outlined earlier: If we want to actually shade a surface with this, we need irradiance rather than radiance, which means that we essentially need to calculate the integral described in [[#A brief primer on radiometry]]:
$$
E(x, n) = \int_{\Omega(n)} L(x, \omega) max(\omega\cdot n, 0) \ d\omega
$$
Ramamoorthi and Hanrahan describe how to do just this in their paper [On the relationship between radiance and irradiance: determining the illumination from images of a convex Lambertian object](https://cseweb.ucsd.edu/~ravir/papers/invlamb/josa.pdf). I'm not going to go over all the details of their derivations, but focus on the conclusions in broad strokes - you can read the paper for more info. 

Ramamoorthi defines an operator for the clamped cosine term called $A(\theta)$: 
$$A(\theta) = max(cos(\theta), 0) = \sum_l A_l Y_l^0$$
Where:
- $\theta$ is the angle between the surface normal and the incident light direction.
- $A_l$ is the coefficient associated with the l'th SH level, of the clamped cosine term projected into the SH basis.
- $Y_l^0$ is the SH basis function at level $l$, index 0.
It's important to note here that we are only indexing the SH coefficients of the for $A(\theta)$ projected into the SH basis using the level $l$, and omitting the index in said level. The clamped cosine term is _only_ dependent on the polar angle $\theta$, and not the azimuthal angle $\phi$. This should make intuitive sense - the total amount of light reflected at a point on a lambertian surface is only dependent on how shallow the angle of incidence is. Functions with this property - azimuthal independence - will only have non-zero coefficients for the SH basis functions with index 0, ie. those in the middle column of the SH pyramid shown earlier. This column of basis functions are called the _zonal harmonics_. Due to this property, we can any coefficient that isn't associated with a zonal harmonic - and we end up with only as many coefficients for the projection of $A(\theta)$ as we have levels of SH.

Ramamoorthi then goes on to derive an analytical expression for the coefficients $A_l$:
$$
A_l =
\begin{cases}
\sqrt{\frac{\pi}{3}}, & \text{if } l = 1\\
0, & \text{if } l > 1 \text{ and $l$ is odd}\\
2\pi \sqrt{\frac{2l+1}{4\pi}}\frac{(-1)^{(l/2-1)}}{(l+2)(l-1)}\frac{l!}{2^l(l!/2)^2}  & \text{if $l$ is even}
\end{cases}
$$

Let's now revisit the irradiance integral with these definitions. We'll start by rewriting it a tiny bit:
$$
E(n) = \int_{\Omega(n)} L(\omega) max(\omega\cdot n, 0) \ d\omega = \int_{\Omega(n)} L(\omega) A(\theta) \ d\omega
$$
Note that we have dropped the surface position $x$ from irradiance function $E(x, n)$. This is because we assume the illumination is _distant_, which means changes in position have negligible effect.

Given the nice property I described in the section [[Spherical Harmonics#Convolution]], one may now be tempted calculate irradiance integral like so:
$$
E(n) = \int_{\Omega(n)} L(\omega) A(\theta) \ d\omega = \sum_{l=0}^\infty\sum_{m=-l}^l L_l^m A_l
$$
However, **this would be incorrect**. The reason is a mismatch in coordinate systems. Incoming radiance, $L(x,\omega)$ is with respect to _global coordinates_, while the clamped cosine term $A(\omega)$ is with a _local coordinates_ ie. the coordinate system where the surface normal at the point we are shading is pointing straight upwards. We need to introduce a rotation term in order to account for this. I'll omit some details here, and name this term somewhat opaquely $R_l^m(n)$:
$$
E(n) = \sum_{l=0}^\infty\sum_{m=-l}^l L_l^m A_l \ R_l^m(n)
$$
Ramamoorthi shows that this rotation term can be rewritten to:
$$
R_l^m(n) = \sqrt{\frac{4\pi}{2l+1}}Y_l^m(n)
$$
Thus the expression for irradiance becomes:
$$
E(n) = \sum_{l=0}^\infty\sum_{m=-l}^l L_l^m A_l \sqrt{\frac{4\pi}{2l+1}} Y_l^m(n)
$$
In the follow up paper [An Efficient Representation for Irradiance Environment Maps](https://cseweb.ucsd.edu/~ravir/papers/envmap/envmap.pdf), Ramamoorthi further simplifies this expression by defining a new set of coefficients $\hat{A}_l$:
$$
\hat{A}_l = A_l \sqrt{\frac{4\pi}{2l+1}}
$$
Which lets us finally express irradiance as:
$$
E(n) = \sum_{l=0}^\infty\sum_{m=-l}^l L_l^m \hat{A}_l \ Y_l^m(n)
$$
Next, we rewrite irradiance in terms of SH coefficients, using what we know from the sections on projection:
$$
E(n) = \sum_{l=0}^\infty\sum_{m=-l}^l E_l^m Y_l^m(n)
$$
It then becomes evident from the last 2 equations that:
$$
E_l^m = L_l^m\hat{A}_l
$$
Phrased in plain English, if we have _radiance_ in SH, given by coefficients $L_l^m$, and we want _irradiance_ in SH, given by coefficients $E_l^m$, we need only multiply the radiance coefficients by a factor $\hat{A}_l$ which varies per SH level. Since we already have an analytical expression for $A_l$, and $A_l$ differs from $\hat{A}_l$ only by a constant factor per SH level, we can derive an analytical expression for $\hat{A}_l$ too, which is exactly what Ramamoorthi does next:
$$
\hat{A}_l =
\begin{cases}
\frac{2\pi}{3}, & \text{if } l = 1\\
0, & \text{if } l > 1 \text{ and $l$ is odd}\\
2\pi \frac{(-1)^{(l/2-1)}}{(l+2)(l-1)}\frac{l!}{2^l(\frac{l}{2}!)^2}  & \text{if $l$ is even}
\end{cases}
$$
The $\hat{A}_l$ term for the first 3 levels of SH (which is what we typically care about for light probes), is:
$$
\begin{gathered}
\hat{A}_0 = \pi = 3.14159...\\
\hat{A}_1 = \frac{2\pi}{3} = 2.09439...\\
\hat{A}_2 = \frac{\pi}{4} = 0.78539...
\end{gathered}
$$
## Putting it into practice
We can put this all together to write some pseudocode that convolves our SH coefficients for radiance incoming from a given direction, to irradiance given the normal vector of a surface point:
```cs
// Given 9 SH coefficients (L2 SH) encoding directional radiance,
// return 9 SH coefficients encoding directional irradiance
float[] ConvolveRadianceToIrradianceSH(float[] radianceSH)
{
	float[] irradianceSH = new float[9];
	
	// L0
	float aHat0 = 3.14159;
	irradianceSH[0] = aHat0 * radianceSH[0];
	
	// L1
	float aHat1 = 2.09439;
	irradianceSH[1] = aHat1 * radianceSH[1];
	irradianceSH[2] = aHat1 * radianceSH[2];
	irradianceSH[3] = aHat1 * radianceSH[3];
	
	// L2
	float aHat2 = 0.78539;
	irradianceSH[4] = aHat2 * radianceSH[4];
	irradianceSH[5] = aHat2 * radianceSH[5];
	irradianceSH[6] = aHat2 * radianceSH[6];
	irradianceSH[7] = aHat2 * radianceSH[7];
	irradianceSH[8] = aHat2 * radianceSH[8];

    return irradianceSH;
}
```

As well as a function to shade a surface given a surface normal, surface albedo and a set of SH coefficients corresponding to the a light probe:
```cs
float ShadeSH(float[] irradianceSH, float surfaceAlbedo, Vector3 surfaceNormal)
{
    float outputColor = 0;
    for (int n = 0; n < 9; n++)
    {
        outputColor += irradianceSH[n] * SHBasis(n, surfaceNormal);
    }
    return outputColor * surfaceAlbedo;
}
```
Note that I am still using monochromatic lighting for simplicity, so the surface albedo is just a single scalar.

With this, we can finally write pseudocode to shade a point using a light probe:
```cs
// First, we "bake" a probe at a given position:
Vector3 probePosition = ...;
float[] radianceSH = ProjectRadianceIntoSH(probePosition);
float[] irradianceSH = ConvolveRadianceToIrradiance(radianceSH);

// Next, we use it to shade a point with a given normal:
Vector3 surfaceNormal = ...;
float surfaceAlbedo = ...;
float colorAtSurface = ShadeSH(irradianceSH, surfaceAlbedo, surfaceNormal);
```

In reality, instead of using a single probe, we would bake multiple probes, yielding several sets of SH coefficients, and then interpolate the few nearest ones to get a set of interpolated SH coefficients for irradiance. This is just a dumbed down example.
# Stupid SH tricks (Peter-Pike Sloan)
# Ringing

#math #global-illumination #light-transport #path-tracing 