> Disclaimer: I'm not a physicist. This is probably all wrong. Don't @ me.
# Background
Light is electromagnetic radiation. Electromagnetic waves consist of 2 perpendicular waves in the electric and magnetic fields. By shifting the phase of these 2 waves, we get different polarization states.
![[Pasted image 20230923211452.png]]
When the waves are perfectly out of phase, we get circularly polarized light, and they are perfectly in phase, we get linearly polarized light. 

```ad-tip
Most natural light is "unpolarized" light. This essentially means a mixture of incoherent linearly and circularly polarized light. Added together, the light as a whole has no polarization.
```

Polarization is directly related to the quantum mechanical "spin" of photons. Photon can have 1 of 2 spin-states, which we'll call left and right spin. Waves of photons with left or right spin correspond to left- or right-hand circularly polarized light. Linearly polarized light consists of photons in superposition of the 2 spin states. The opposing circular rotations add together resulting in oscillation in a single direction.
![[eTkln5rPR6.gif]]
The image shows 2 circularly polarized waves with opposing handedness adding together to form linear polarization. Note how we can change the angle of linear polarization by shifting the phase of the 2 circularly polarized waves. 
![[gficFwsUQ5.gif]]
# Polarizing filters
Using polarizing filters, we can change the polarization of light waves passing through them. They can be used to convert unpolarized light to linearly polarized light. 
![[Pasted image 20230923214156.png]]
When _linearly_ polarized light passes through a polarizing filter, only some of it is let through. How much is let through depends on the difference between the angle of polarization and the angle of the polarizing filter. If the angles are the same, all the light is let through. If they are exactly perpendicular, none of the light is let through.
![[Pasted image 20230923214516.png]]
Polarizing filters can be viewed as quantum mechanical measurement devices, and can thus be used to prove the infamous [bell inequality](https://en.wikipedia.org/wiki/Bell%27s_theorem). By inserting a diagonal polarizing filter between 2 perpendicular polarizing filters, we somehow get more light passing through.
![[Pasted image 20230923214729.png]]
# Mueller-Stokes calculus
One mathematical object for describing the state of polarized light is the stokes vector. It is a 4D vector written as:
$$
\begin{bmatrix}
S_0 \\
S_1 \\
S_2 \\
S_3
\end{bmatrix}
=
\begin{bmatrix}
I \\
Q \\
U \\
V
\end{bmatrix}
$$
![[Pasted image 20230923215710.png]]
The I parameter describes the intensity of the light. It is equivalent to radiance. The Q parameter distinguishes horizontal and linear polarization with a sign. The U parameter distinguishes diagonal polarization at +45 or -45 degrees, again using the sign. The V parameter distinguishes right and left circular polarization using the sign.
![[Pasted image 20230923220012.png]]
Of course, this only makes sense given a reference frame (coordinate system). We typically say +Z is along the direction of the light by convention. That leaves choosing a direction for X and Y, the 2 of which form the reference frame.

Mueller matrices are matrices that transform stokes vectors, altering their polarization state. Each column of a Mueller matrix is a stokes vector. Mueller matrices are only valid to apply if the reference frame of the stokes vector and Mueller matrix are identical. This can be achieved by applying a rotation before multiplying the Mueller matrix. The rotation is itself represented by a matrix.
![[Pasted image 20230923220434.png]]
# Rendering
When implementing polarized light into a path tracer, the state of each ray can be represented by a combination of stokes vectors, rather than a combination of scalars. In a typical, non-spectral, path tracer, each ray carries a 3D dimensional vector with values corresponding to the intensity of red, green and blue light respectively. When adding polarized light, we may instead have a stokes vector for each channel, so $4*3=12$ scalars for each ray in total. Each ray must also carry its reference frame, such that we can properly rotate Mueller matrices before multiplying them.

Materials that affect polarization state are then naturally represented by Mueller matrices, also with each their own reference frame. For example, a horizontal polarizing filter may have the Mueller matrix:
![[Pasted image 20230923221034.png]]

- [ ] Fresnel equations
- [ ] Mueller matrices for fresnel

https://mitsuba.readthedocs.io/en/stable/src/key_topics/polarization.html

#math #physics #light-transport 