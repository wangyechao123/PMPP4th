# Chapter 7 Convolution

### 1
51

### 2
8, 21, 13, 20, 7

### 3
a. stay the same  
b. move to the left by one element  
c. move to the right by one element  
d. calculate the first derivative  
e. calculate the average

### 4
assume M is odd number, and let $r = (M - 1) / 2$  
a. $2r$  
b. $N * M$ multiplications. The size of output array is N, M multiplications for each element.  
c. $(N - 2r) * M$. The size of output array is N - 2r, M multiplications for each element.

### 5
assume M is odd number, and let $r = (M - 1) / 2$  
a. $4r(r + 1)$. The four corners.  
b. $N^2M^2$  
c. $(N-2r)^2M^2$

### 6
let $r_1 = (M_1 - 1) / 2$ and $r_2 = (M_2 - 1) / 2$  
a. $4r_{1}r_{2} + 2N_{1}r_{2} + 2N_{2}r_{1}$  
b. $N_{1}N_{2}M_{1}M_{2}$  
c. $(N_{1} - 2r_1)(N_{2} - 2r_2)M_{1}M_{2}$

### 7
let $r = (M - 1) / 2$  
a. $(\lfloor{\frac{N}{T}}\rfloor{} + 1) \times (\lfloor{\frac{N}{T}}\rfloor{} + 1)$  
b. $(T+2r) \times (T+2r)$  
c. $(T+2r) \times (T+2r) \times 4$ Byte  
d. num of blocks is the same, num of threads per block is $T \times T$, shared memory is $4T^2$ Byte  
