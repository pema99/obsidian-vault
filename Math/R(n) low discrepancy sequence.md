Great blog post about R(2) sequence http://extremelearning.com.au/unreasonable-effectiveness-of-quasirandom-sequences/. It's essentially just a mul and an add.

The post suggest to use the generalized golden ratio as the irrational numbers for R(n)
```python
import numpy as np

def phi(d): 
  x=2.0000 
  for i in range(10): 
    x = pow(1+x,1/(d+1)) 
  return x

d=2 # Number of dimensions. 
n=50 # number of required points 

g = phi(d) 
alpha = np.zeros(d) 
for j in range(d): 
  alpha[j] = pow(1/g,j+1) %1 
z = np.zeros((n, d)) 

for i in range(n): 
  z[i] = (seed + alpha*(i+1)) %1 
print(z)
```

However I found that for very high dimensionality, square roots of various primes work better. Thanks to @loicvdbruh for this:

```rust
// From loicvdbruh: https://www.shadertoy.com/view/NlGXzz. Square roots of primes.
const LDS_MAX_DIMENSIONS: usize = 32;
const LDS_PRIMES: [u32; LDS_MAX_DIMENSIONS] = [
    0x6a09e667u32, 0xbb67ae84u32, 0x3c6ef372u32, 0xa54ff539u32, 0x510e527fu32, 0x9b05688au32, 0x1f83d9abu32, 0x5be0cd18u32,
    0xcbbb9d5cu32, 0x629a2929u32, 0x91590159u32, 0x452fecd8u32, 0x67332667u32, 0x8eb44a86u32, 0xdb0c2e0bu32, 0x47b5481du32,
    0xae5f9155u32, 0xcf6c85d1u32, 0x2f73477du32, 0x6d1826cau32, 0x8b43d455u32, 0xe360b595u32, 0x1c456002u32, 0x6f196330u32,
    0xd94ebeafu32, 0x9cc4a611u32, 0x261dc1f2u32, 0x5815a7bdu32, 0x70b7ed67u32, 0xa1513c68u32, 0x44f93634u32, 0x720dcdfcu32
];

pub fn lds(n: u32, dimension: usize, offset: u32) -> f32 {
    const INV_U32_MAX_FLOAT: f32 = 1.0 / 4294967296.0;
    (LDS_PRIMES[dimension].wrapping_mul(n.wrapping_add(offset))) as f32 * INV_U32_MAX_FLOAT 
}
```

Bonus: PCG hash function. Sometimes useful when doing low discrepancy stuff:
```rust
pub fn pcg_hash(input: u32) -> u32 {
    let state = input * 747796405u32 + 2891336453u32;
    let word = ((state >> ((state >> 28u32) + 4u32)) ^ state) * 277803737u32;
    (word >> 22u32) ^ word
}
```

#math #probability