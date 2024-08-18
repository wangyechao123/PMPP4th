# Chapter 5

### 1
No, there is no coorperation between different threads.  

### 2
Exercise 2 is just a verification of page 104.

### 3

### 4
Shared memory can be utilized by all threads in one block. But registers are private to one thread.

### 5
The memory bandwidth of tiled matrix multiplication version is 1/32 of non-tiled version.

### 6
512 * 1000

### 7 
1000

### 8
a. N.  
b. N/T. One block need only one global memory access, and there are N/T blocks.

### 9
a. memory-bound  
b. compute-bound  

### 10
a. BLOCK_SIZE should be equal to BLOCK_WIDTH  
b. Thread need to be synchronized. Add `__syncthreads()` between Line 10 and Line 11.  

### 11
a. 1024. equal to num of threads in the grid.  
b. 1024  
c. 8  
d. 8  
e. (1 + 128) * 4 B = 516 B  
f. Global Memory Access: Line 07: 4 times, Line 12: 1, Line 14: 1. Totally 6 times global memory access.   
Float point operation: only Line 14 15: 5 multiplication, 1 mod, 5 add. Totally 11 times float point operation.  
So 11/6 OP/B.

### 12
a. num of blocks = min{2048/64, 32, 65536/(27\*64), 96/4} = min{32, 32, 37.9, 24} = 24  
The limiting factor is shared memory per SM.  
b. num of blocks = min{2048/256, 32, 65536/(31\*256), 96/8} = min{8, 32, 8.26, 12} = 8  
The limiting factor is threads per SM.