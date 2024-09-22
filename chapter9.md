# Chapter 9

### 1
$\frac{1}{100 \times 10^{-9}} = 10^7$ atomic operations per second

### 2
total time is $0.9 \times 4 + 0.1 \times 100 = 13.6 ns$, so the throughput is $7.35 \times 10^7$ atomic operations per second

### 3 
$5 \times 10^7$ floating-point operations per second

### 4
$5 \times \frac{1}{1.1 \times 10^{-9}} = 4.54 \times 10^9$ floating-point operations per second

### 5 
d. atomicAdd(&Total, Partial);  

### 6
a. 524288 atomic operations performed on global memory.  
b. Only the final step (commit to global memory) exists atomic operations performed on global memory because of shared memory. The number of blocks is $\lceil{}\frac{524288}{1024}\rceil{} = 512$, and there are the first 128 threads execute commiting to global memory. So there are $512 \times 128 = 65536$ atomic operations performed on global memory.  
c. Thread coarsening can decrease the number of blocks. The number of blocks is $\lceil{}\frac{524288}{1024 \times 4}\rceil{} = 128$. So there are $128 \times 128 = 16384$ atomic operations performed on global memory
