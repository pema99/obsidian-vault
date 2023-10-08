Here is the correspondence between SphericalHarmonicsL2 and the properties fed to shaders (unity_SHAr...unity_SHC):
```c
// outCoeffs must be size 7
private void SHToShaderCoefficients(ref SphericalHarmonicsL2 sh, ref Vector4[] outCoeffs)
{
	// outCoeffs will have this order:
	// [0] = unity_SHAr
	// [1] = unity_SHAg
	// [2] = unity_SHAb
	// [3] = unity_SHBr
	// [4] = unity_SHBg
	// [5] = unity_SHBb
	// [6] = unity_SHC

	for (int i = 0; i < 3; i++)
	{
		outCoeffs[i] = new Vector4(
			sh[i, 3],
			sh[i, 1],
			sh[i, 2],
			sh[i, 0] - sh[i, 6]
		);

		outCoeffs[i + 3] = new Vector4(
			sh[i, 4],
			sh[i, 5],
			sh[i, 6] * 3.0f,
			sh[i, 7]
		);
	}

	outCoeffs[6] = new Vector4(
		sh[0, 8],
		sh[1, 8],
		sh[2, 8],
		1.0f
	);
}
```

#shaders #unity