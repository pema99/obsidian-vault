```c
float4x4 inverse(float4x4 mat)
{
	float4x4 M=transpose(mat);
	float m01xy=M[0].x*M[1].y-M[0].y*M[1].x;
	float m01xz=M[0].x*M[1].z-M[0].z*M[1].x;
	float m01xw=M[0].x*M[1].w-M[0].w*M[1].x;
	float m01yz=M[0].y*M[1].z-M[0].z*M[1].y;
	float m01yw=M[0].y*M[1].w-M[0].w*M[1].y;
	float m01zw=M[0].z*M[1].w-M[0].w*M[1].z;
	float m23xy=M[2].x*M[3].y-M[2].y*M[3].x;
	float m23xz=M[2].x*M[3].z-M[2].z*M[3].x;
	float m23xw=M[2].x*M[3].w-M[2].w*M[3].x;
	float m23yz=M[2].y*M[3].z-M[2].z*M[3].y;
	float m23yw=M[2].y*M[3].w-M[2].w*M[3].y;
	float m23zw=M[2].z*M[3].w-M[2].w*M[3].z;
	float4 adjM0,adjM1,adjM2,adjM3;
	adjM0.x=+dot(M[1].yzw,float3(m23zw,-m23yw,m23yz));
	adjM0.y=-dot(M[0].yzw,float3(m23zw,-m23yw,m23yz));
	adjM0.z=+dot(M[3].yzw,float3(m01zw,-m01yw,m01yz));
	adjM0.w=-dot(M[2].yzw,float3(m01zw,-m01yw,m01yz));
	adjM1.x=-dot(M[1].xzw,float3(m23zw,-m23xw,m23xz));
	adjM1.y=+dot(M[0].xzw,float3(m23zw,-m23xw,m23xz));
	adjM1.z=-dot(M[3].xzw,float3(m01zw,-m01xw,m01xz));
	adjM1.w=+dot(M[2].xzw,float3(m01zw,-m01xw,m01xz));
	adjM2.x=+dot(M[1].xyw,float3(m23yw,-m23xw,m23xy));
	adjM2.y=-dot(M[0].xyw,float3(m23yw,-m23xw,m23xy));
	adjM2.z=+dot(M[3].xyw,float3(m01yw,-m01xw,m01xy));
	adjM2.w=-dot(M[2].xyw,float3(m01yw,-m01xw,m01xy));
	adjM3.x=-dot(M[1].xyz,float3(m23yz,-m23xz,m23xy));
	adjM3.y=+dot(M[0].xyz,float3(m23yz,-m23xz,m23xy));
	adjM3.z=-dot(M[3].xyz,float3(m01yz,-m01xz,m01xy));
	adjM3.w=+dot(M[2].xyz,float3(m01yz,-m01xz,m01xy));
	float invDet=rcp(dot(M[0].xyzw,float4(adjM0.x,adjM1.x,adjM2.x,adjM3.x)));
	return transpose(float4x4(adjM0*invDet,adjM1*invDet,adjM2*invDet,adjM3*invDet));
}
```

#shaders #shader-snippets #math 