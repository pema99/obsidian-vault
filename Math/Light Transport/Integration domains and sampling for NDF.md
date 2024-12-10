> Note: This page is still quite rough / unfinished.

NDF (normal distribution function) are typically given in terms of microfacet area. To convert to macrosurface area, we need to multiple by $cos(\theta)$. This is because $dA = (n \cdot h) dA_h$. To get into spherical coordinates, we need to multiple by $sin(\theta)$ since $d\Omega = sin(\theta) d\theta d\phi$.

![[Pasted image 20241108005122.png]]

So before creating sampling routines for an NDF, first multiply by
$cos(\theta)sin(\theta)$, the jacobian determinant of the transformation.

Quick reference:
$d\Omega = sin(\theta) d\theta d\phi$
$dA=r^2 cos(\theta) d\Omega$
$dA = (n \cdot h) dA_h$

To get a sampling routine given the PDF in spherical coordinate domain, first integrate out $\phi$ to get the marginal density $p(\theta)=\int_0^{2\pi}p(\theta, \phi)d\phi$. Then get conditional density $p(\phi|\theta)=\frac{p(\theta, \phi)}{p(\theta)}$.

Then use inverse transform sampling. For each function calculate the CDF by integration:
$CDF(t) = \int_0^t p(t) dt$

Finally set the CDF equal to a random variable and solve for the integration variable:
$solve(X=CDF(t), t)$

Links:
https://www.reedbeta.com/blog/hows-the-ndf-really-defined/
https://agraphicsguynotes.com/posts/sample_microfacet_brdf/
https://www.tobias-franke.eu/log/2014/03/30/notes_on_importance_sampling.html
https://jcgt.org/published/0007/04/01/