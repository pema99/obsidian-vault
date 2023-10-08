```c
void revisedONB(const Vec3f &n, Vec3f &b1, Vec3f &b2) {
    if(n.z<0.)
    {
	    const float a = 1.0f / (1.0f - n.z);
	    const float b = n.x * n.y * a;
	    b1 = Vec3f(1.0f - n.x * n.x * a, -b, n.x);
	    b2 = Vec3f(b, n.y * n.y*a - 1.0f, -n.y);
	} 
	else
	{
		const float a = 1.0f / (1.0f + n.z);
		const float b = -n.x * n.y * a;
		b1 = Vec3f(1.0f - n.x * n.x * a, b, -n.x);
		b2 = Vec3f(b, 1.0f - n.y * n.y * a, -n.y); 
	}
}
```
Branchless:
```c
void branchlessONB(const Vec3f &n, Vec3f &b1, Vec3f &b2)
{ 
	float sign = copysignf(1.0f, n.z); 
	const float a = -1.0f / (sign + n.z); 
	const float b = n.x * n.y * a; 
	b1 = Vec3f(1.0f + sign * n.x * n.x * a, sign * b, -sign * n.x); 
	b2 = Vec3f(b, sign + n.y * n.y * a, -n.y);
}
```
https://jcgt.org/published/0006/01/01/paper.pdf

#math