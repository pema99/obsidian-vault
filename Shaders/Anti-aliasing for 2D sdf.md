```c
float lerpstep(float a, float b, float x)
{
    return saturate((x - a)/(b - a));
}

void addElement(inout float3 existing, float3 elementColor, float elementDist)
{ 
	const float pixelDiagonal = sqrt(2.0) / 2.0;
	float distDerivativeLength = sqrt(pow(ddx(elementDist), 2) + pow(ddy(elementDist), 2));
	existing = lerp(elementColor, existing, lerpstep(-pixelDiagonal, pixelDiagonal, elementDist/distDerivativeLength)); #endif 
}
```
- Calculate magnitude of screen-space gradient of distance
- Convert distance to screen-space distance by dividing by magnitude
- Lerp between existing and new color, using lerpstep or smoothstep with a neighborhood of $\frac{sqrt(2)}2$ ie. half the pixel diagonal

#shader-snippets #shaders 