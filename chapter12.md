# Chapter 12

### 1
5

### 2 
co_rank(6, A, 5, B, 4)  
i=5, j=1, i_low=2, j_low=1  
then enter the "else" branch,  and return i=5  

### 3
This question is the exercise in the below of page 276.  
Firstly I list the code in Fig12.12  
```c++
for(int i = 0; i < tile_size; i += blockDim.x){
    if(A_consumed + i + threadIdx.x < A_length){
        A_S[i + threadIdx.x] = A[A_curr + A_consumed + i + threadIdx.x];
    }
}
```
And then I give the answers below  
```C++
c_iter_curr = counter * tile_size; 
c_iter_next = min((counter + 1) * tile_size, C_length); 
a_iter_curr = co_rank(
    c_iter_curr, 
    A + A_curr + A_consumed, A_length, 
    B + B_curr + B_consumed, B_length); 
a_iter_next = co_rank(
    c_iter_next, 
    A + A_curr + A_consumed, A_length, 
    B + B_curr + B_consumed, B_length); 

for(int i = 0; i < tile_size; i += blockDim.x){
    if(i + threadIdx.x < a_iter_next - a_iter_curr){
        A_S[i + threadIdx.x] = A[A_curr + A_consumed + i + threadIdx.x];
    }
}
```

### 4
m + n = 1638400, every block handles 8 * 1024 = 8192 elements, so there are 200 blocks.  
a. All the threads perform binary search. There are 200 * 1024 = 204800 threads.  
b. Only the first thread perform binary search in global memory. So 200 threads.  
c. All the threads need perform binary search in shared memory. So 204800 threads.  