```c
v2f vert (float4 vertex : POSITION, float2 uv : TEXCOORD0)
{
	v2f o;
	o.vertex = float4(float2(1,-1)*(uv*2-1),1,1);
	o.clipPos = o.vertex;
	o.inverseVP = inverse(UNITY_MATRIX_VP);
	return o;
}

float4 frag (v2f i) : SV_Target
{
	float4 clipPos = i.clipPos / i.clipPos.w;
	clipPos.z = tex2Dproj(_CameraDepthTexture, ComputeScreenPos(clipPos));
	float4 homWorldPos = mul(i.inverseVP, clipPos);
	float3 wpos = homWorldPos.xyz / homWorldPos.w;
	return float4(wpos, 1.0f);
}
```
Just use inverse VP matrix. Need the perspective divide because clip space is homogenous (scaling doesn't affect).

https://gist.github.com/pema99/b13a76508bba3e8b70caaaea920ec1c3

#shaders #shader-snippets #math #unity 
