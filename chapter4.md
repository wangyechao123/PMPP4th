# Chapter 4


### 1a 
There are 128 threads, so there are 128 / 32 = 4 warps.

### 1b
32 warps

### 1c
1. There are 4 warps in one block, the first, second and fourth warp will do line 04, so there are 3 active warps in one block. There are 24 warps active in the grid.  
2. The second and fourth warp are divergent in one block. So there are 16 warps in the grid are divergent.  
3. SIMD efficiency refers to the ratio of active threads to the maximum number of threads per warp. So SIMD efficiency of warp 0 is 100%.  
4. 8 / 32 = 25%  
5. 24 / 32 = 75%  

### 1d
i is the global index of thread.  
1. All the warps are active.
2. All the warps are divergent.
3. 50%

### 1e
i=0, j=0,1,2,3,4  
i=1, j=0,1,2,3  
i=2, j=0,1,2  
......

All thread are active when j=0,1,2, but warps are divergent when j=3,4.
So 3 iterations have no divergence, 2 iterations have divergence.


### 2
2048

### 3
\[1984, 2016\) threads will execute different code, so 1 warp have divergence. (not 2 warps have divergence, the last warp don't have divergence because all threads in the last warp execute nothing)

### 4
$$
1 - \frac{2.0 + 2.3 + 3.0 + 2.8 + 2.4 + 1.9 + 2.6 + 2.9}{3.0 * 8} = 17\%
$$

### 5 
It's not a good idea, the threads in one warp may execute different instruction, it may result in dead lock.

### 6 
C  
We can get the execution configuration parameters for each case under such resource limitations.  
a. <<<4, 128>>> # threads = 512  
b. <<<4, 256>>> # threads = 1024  
c. <<<3, 512>>> # threads = 1536  
d. <<<1, 1024>>> # threads = 1024  

### 7 
All cases are possible. The occupancy level are 50%, 50%, 50%, 100%, 100%.

### 8
Under such resource limits, we first conclude the formula of the number of blocks:
$$
num\_of\_blocks = \min\{32 \; blocks\_per\_SM, \frac{2048 \; threads\_per\_SM}{threads\_per\_block}, \frac{65536 \; registers\_per\_SM}{registers\_per\_threads * threads\_per\_block}\}
$$
a. #blocks = min{32, 16, 17.1} = 16, occupancy = 100%  
b. #blocks = min{32, 64, 70.6} = 32, occupancy = 32 * 32 / 2048 = 50%, the limits of blocks per SM  
c. #blocks = min{32, 8, 7.5} = 7, occupancy = 7 * 256 / 2048 = 87.5%, the limits of registers  

### 9
It can not work. It needs 32 * 32 = 1024 threads per block, which beyond the limits of 512 threads per block. (Attention: 32 * 32 = 1024 blocks, which is greater than 8 blocks per SM is just OK)
